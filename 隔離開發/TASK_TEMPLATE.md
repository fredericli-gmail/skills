# TASK.md 模板

> 此模板供「隔離開發」Skill 產生 TASK.md 時參考使用。
> 請將 `{變數}` 替換為實際內容。

---

```markdown
# 開發任務：{任務標題}

> ⚠️ **Claude 請注意**：這是由「隔離開發」Skill 自動產生的任務檔案。
> 當用戶輸入「開始」、「執行」或「請讀取 TASK.md」時，請：
> 1. 完整讀取此檔案
> 2. 確認理解任務內容
> 3. 依照「執行計畫」逐階段開發
> 4. 每階段完成後 commit + push
> 5. 所有階段完成後，依照「開發完成後流程」進行

---

## 需求摘要

{一段話描述核心需求，包含背景與目標}

---

## 功能需求

- [ ] {功能需求 1}
- [ ] {功能需求 2}
- [ ] {功能需求 3}

---

## 執行計畫

### 階段 1：{階段名稱}

**目標**：{此階段要達成的目標}

**檔案清單**：
| 檔案路徑 | 操作 | 說明 |
|---------|------|------|
| `{路徑}` | 新增/修改 | {修改內容說明} |
| `{路徑}` | 新增/修改 | {修改內容說明} |

**完成標準**：
- {如何判斷此階段完成}

**Commit 訊息**：`feat: {commit 說明}`

---

### 階段 2：{階段名稱}

**目標**：{此階段要達成的目標}

**檔案清單**：
| 檔案路徑 | 操作 | 說明 |
|---------|------|------|
| `{路徑}` | 新增/修改 | {修改內容說明} |

**完成標準**：
- {如何判斷此階段完成}

**Commit 訊息**：`feat: {commit 說明}`

---

### 階段 N：{階段名稱}

（依此類推）

---

## 技術規範

### 程式碼風格
- 禁用 Lombok - 手動撰寫 getter、setter、constructor
- 禁用 Lambda - 使用傳統迴圈或匿名內部類別
- 禁用 Stream API - 使用傳統迴圈處理集合
- 每行程式碼加繁體中文註解

### Log 日誌規範
- 所有 log 必須加上中文功能前綴，格式：`[中文功能名稱]`
- 結構化參數格式：`key=value`，以 `|` 分隔
- `logger.error()` 必須傳入 Exception 物件

### 資安規範
- 防止 XSS：輸出時進行 HTML 編碼
- 防止 CSRF：確保表單包含 CSRF Token
- 防止 SQL Injection：使用參數化查詢，禁止字串拼接

---

## Commit 規範

1. **以邏輯階段為單位**：同一階段的檔案必須一起 commit
2. **格式**：
   - 新功能：`feat: {說明}`
   - 問題修正：`fix: {說明}`
   - 進行中：`wip: {說明}`
3. **每次 commit 後立即 push**

---

## 完成後通知

每完成一個階段，請輸出：

```
✅ 階段 {N}：{階段名稱} 已完成

已執行：
- git add {檔案清單}
- git commit -m "feat: {說明}"
- git push origin {branch_name}

您可以在主目錄執行以下指令測試：
  cd /Users/fredericli/Git/Danova/Danova-woms
  git fetch origin
  git checkout {branch_name}
  ./mvnw spring-boot:run

📌 提醒：此分支完成後應合併回「{base_branch}」分支
```

---

## 任務資訊

| 項目 | 值 |
|------|-----|
| 建立時間 | {YYYY-MM-DD HH:mm} |
| 分支名稱 | {branch_name} |
| 基底分支 | {base_branch} |
| Worktree 路徑 | ./sub-danova-worm/{branch_name} |

> **重要**：開發完成後，此分支應合併回「{base_branch}」分支。

---

## 開始執行

收到「開始」指令後，請：
1. 確認已讀取並理解此任務
2. 從階段 1 開始執行
3. 每階段完成後 commit + push + 通知用戶

---

## 開發完成後流程（強制執行）

> ⚠️ 所有階段完成後，必須依照以下流程進行後續品質檢查。

### 完成條件
- [ ] 所有階段的程式碼已撰寫完成
- [ ] 所有 commit 已完成並 push
- [ ] 專案可正常編譯（`./mvnw compile` 成功）

### Skill 串接流程

```
開發完成
    │
    ▼
┌─────────────────────────────────────┐
│ CodeReview Skill                    │
│ - 程式碼品質審查                     │
│ - 禁用語法檢查                       │
│ - 資安規範審查                       │
│ - 註解完整度檢查                     │
└─────────────────────────────────────┘
    │
    ├── 通過 ──→ 測試 Skill
    │            - Selenium 自動化測試
    │            - 功能驗證
    │
    └── 失敗 ──→ 回到開發修正
                 - 依審查報告修正
                 - 修正後再次 CodeReview
```

### 觸發方式

當所有開發階段完成後，請輸出：

```
【開發完成通知】

✅ 所有開發階段已完成
✅ 已 commit 並 push 至遠端
✅ 專案編譯成功

即將進入程式碼審查階段。
請輸入「OKOKYES」啟動「CodeReview」Skill。
```

等待用戶輸入「OKOKYES」後，執行 `/CodeReview` skill。

---

## 測試通過後：合併與清理（最終步驟）

> ⚠️ 當 CodeReview 通過且測試通過後，輸出以下通知：

```
【隔離開發流程完成通知】

✅ 分析階段 → 已完成
✅ 開發階段 → 已完成
✅ CodeReview → 已通過
✅ 測試階段 → 已通過

🎉 開發流程全部完成！

📌 請執行以下步驟完成合併與清理：

1. 回到主目錄：
   cd /Users/fredericli/Git/Danova/Danova-woms

2. 切回基底分支：
   git checkout {base_branch}

3. 合併功能分支：
   git merge {branch_name}

4. 推送到遠端：
   git push origin {base_branch}

5. 刪除 worktree：
   git worktree remove ./sub-danova-worm/{branch_name}

6. 刪除分支（可選）：
   git branch -d {branch_name}
   git push origin --delete {branch_name}
```
```
