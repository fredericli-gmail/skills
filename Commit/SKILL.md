---
name: Commit
description: "Git Commit 流程規範。提交前必須同步主線並處理衝突，自動分析變更內容，依單一職責原則拆分 commit，產生規範化的繁體中文 commit message，最後 push 到遠端 feature branch。當 CodeReview 審查通過後觸發，完成後串接「測試」Skill。當用戶說「commit」、「提交」、「幫我 commit」、「版控」、「push」時亦可手動觸發。確認後使用者必須輸入 OKOKYES 才會執行 commit。"
---

# Skill: Git Commit 流程規範

## Usage Trigger

- 當「CodeReview」Skill 審查通過後，使用者輸入 `OKOKYES` 觸發
- 當使用者主動要求提交程式碼時（「commit」、「提交」、「幫我 commit」、「版控」、「push」）
- 當使用者輸入 `/Commit` 時啟用

---

## 完整流程總覽

```
Phase 0：建立 / 切換 feature branch + 同步主線
    ↓
Phase 1：分析變更
    ↓
Phase 2：變更分組（等待 OKOKYES 確認）
    ↓
Phase 3：依序執行 Commit
    ↓
Phase 4：Push 到遠端 feature branch
    ↓
Phase 5：完成報告 + Skill 串接
```

> **註**：本 SKILL 不負責部署。若需要部署到 DEV 環境，請改用 `/上版` SKILL。

---

## 0. 建立 feature branch + 同步主線

> **強烈建議**：所有開發在 feature branch 上進行，禁止直接在 main 上 commit 並 push。

### 0.1 判斷目前分支

```bash
git branch --show-current
git remote -v
```

### 0.2 建立 feature branch（若目前在 main / master）

```bash
# 同步主線
git fetch origin
git pull origin main    # 或 master

# 建立開發分支
git checkout -b feature/功能描述
```

### 0.3 分支命名規則

| 類型 | 命名格式 | 範例 |
|------|---------|------|
| 新功能 | `feature/<功能描述>` | `feature/user-login` |
| Bug 修正 | `fix/<修復描述>` | `fix/login-redirect-error` |
| 重構 | `refactor/<重構描述>` | `refactor/auth-service` |
| 設定 | `config/<設定描述>` | `config/db-connection-pool` |
| 文件 | `docs/<文件描述>` | `docs/readme-update` |

### 0.4 同步主線（若已在 feature branch）

```bash
git fetch origin
git merge origin/main      # 或 origin/master
# → 若有衝突，必須先解完衝突才能繼續
```

### 0.5 衝突處理

- 若 merge 產生衝突，**必須先解決衝突**再進行 commit
- 解衝突後先 `git add <衝突檔案>`，再 `git commit` 完成 merge commit
- ❌ **禁止跳過衝突直接 push**

### 0.6 同步完成確認

```
【分支準備】
✅ 已同步 origin/main（無衝突）
✅ 目前分支：feature/xxx
可開始進行 commit。
```

---

## 1. Commit 核心原則

### 1.1 單一職責原則（最高原則）

> 每一個 commit 只做一件事。禁止將多個不相關的變更混在同一個 commit 中。

| 正確做法 | 錯誤做法 |
|---------|---------|
| commit 1: 新增 Entity + Repository | 把新功能、bug 修正、重構全部塞在同一個 commit |
| commit 2: 新增 Service 業務邏輯 | 一次 commit 包含 10 個不相關的檔案異動 |
| commit 3: 新增 Controller API 端點 | 「先 commit 再說」然後用 amend 補東補西 |

### 1.2 Commit 必須可編譯 / 可執行

> 每一個 commit 都必須是可獨立編譯（Java）或語法正確（Python / JS）的狀態。禁止提交會導致編譯失敗的中間狀態。

| 技術棧 | 驗證方式 |
|-------|---------|
| **Java** | `./mvnw compile -q` |
| **Python** | `python -m py_compile <file.py>` |
| **JavaScript** | `node --check <file.js>` |
| **React** | `npm run build` |

