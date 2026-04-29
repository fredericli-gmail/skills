---
name: CodeReview
description: "程式碼審查專家。於開發完成後自動審查程式碼品質，檢查項目包含：程式碼風格（禁用 Lombok/Lambda/Stream）、繁體中文註解完整度、資安規範（XSS、CSRF、SQL Injection）、Null Check 與異常處理、邏輯正確性。產出表格式分類報告，依嚴重程度分級（錯誤、警告、建議）。審查通過後串接「Commit」Skill 進行規範化提交，發現問題則串接對應的開發 Skill 進行修正。"
---

# Skill: CodeReview（程式碼審查專家）

> ⚠️ **本 SKILL 引用共享規範**：
> - 編碼規範：[_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md)
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)
> - 完整檢查項目：[CHECKLIST.md](CHECKLIST.md)
> - GitHub Models 引擎：[COPILOT.md](COPILOT.md)

## Usage Trigger

- 當任一「開發-*」Skill 完成且編譯/執行成功後自動觸發
- 當使用者要求進行程式碼審查時
- 當需要檢查程式碼是否符合專案規範時

---

## 1. 審查流程

### 1.1 審查前置作業

#### 1.1.1 審查範圍選擇（強制執行）

> ⚠️ **每次審查啟動時都必須詢問範圍選擇，不得記憶上次選擇**。

```
【審查範圍選擇】

請選擇本次審查涵蓋的範圍：

  [1] 工作區未 commit 的異動（預設）
      內容：git diff HEAD（含 staged 與 unstaged） + 未追蹤新檔
      適用：開發中還沒 commit 想先做一次審查

  [2] 最近一個 commit（HEAD~1..HEAD）
      內容：上一個 commit 的所有異動
      適用：剛 commit 完想立即審查

  [3] 整條 feature branch（main..HEAD）
      內容：本分支自 main 分歧後累積的全部異動
      適用：feature branch 累積多個 commit 想一次審完

  [4] 指定檔案
      內容：使用者輸入檔案路徑（逗號分隔），整檔案審查
      適用：只想審查特定檔案、無 diff 比較

  [5] 全專案（不限異動）
      內容：所有受版控的程式碼檔案
      適用：初次接手專案做整體審查（耗時較久、token 消耗大）

請輸入「1」～「5」或「OKOKYES」（OKOKYES = 預設使用 [1]）。
```

#### 1.1.2 依範圍取得檔案與 diff

依使用者選擇執行對應指令：

| 選項 | 檔案清單命令 | Diff 命令 | 說明 |
|------|-------------|-----------|------|
| `[1]` | `git status --porcelain` + `git diff --name-only HEAD` | `git diff HEAD` | 工作區異動（含未 staged） |
| `[2]` | `git diff --name-only HEAD~1 HEAD` | `git diff HEAD~1 HEAD` | 最近一次 commit |
| `[3]` | `git diff --name-only main...HEAD` | `git diff main...HEAD` | 整條 branch（三點 diff） |
| `[4]` | （使用者輸入） | 無 | 整檔案審查模式 |
| `[5]` | `git ls-files \| grep -E '\.(java\|py\|jsx?\|tsx?\|html)$'` | 無 | 整檔案審查模式 |

> **注意**：選項 `[4]`、`[5]` 沒有 diff，GitHub Models 引擎會改採「整檔案餵入」模式，token 消耗較大。

#### 1.1.3 輸出審查範圍確認

```
【審查範圍確認】

📌 範圍：選項 [N] - <範圍說明>
📌 模式：<diff 審查 / 整檔案審查>

即將審查以下檔案：
├── src/main/java/xxx/XxxController.java
├── src/main/java/xxx/XxxService.java
└── src/main/resources/templates/xxx.html

共 {n} 個檔案，是否開始審查？（請輸入 OKOKYES 確認）
```

### 1.2 審查引擎選擇（強制執行）

> ⚠️ **每次審查啟動時都必須詢問引擎選擇，不得自動套用上次選擇**。

#### 1.2.1 引擎選單

```
【審查引擎選擇】

請選擇本次審查使用的引擎：

  [1] Claude CLI 內建引擎（預設）
      - 由 Claude 逐檔讀取，依 CHECKLIST.md 規則比對
      - 完全離線、不需 GitHub Token
      - 規則明確、結果穩定

  [2] GitHub Models 引擎
      - 透過 ~/Copilota/code_review.py 呼叫 GitHub Models API
      - 預設模型：openai/gpt-4o（可由環境變數 COPILOT_REVIEW_MODEL 覆寫）
      - 需設定 GITHUB_TOKEN / GH_TOKEN 環境變數
      - LLM 推理、可能發現規則外的潛在問題

請輸入「1」、「2」或「OKOKYES」（OKOKYES = 預設使用引擎 1）。
```

