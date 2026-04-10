---
name: CodeReview
description: "程式碼審查專家。於開發完成後自動審查程式碼品質，檢查項目包含：程式碼風格（禁用 Lombok/Lambda/Stream）、繁體中文註解完整度、資安規範（XSS、CSRF、SQL Injection）、Null Check 與異常處理、邏輯正確性。產出表格式分類報告，依嚴重程度分級（錯誤、警告、建議）。審查通過後串接「Commit」Skill 進行規範化提交，發現問題則串接對應的開發 Skill 進行修正。"
---

# Skill: CodeReview（程式碼審查專家）

> ⚠️ **本 SKILL 引用共享規範**：
> - 編碼規範：[_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md)
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)
> - 完整檢查項目：[CHECKLIST.md](CHECKLIST.md)

## Usage Trigger

- 當任一「開發-*」Skill 完成且編譯/執行成功後自動觸發
- 當使用者要求進行程式碼審查時
- 當需要檢查程式碼是否符合專案規範時

---

## 1. 審查流程

### 1.1 審查前置作業

1. **確認審查範圍**：
   - 預設：審查本次開發階段異動的所有檔案
   - 可選：使用者指定特定檔案進行審查

2. **取得異動檔案清單**：
   ```bash
   git diff --name-only HEAD~1
   git status --porcelain
   ```

3. **輸出審查範圍確認**：
   ```
   【審查範圍確認】

   即將審查以下檔案：
   ├── src/main/java/xxx/XxxController.java
   ├── src/main/java/xxx/XxxService.java
   └── src/main/resources/templates/xxx.html

   共 {n} 個檔案，是否開始審查？
   ```

### 1.2 審查執行

依序讀取每個檔案，逐一檢查 [CHECKLIST.md](CHECKLIST.md) 中的項目。

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

### 5.1 審查通過 → Commit

```
【Skill 串接通知】

✅ CodeReview 審查通過
✅ 無必須修正的錯誤項目
✅ 已找到對應的 Skill：「Commit」(/Commit)

此 Skill 將協助您進行：
- 同步主線並處理衝突
- 分析變更內容並依單一職責原則拆分 commit
- 產生規範化的繁體中文 commit message
- Push 到遠端分支

請輸入「OKOKYES」啟動「Commit」Skill，或輸入其他內容繼續對話。
```

### 5.2 審查失敗 → 開發

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

- ❌ 禁止未執行審查就宣告通過
- ❌ 禁止忽略 🔴 錯誤項目
- ❌ 禁止未告知就自動啟動下一個 Skill
- ❌ 禁止跳過「已找到 Skill」的通知
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動 Skill

---

## 7. 相關文件

- [CHECKLIST.md](CHECKLIST.md) — 完整審查檢查清單與正則表達式
- [_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md) — 共享編碼規範
- [_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md) — 共享資安規範