### 1.3 Commit 粒度

| 粒度 | 適用情境 | 範例 |
|------|---------|------|
| **檔案級** | 設定檔異動、獨立配置變更 | 新增資料庫連線設定 |
| **功能級** | 一個完整功能模組（含 Entity / Service / Controller） | 新增使用者建立功能 |
| **邏輯級** | 分析報告中的一個「邏輯階段」 | 階段一：資料層 |
| **修正級** | 單一 bug 修正 | 修正登入頁面驗證錯誤 |

---

## 2. Commit Message 格式

### 2.1 格式規範

```
<type>: <subject>

<body>（可選，大型變更時使用）

Co-Authored-By: Claude <noreply@anthropic.com>
```

### 2.2 Type 分類

| Type | 使用時機 | 範例 |
|------|---------|------|
| `feat` | **新增**全新的功能、API、頁面、元件 | `feat: 新增使用者建立 API` |
| `fix` | **修正** bug、錯誤行為、資料異常 | `fix: 修正登入頁面驗證錯誤` |
| `refactor` | **重構**程式碼結構，不改變外部行為 | `refactor: 將 UserService 拆分為 UserAuthService` |
| `docs` | **文件**異動（README、註解） | `docs: 更新 README 專案說明` |
| `style` | **格式**調整（縮排、空白、分號），不影響邏輯 | `style: 統一 Controller 縮排格式` |
| `test` | **測試**相關（新增、修改、刪除測試） | `test: 新增使用者管理 E2E 測試` |
| `chore` | **雜務**（建置設定、CI/CD、套件更新、啟動腳本） | `chore: 更新 startup script 環境變數` |
| `perf` | **效能**優化 | `perf: 優化使用者清單查詢加入分頁索引` |
| `revert` | **回退**先前的 commit | `revert: 回退 feat: 新增使用者建立 API` |
| `db` | **資料庫** changelog 異動 | `db: 新增 user 資料表` |
| `config` | **設定檔**異動（YAML、環境變數） | `config: 新增資料庫連線池設定` |

### 2.3 Subject 撰寫規範

| 規則 | 說明 |
|------|------|
| **語言** | 繁體中文 |
| **動詞開頭** | 以動作描述開頭：`新增`、`修正`、`移除`、`更新`、`重構`、`優化` |
| **長度限制** | Subject 不超過 30 字 |
| **不加句號** | 結尾不加標點符號 |
| **說明做了什麼** | 描述變更內容而非原因 |

### 2.4 Body 撰寫（大型變更時使用）

當 commit 涉及多個檔案或需要解釋「為什麼」時，加上 body：

```
feat: 新增使用者管理功能

- UserController: 新增 CRUD API 端點
- UserService: 實作業務邏輯與權限控制
- UserRepository: 新增 JPA Repository
- 對應 Liquibase changelog: 新增 user 資料表

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## 3. 版控執行流程

### 3.1 Phase 0：同步主線

依照第 0 節流程同步 `origin/main`，確認無衝突後繼續。

### 3.2 Phase 1：分析變更

```bash
# 查看所有變更檔案
git status

# 查看具體差異
git diff
git diff --staged

# 查看最近的 commit 歷史（學習風格）
git log --oneline -10
```

### 3.3 Phase 2：變更分組

依**單一職責原則**將變更檔案分組，輸出方案：

```
【變更分組分析】

本次共有 {n} 個檔案異動，建議拆分為 {m} 個 commit：

+-------------------------------------------------------+
| Commit 1: config: 新增資料庫連線池設定                  |
| ├── src/main/resources/application.yml                  |
| └── 理由：獨立的設定檔異動，不依賴其他變更               |
+-------------------------------------------------------+
| Commit 2: feat: 新增使用者管理功能                      |
| ├── src/main/java/.../UserController.java               |
| ├── src/main/java/.../UserService.java                  |
| ├── src/main/java/.../UserRepository.java               |
| └── 理由：完整的使用者 CRUD 功能模組                     |
+-------------------------------------------------------+