#### 1.2.2 GitHub Models 引擎前置檢查

若使用者選擇 `[2]`，**必須**依序檢查：

```bash
# 檢查項 1：腳本是否存在
test -f ~/Copilota/code_review.py

# 檢查項 2：Python 3
command -v python3

# 檢查項 3：環境變數（不得印出 token 內容）
test -n "${GH_TOKEN:-${GITHUB_TOKEN}}"
```

**任一項失敗** → 顯示原因 → **自動 fallback 至引擎 [1]** 並提示使用者：

```
⚠️ GitHub Models 引擎不可用：<原因>
👉 已自動 fallback 至 Claude CLI 內建引擎
   修復方式：<對應修復指引，例如 export GITHUB_TOKEN=...>
```

#### 1.2.3 引擎參數記錄

確認可用後，輸出本次審查的引擎資訊：

```
📌 本次審查引擎：GitHub Models
📌 模型：openai/gpt-4o（或 COPILOT_REVIEW_MODEL 設定值）
📌 上下文：受影響檔案 + 相關依賴檔案
```

---

### 1.3 審查執行

#### 1.3.A Claude CLI 內建引擎

依 §1.1.2 取得的檔案清單，依序讀取每個檔案，逐一檢查 [CHECKLIST.md](CHECKLIST.md) 中的項目，套用 §3 報告格式。

> **依範圍調整審查重點**：
> - 範圍 `[1]` / `[2]` / `[3]`：以 diff 為主軸，重點檢查異動行；異動行影響的鄰近邏輯也須一併確認
> - 範圍 `[4]` / `[5]`：無 diff，整檔案逐項檢查 CHECKLIST

#### 1.3.B GitHub Models 引擎

> 詳細規範見 [COPILOT.md](COPILOT.md)

**Step 1：依 §1.1.1 範圍選項收集審查素材**

```bash
# 依使用者於 §1.1.1 選擇的範圍，取對應的 diff 與檔案清單
case "$SCOPE" in
    1)  # 工作區未 commit
        DIFF_CMD="git diff HEAD"
        FILES_CMD="git diff --name-only HEAD"
        ;;
    2)  # 最近一個 commit
        DIFF_CMD="git diff HEAD~1 HEAD"
        FILES_CMD="git diff --name-only HEAD~1 HEAD"
        ;;
    3)  # 整條 branch
        DIFF_CMD="git diff main...HEAD"
        FILES_CMD="git diff --name-only main...HEAD"
        ;;
    4|5)  # 指定檔案 / 全專案 → 整檔案模式（無 diff）
        DIFF_CMD="echo '(整檔案審查模式，無 diff)'"
        # FILES 由 §1.1.2 已蒐集
        ;;
esac

CHANGED=$(eval "$FILES_CMD")
CONTEXT=$(echo "$CHANGED" | tr '\n' ',' | sed 's/,$//')

# 識別相關依賴檔案（被異動檔案 import / 繼承 / 引用的檔案）
# 由 SKILL 智慧分析（grep import/extends/include）後填入 RELATED 變數
```

**Step 2：呼叫審查腳本**

```bash
eval "$DIFF_CMD" | python3 ~/Copilota/code_review.py \
    --context "$CONTEXT" \
    --related "$RELATED" \
    > /tmp/code_review_result.json \
    2> /tmp/code_review_stderr.log

REVIEW_EXIT=$?
```

> **整檔案模式（選項 [4]/[5]）特別處理**：由於沒有 diff，`stdin` 改餵入「請審查這些檔案的完整內容」說明文字，所有檔案內容透過 `--context` 帶入。

**Step 3：依退出碼處理**

| Exit Code | 動作 |
|-----------|------|
| `0` | 解析 JSON 結果，套用 §3 報告格式呈現 |
| `1` (`missing_token`) | 提示使用者設定 `GITHUB_TOKEN` → fallback 至引擎 [1] |
| `1` (`empty_diff`) | 提示無異動，跳過審查 |
| `2` (`invalid_response`) | 顯示 raw_response → 詢問是否 fallback 至引擎 [1] |
| `3` (`http_error`) | 顯示錯誤碼 → 詢問是否 fallback 至引擎 [1] |
| `4` (`exception`) | 顯示例外資訊 → 詢問是否 fallback 至引擎 [1] |

**Step 4：報告呈現**

- 套用 §3 三級分類報告格式
- **必須**附註引擎來源：`📌 審查引擎：GitHub Models（model=<模型名稱>）`
- **必須**保留原始 JSON 摘要供使用者查閱

---

## 2. 審查項目摘要

> 完整檢查清單請參考 [CHECKLIST.md](CHECKLIST.md)

### 2.1 後端 Java 審查

#### 🔴 錯誤（必須修正）

| 類別 | 檢查項目 |
|------|---------|
| **架構違規：跨 Controller** | Controller 使用其他頁面專用的 Service |
| **架構違規：缺少專用 Service** | Controller 專用方法未放在專用 Service |
| **Lombok 禁用** | `@Data`、`@Getter`、`@Setter`、`@Builder`、`@Slf4j` 等註解 |
| **Lambda 禁用** | Lambda 表達式 `->` |
| **Stream 禁用** | `.stream()`、`.map()`、`.filter()`、`.collect()`、`.forEach()` 等 |
| **SQL Injection** | 字串拼接 SQL |
| **硬編碼密碼** | `password = "xxx"`、`secret = "xxx"` 等 |
| **不安全亂數** | `new Random()` 用於安全場景（應改 `SecureRandom`） |
| **Exception 缺少堆疊追蹤** | `log.error()` 未傳入 Exception 物件 |

#### 🟡 警告（建議修正）

- Null Check 不足
- 異常處理不當（空 catch、`printStackTrace`、吞掉例外）
- `log.error` 格式錯誤（遺失堆疊）
- 輸入驗證缺失（缺 `@Valid`）
- 註解缺失

#### 🟢 建議（可選改善）

- 命名規範
- 程式碼重複
- 方法長度（建議 < 50 行）

### 2.2 後端 Python 審查

#### 🔴 錯誤（必須修正）

| 類別 | 檢查項目 |
|------|---------|
| **架構違規：跨 Router** | Router A 直接呼叫 Router B 的內部函式 |
| **bare except** | `except:` 未指定例外類型 |
| **`print` 作為日誌** | 使用 `print()` 取代 `logging` |
| **mutable default args** | `def foo(items=[])` |
| **wildcard import** | `from xxx import *` |
| **SQL Injection** | f-string / `%` 拼接 SQL |
| **硬編碼密碼** | JWT_SECRET、API_KEY 等 |
| **遺失堆疊** | `logger.error` 未傳入 `exc_info=True` |

#### 🟡 警告

- 缺少 type hints
- 缺少 Docstring
- Null 檢查不足

### 2.3 React 前端審查

#### 🔴 錯誤（必須修正）

| 類別 | 檢查項目 |
|------|---------|
| **XSS 風險** | 使用 `dangerouslySetInnerHTML` |
| **動態執行** | 使用 `eval()`、`new Function()` |
| **DOM 注入** | 使用 `innerHTML = userInput` |
| **TypeScript 檔案** | 出現 `.tsx` / `.ts` 檔案（本專案禁用） |
| **var 宣告** | 使用 `var`（應改 `const` / `let`） |
| **Class Component** | 使用 Class 元件（應改函式元件） |

#### 🟡 警告

- 未處理 loading / error / empty 狀態
- 缺少繁體中文註解
- 表單驗證不足

### 2.4 Thymeleaf 前端審查

#### 🔴 錯誤（必須修正）

| 類別 | 檢查項目 |
|------|---------|
| **XSS 風險** | 使用 `th:utext` |
| **XSS 風險** | 手動拼接 URL 而非使用 `@{...}` |
| **CSRF 風險** | POST 表單缺少 `th:action` |
| **CSRF 風險** | AJAX POST 未帶 CSRF Header |
| **動態執行** | 使用 `eval()`、`new Function()` |

#### 🟡 警告

- Inline style（`style="..."`）
- Inline script（`onclick="..."`）
- 表單元素缺少 `<label>`
- 圖片缺少 `alt`
- 缺少註解

---

## 3. 審查報告格式

審查完成後，輸出以下格式的報告：

```
【CodeReview 審查報告】

════════════════════════════════════════════════════════════════

📊 審查摘要
├── 審查檔案數：{n}
├── 🔴 錯誤：{n} 項（必須修正）
├── 🟡 警告：{n} 項（建議修正）
└── 🟢 建議：{n} 項（可選改善）

════════════════════════════════════════════════════════════════

🔴 錯誤（必須修正）

┌──────────────────────────┬──────────┬────────────────────────────────────┐
│ 檔案:行號                │ 類型     │ 問題說明                           │
├──────────────────────────┼──────────┼────────────────────────────────────┤
│ UserService.java:45      │ Lombok   │ 發現 @Data 註解，須改為手寫 Getter │
│ UserController.java:78   │ SQL      │ 字串拼接 SQL，有注入風險           │
│ login.html:23            │ XSS      │ 使用 th:utext，應改為 th:text      │
└──────────────────────────┴──────────┴────────────────────────────────────┘

修正建議：
1. UserService.java:45 — 移除 @Data，手動撰寫 getter/setter 方法
2. UserController.java:78 — 改用 JPA Named Parameter 或 PreparedStatement
3. login.html:23 — 將 th:utext 改為 th:text

════════════════════════════════════════════════════════════════

🟡 警告（建議修正）
[同上格式]

════════════════════════════════════════════════════════════════

🟢 建議（可選改善）
[同上格式]

════════════════════════════════════════════════════════════════
```