請確認分組方案，輸入「OKOKYES」開始依序 commit。
```

⏸️ **停止點：等待使用者輸入 `OKOKYES` 確認後，才開始執行 commit。**

### 3.4 Phase 3：依序執行 Commit

使用者確認後，依序執行每個 commit：

1. **Stage 檔案**：只加入該 commit 相關的檔案（明確指定）
2. **建立 commit**：使用 HEREDOC 格式撰寫 commit message
3. **驗證結果**：執行 `git status` 確認

```bash
# 正確：指定具體檔案
git add src/main/resources/application.yml

# 正確：使用 HEREDOC 格式
git commit -m "$(cat <<'EOF'
config: 新增資料庫連線池設定

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

# ❌ 錯誤：禁止使用
git add .
git add -A
```

### 3.5 Phase 4：Push 到遠端 feature branch

所有 commit 完成後，推送到遠端 feature branch：

```bash
git push origin <feature-branch-name>
# 例：git push origin feature/user-login
```

**Push 規範**：
- 推送前再次確認分支名稱正確（應為 feature branch，**不是 main**）
- ❌ 禁止 `git push --force`（除非使用者明確要求）
- ❌ 禁止推送到非目前分支的遠端分支

### 3.6 Phase 5：完成報告

```
【版控完成報告】

✅ 主線同步：已合併 origin/main（無衝突）
✅ 共建立 {n} 個 commit：

| # | Hash    | Type   | Message |
|---|---------|--------|---------|
| 1 | abc1234 | config | 新增資料庫連線池設定 |
| 2 | def5678 | feat   | 新增使用者管理功能 |

✅ 已推送至 origin/<feature-branch>
```

---

## 4. 特殊情境處理

### 4.1 混合變更的拆分策略

| 情境 | 處理方式 |
|------|---------|
| 一個檔案有 feat + fix | 若可拆分（不同方法），使用 `git add -p` 分段 stage；若無法拆分，歸類為主要變更的 type |
| 設定檔 + 程式碼 | 設定檔獨立一個 commit，程式碼另一個 |
| Entity + Service + Controller | 若為同一功能，合併為一個 `feat` commit |
| 前端 + 後端 | 同一功能可合併；不同功能必須分開 |

### 4.2 分析報告中的邏輯階段

若開發是依照「分析」Skill 的邏輯階段進行，每個邏輯階段應對應一個 commit：

```
分析報告邏輯階段          →    Commit
─────────────────────────────────────
階段一：資料層             →    feat: 新增 xxx Entity 與 Repository
階段二：業務邏輯層         →    feat: 新增 xxx Service 業務邏輯
階段三：API 端點           →    feat: 新增 xxx Controller API 端點
階段四：前端頁面           →    feat: 新增 xxx 管理頁面
```

### 4.3 Pre-commit Hook 失敗處理

若 commit 因 pre-commit hook 失敗：

1. **分析失敗原因**：讀取 hook 輸出的錯誤訊息
2. **修正問題**：依據錯誤訊息修正程式碼
3. **重新 stage**：`git add` 修正後的檔案
4. **建立新 commit**：使用 `git commit`（**禁止使用 `--amend`**，因為前一次 commit 未成功）

### 4.4 無變更時的處理

若 `git status` 顯示沒有任何變更：

```
【版控結果】

ℹ️ 目前沒有待提交的變更（工作目錄乾淨）。

可能原因：
- 變更已在先前的 commit 中提交
- 開發階段已自動 commit

無需執行 commit，直接進入下一步。
```

---

## 5. 禁止行為

| 禁止事項 | 原因 |
|---------|------|
| `git add .` 或 `git add -A` | 可能意外加入敏感檔案或不相關的變更 |
| `git commit --amend` | 除非使用者明確要求，否則永遠建立新 commit |
| `git push --force` | 可能覆蓋他人的工作 |
| `git commit --no-verify` | 不可跳過 pre-commit hook |
| 英文 commit message | 強制使用繁體中文 |
| 多件事混在一個 commit | 違反單一職責原則 |
| 未確認就自動 commit | 必須等待使用者輸入 `OKOKYES` 確認分組方案 |
| commit 含有敏感檔案 | `.env`、密碼、金鑰等必須警告使用者 |
| 使用 `-i` 互動模式 | `git rebase -i`、`git add -i` 不支援 |
| 跳過同步主線直接 push | 必須先 fetch + merge |
| 未解決衝突就 push | 衝突必須先解決 |
| commit message 使用無意義訊息 | 禁止 `update`、`fix`、`修改` 等模糊訊息 |
| 直接在 main 上 commit | 必須開 feature branch |

---

## 6. 品質檢查清單

在每次提交前，依序確認：

- [ ] 已 `git fetch origin`
- [ ] 已 `git merge origin/main`（若在開發分支）
- [ ] 衝突已全部解決
- [ ] commit 只包含一件事（單一職責）
- [ ] commit message 符合 `<type>: <subject>` 格式
- [ ] subject 使用繁體中文、動詞開頭、不超過 30 字
- [ ] type 分類正確
- [ ] 沒有加入敏感檔案（`.env`、密碼、金鑰）
- [ ] 沒有加入不相關的檔案（IDE 設定、暫存檔）
- [ ] commit 後專案仍可編譯
- [ ] Push 到正確的 feature branch

---

## 7. Commit Message 範例集

### 後端功能

```
feat: 新增使用者建立 API 與表單驗證
feat: 實作訂單付款狀態同步功能
fix: 修正使用者列表排序方向錯誤
refactor: 將 NotificationService 拆分為 EmailService 與 SmsService
```

### 前端功能

```
feat: 新增使用者管理列表頁面（含篩選、分頁、批次操作）
fix: 修正深色模式下表格邊框顏色異常
fix: 修正下拉選單被表格容器裁切遮蔽
style: 統一表單元件 Padding 設定
```

### 設定與部署

```
config: 新增資料庫連線池設定
chore: 更新建置腳本環境變數
chore: 升級 Spring Boot 至 3.2.0
```

### 資料庫

```
db: 新增 user 資料表與索引
db: users 資料表新增 last_login_at 欄位
db: 修正 permissions 外鍵約束名稱衝突
```

---

## 8. Skill 串接機制（強制執行）

### 8.1 版控完成 → 測試

所有 commit + push 完成後，必須輸出以下串接通知：

```
【Skill 串接通知】

✅ 版控階段已完成
✅ 共建立 {n} 個規範化 commit
✅ 已推送至 origin/<feature-branch>
✅ 已找到對應的 Skill：「測試」(/測試)

此 Skill 將協助您進行：
- Playwright 自動化測試
- 使用實體瀏覽器（非 headless）
- 元素定位掃描與測試撰寫

請輸入「OKOKYES」啟動「測試」Skill，或輸入其他內容繼續對話。
```

**等待使用者確認**：
- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/測試`
- ❌ 使用者輸入其他內容 → 不啟動，繼續正常對話

### 8.2 無變更時 → 測試

若沒有待提交的變更（工作目錄乾淨），同樣串接至測試：

```
【Skill 串接通知】

ℹ️ 無待提交的變更（已在先前提交完成）
✅ 已找到對應的 Skill：「測試」(/測試)

請輸入「OKOKYES」啟動「測試」Skill，或輸入其他內容繼續對話。
```

---

## 9. 禁止行為（Skill 層級）

- ❌ 禁止未同步主線就直接 commit
- ❌ 禁止未分析變更就直接 commit
- ❌ 禁止未等待 `OKOKYES` 確認就執行 commit
- ❌ 禁止未告知就自動啟動下一個 Skill
- ❌ 禁止跳過「已找到 Skill」的通知
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動 Skill
- ❌ 禁止在 commit 失敗時提示串接測試 Skill