---

## 4. 審查結果處理

### 4.1 審查通過（無 🔴 錯誤）

```
【審查結果】

✅ 審查通過！

📊 審查摘要
├── 🔴 錯誤：0 項
├── 🟡 警告：{n} 項
└── 🟢 建議：{n} 項

雖有警告與建議項目，但不影響程式正確性與安全性。
您可以選擇：
1. 輸入「OKOKYES」→ 啟動「Commit」Skill 進行規範化提交
2. 輸入「修正」→ 啟動對應的「開發-*」Skill 修正警告項目
3. 輸入其他內容 → 繼續對話
```

### 4.2 審查失敗（有 🔴 錯誤）

```
【審查結果】

❌ 審查未通過

📊 審查摘要
├── 🔴 錯誤：{n} 項（必須修正）
├── 🟡 警告：{n} 項
└── 🟢 建議：{n} 項

發現 {n} 項必須修正的錯誤，請先完成修正後再繼續流程。

請輸入「OKOKYES」啟動對應的「開發-*」Skill 進行修正。
```

---

## 5. Skill 串接機制（強制執行）

### 5.1 審查通過 → 原始碼掃描（SAST）

```
【Skill 串接通知】

✅ CodeReview 審查通過
✅ 無必須修正的錯誤項目
✅ 已找到對應的 Skill：「原始碼掃描」(/原始碼掃描)

此 Skill 將協助您進行：
- 依 OWASP Top 10 2021 進行靜態資安掃描
- 使用 Semgrep / Bandit / SpotBugs / ESLint Security 等工具
- 產出含嚴重度、OWASP 分類、修補建議的資安報告
- 掃描通過後可繼續串接「網頁弱點掃描」與「滲透測試」

請輸入「OKOKYES」啟動「原始碼掃描」Skill，或輸入「跳過」直接進入「Commit」，或輸入其他內容繼續對話。
```

### 5.2 原始碼掃描通過 / 使用者跳過 → Commit

```
【Skill 串接通知】

✅ 已找到對應的 Skill：「Commit」(/Commit)

此 Skill 將協助您進行：
- 同步主線並處理衝突
- 分析變更內容並依單一職責原則拆分 commit
- 產生規範化的繁體中文 commit message
- Push 到遠端分支

請輸入「OKOKYES」啟動「Commit」Skill，或輸入其他內容繼續對話。
```

### 5.3 審查失敗 → 開發

```
【Skill 串接通知】

❌ CodeReview 審查未通過
❌ 發現 {n} 項必須修正的錯誤
✅ 已找到對應的 Skill：「<開發 SKILL 名稱>」

此 Skill 將協助您進行：
- 根據審查報告修正錯誤項目
- 確保符合程式碼規範
- 修正完成後將再次觸發 CodeReview

請輸入「OKOKYES」啟動修正流程，或輸入其他內容繼續對話。
```

---

## 6. 禁止行為

### 6.1 流程紅線

- ❌ 禁止未執行審查就宣告通過
- ❌ 禁止忽略 🔴 錯誤項目
- ❌ 禁止未告知就自動啟動下一個 Skill
- ❌ 禁止跳過「已找到 Skill」的通知
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動 Skill
- ❌ 禁止跳過「引擎選擇」步驟自動套用某一引擎
- ❌ 禁止記憶上次的引擎選擇（每次都必須詢問）

### 6.2 安全紅線（GitHub Models 引擎）

- ❌ **絕對禁止**將 `GITHUB_TOKEN` / `GH_TOKEN` 寫入任何檔案（SKILL、settings.json、commit log、註解）
- ❌ **絕對禁止**在訊息或日誌中印出 token 完整字串（即使部分截斷亦不可）
- ❌ **絕對禁止**將使用者程式碼上傳至 GitHub Models 以外的服務
- ❌ **絕對禁止**要求使用者在對話中貼出 token 內容
- ✅ **必須**僅從環境變數讀取 token
- ✅ **必須**在 token 不存在時，引導使用者自行設定（提供命令範例，但不接收使用者貼出的 token）

---

## 7. 相關文件

- [CHECKLIST.md](CHECKLIST.md) — 完整審查檢查清單與正則表達式（Claude 引擎使用）
- [COPILOT.md](COPILOT.md) — GitHub Models 引擎使用規範
- [_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md) — 共享編碼規範
- [_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md) — 共享資安規範
- `~/Copilota/code_review.py` — GitHub Models 審查腳本
