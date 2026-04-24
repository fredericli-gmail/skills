---
name: Bug管理
description: "Azure DevOps Bug 單管理工具。支援拉取 Bug 清單、匯入待辦、回寫修正結果與截圖、同步狀態。當用戶說「/Bug管理」、「/bug」、「拉Bug單」、「回寫Bug」、「同步Bug狀態」時觸發。"
---

# Skill: Azure DevOps Bug 管理

## Usage Trigger

- 當用戶輸入「/Bug管理」、「/bug」時觸發
- 當用戶說「拉 Bug 單」、「下載 Bug」、「同步 Bug」、「回寫 Bug」時觸發
- 當 Commit 或測試 Skill 完成後，主動提醒用戶是否回寫 Azure DevOps

---

## 0. 連線設定

### 0.1 環境變數（必要）

> **禁止在 Skill 檔案或 Memory 中儲存 PAT！** 一律透過環境變數讀取。

| 環境變數 | 說明 | 範例 |
|---------|------|------|
| `AZURE_DEVOPS_PAT` | Personal Access Token（Scope: Work Items Read+Write） | `xxx...xxx` |
| `AZURE_DEVOPS_ORG` | 組織名稱（選填，預設從 URL 解析） | `myorg` |
| `AZURE_DEVOPS_PROJECT` | 專案名稱（選填，預設從 URL 解析） | `MyProject` |

**首次使用設定方式：**

```bash
# 方式一：在 shell 設定（推薦，加入 .zshrc 或 .bashrc）
export AZURE_DEVOPS_PAT="你的PAT"

# 方式二：執行時臨時帶入
AZURE_DEVOPS_PAT="你的PAT" claude

# 方式三：使用者直接貼 PAT，Skill 暫存於當次對話（不持久化）
```

### 0.2 PAT 產生方式

1. Azure DevOps 右上角頭像 → **Personal access tokens** → **+ New Token**
2. Scope 勾選：
   - **Work Items → Read & Write**（讀取 + 回寫 Bug 單）
3. 複製 Token 設定到環境變數

### 0.3 連線驗證

每次執行前自動驗證 PAT 有效性：

```bash
curl -s -u ":${AZURE_DEVOPS_PAT}" "https://dev.azure.com/${ORG}/_apis/projects?api-version=7.1" | head -1
```

若回傳 401/403，提示使用者更新 PAT。

### 0.4 自動偵測專案狀態選項（首次 pull 時執行）

> **不同專案的 Work Item State 名稱不同**（例：某專案用 `1-尚未啟動`、`3-已完成`；其他專案可能用 `New`、`Active`、`Closed`）。
> 首次對某個 org/project 執行 pull 時，自動偵測並快取狀態選項。

**偵測 API：**

```bash
# 取得 Bug Work Item Type 的所有合法 State
GET https://dev.azure.com/{org}/{project}/_apis/wit/workitemtypes/Bug/states?api-version=7.1
Authorization: Basic :{PAT}
```

**回傳範例：**
```json
{
  "count": 4,
  "value": [
    { "name": "New", "color": "b2b2b2", "category": "Proposed" },
    { "name": "Active", "color": "007acc", "category": "InProgress" },
    { "name": "Resolved", "color": "ff9d00", "category": "Resolved" },
    { "name": "Closed", "color": "339933", "category": "Completed" }
  ]
}
```

**狀態映射規則（依 category 自動對應）：**

| 待辦狀態 | Azure DevOps State Category | 說明 |
|---------|---------------------------|------|
| ⬜ 待辦 | `Proposed` | 找 category 為 Proposed 的第一個 State |
| 🔄 進行中 | `InProgress` | 找 category 為 InProgress 的第一個 State |
| ✅ 完成 | `Completed` 或 `Resolved` | 優先 Completed，沒有則用 Resolved |

**快取位置：**

偵測結果暫存在 `azure-devops-bugs.json` 的 `_meta` 欄位中：

```json
{
  "_meta": {
    "org": "myorg",
    "project": "MyProject",
    "queryId": "2f824d02-...",
    "pulledAt": "2026-04-16",
    "stateMapping": {
      "proposed": "1-尚未啟動",
      "inProgress": "2-進行中",
      "completed": "3-已完成"
    }
  },
  "bugs": [ ... ]
}
```

**若偵測失敗（API 不支援或權限不足）：**

提示使用者手動指定：

```
⚠️ 無法自動偵測此專案的狀態選項。
請輸入此專案的「已完成」狀態名稱（例：Closed、Done、3-已完成）：
```

---

## 1. 子指令總覽

| 子指令 | 用法 | 說明 |
|--------|------|------|
| （無參數） | `/Bug管理` | 顯示目前 Bug 清單狀態（從待辦讀取） |
| `pull` | `/Bug管理 pull <查詢URL>` | 從 Azure DevOps 拉取 Bug 單 |
| `sync` | `/Bug管理 sync` | 同步待辦完成狀態回 Azure DevOps |
| `close` | `/Bug管理 close <BUG_ID>` | 關閉指定 Bug + 貼修正留言 |
| `comment` | `/Bug管理 comment <BUG_ID> [內容] [圖片路徑]` | 對指定 Bug 貼留言（支援文字、截圖、或兩者混合） |

---

## 2. 子指令：pull（拉取 Bug 單）

### 2.1 輸入

使用者提供 Azure DevOps **查詢頁面 URL** 或 **Query ID**：

```
/Bug管理 pull https://dev.azure.com/{org}/{project}/_queries/query/{queryId}
```

### 2.2 執行流程

```
Step 1: 從 URL 解析 org / project / queryId
    ↓
Step 2: 呼叫 WIQL API 取得 Work Item ID 列表
    ↓
Step 3: 批次取得所有 Work Item 詳情（Title / State / Description / Priority）
    ↓
Step 4: 取得每個 Work Item 的 Comments
    ↓
Step 5: 儲存 JSON 到專案根目錄（azure-devops-bugs.json）
    ↓
Step 6: 匯入待辦清單（串接 /待辦事項）
    ↓
Step 7: 輸出摘要報告
```

### 2.3 API 呼叫

```bash
# Step 2: 執行查詢
GET https://dev.azure.com/{org}/{project}/_apis/wit/wiql/{queryId}?api-version=7.1
Authorization: Basic :{PAT}

# Step 3: 批次取得 Work Items（最多 200 個一批）
GET https://dev.azure.com/{org}/{project}/_apis/wit/workitems?ids=1,2,3&$expand=all&api-version=7.1

# Step 4: 取得留言
GET https://dev.azure.com/{org}/{project}/_apis/wit/workitems/{id}/comments?api-version=7.1-preview.4
```

### 2.4 JSON 儲存格式

```json
[
  {
    "id": 50021,
    "title": "#mind-首頁: 紅色數字用途不明確...",
    "state": "1-尚未啟動",
    "assigned": "指派人員",
    "priority": 3,
    "description": "...",
    "comments": [
      {"author": "盧佳柔", "text": "<<Steps>> ..."}
    ],
    "area": "MyProject",
    "iteration": "MyProject\\Sprint1"
  }
]
```

### 2.5 匯入待辦清單

拉取完成後，自動依以下規則分類並寫入 `todo_backlog.md`：

| 條件 | 分類 | 提醒 |
|------|------|------|
| 功能不運作（關鍵字：不運作、空白、錯誤、無法） | 🔴 功能 Bug | |
| UI 調整（關鍵字：Dark、顯示、label、icon、按鈕） | 🟢 簡單 | |
| 需要前後端（關鍵字：儲存、驗證、API） | 🟡 可開始 | |
| 其他 | 🟡 可開始 | |

### 2.6 輸出格式

```
【Bug 單拉取完成】

✅ 來源：{查詢名稱}（{org}/{project}）
✅ 共 {n} 張 Bug，已儲存至 azure-devops-bugs.json
✅ 已匯入待辦清單

| 分類 | 數量 |
|------|------|
| 🔴 功能 Bug | {n} |
| 🟡 可開始 | {n} |
| 🟢 簡單 | {n} |

建議從 🔴 功能 Bug 開始修。
```

---

## 3. 子指令：sync（同步狀態）

### 3.1 執行流程

```
Step 1: 讀取待辦清單中的 Azure DevOps Bug 項目
    ↓
Step 2: 比對每個 Bug 的待辦狀態 vs Azure DevOps 狀態
    ↓
Step 3: 對已完成的 Bug，呼叫 API 更新 State
    ↓
Step 4: 對已完成的 Bug，自動貼修正留言（若尚未貼過）
    ↓
Step 5: 輸出同步結果
```

### 3.2 狀態對應（動態，依 0.4 偵測結果）

> **狀態名稱不寫死**，從 `azure-devops-bugs.json` 的 `_meta.stateMapping` 讀取。

| 待辦狀態 | State Category | 範例 A | 範例（通用） |
|---------|---------------|---------------|-------------|
| ⬜ 待辦 | Proposed | 1-尚未啟動 | New |
| 🔄 進行中 | InProgress | 2-進行中 | Active |
| ✅ 完成 | Completed | 3-已完成 | Closed |

### 3.3 更新 State API

```bash
# 更新 Work Item 欄位（State 名稱從 stateMapping 動態取得）
PATCH https://dev.azure.com/{org}/{project}/_apis/wit/workitems/{id}?api-version=7.1
Content-Type: application/json-patch+json

[
  {
    "op": "replace",
    "path": "/fields/System.State",
    "value": "{stateMapping.completed}"
  }
]
```

### 3.4 輸出格式

```
【Bug 狀態同步完成】

✅ 同步 {n} 張 Bug 到 Azure DevOps

| BUG ID | 標題 | 舊狀態 | 新狀態 |
|--------|------|--------|--------|
| 50021 | 紅色數字... | {stateMapping.proposed} | {stateMapping.completed} |
```

---

## 4. 子指令：close（關閉 Bug）

> **close 是高風險操作**，通常由測試組驗證通過後才執行。必須經過確認步驟。

### 4.1 用法

```
/Bug管理 close 50021
/Bug管理 close 50021,50024,50025    # 批次關閉
```

### 4.2 執行流程

```
Step 1: 檢查前置條件
    ├── 無 commit → 阻擋：「BUG {id} 尚無任何 commit，請先修完」
    ├── 有 commit 但無測試截圖 → 警告：「建議先執行 /測試」
    └── 有 commit + 有截圖 → 繼續
    ↓
Step 2: 顯示確認提示（強制）
    ↓
Step 3: 使用者輸入「確認關閉」後才執行
    ↓
Step 4: 從 git log 組合修正留言
    ↓
Step 5: 貼 Discussion 留言
    ↓
Step 6: 更新 State 為已完成（依 stateMapping）
    ↓
Step 7: 更新待辦清單
```

### 4.3 確認提示（強制，不可跳過）

```
⚠️ 即將關閉以下 Bug 並變更狀態為「已完成」：

| BUG ID | 標題 |
|--------|------|
| 50021 | 紅色數字用途不明確... |

此操作通常由測試組驗證通過後執行。
確定要關閉嗎？輸入「確認關閉」繼續，其他任意輸入取消。
```

### 4.4 自動組合留言格式

```html
<p><strong><<修正結果>></strong></p>
<p>{修正說明，從 commit message 自動取得}</p>
<p>已部署至 DEV 環境並通過測試驗證。</p>
```

---

## 5. 子指令：comment（貼留言 + 上傳截圖）

### 5.1 用法

```
/Bug管理 comment 50021                                    # 自動抓 commit + 截圖
/Bug管理 comment 50021 已修正，首頁 Card 新增 tooltip 說明  # 自訂文字
/Bug管理 comment 50021 tests/e2e/screenshots/tc01.png      # 指定截圖
/Bug管理 comment 50021 已修正 tests/e2e/screenshots/       # 自訂文字 + 整個目錄截圖
```

### 5.2 前置條件檢查（自動偵測）

| 條件 | 行為 |
|------|------|
| 連 commit 都沒有 | **阻擋**：「BUG {id} 尚無任何 commit，請先修完再回寫」 |
| 有 commit 但沒截圖 | **提醒**：「偵測到 BUG {id} 尚未執行測試驗證，建議先執行 /測試 產生截圖佐證。輸入『跳過』繼續，其他任意輸入取消。」 |
| 有 commit + 有截圖 | **直接執行**，不提醒 |

### 5.3 無參數時的自動偵測

當只輸入 `/Bug管理 comment 50021`（不帶文字和圖片）時，自動偵測內容：

| 內容來源 | 偵測方式 |
|---------|---------|
| **文字** | 從 `git log --oneline --grep="BUG 50021"` 取得 commit message |
| **截圖** | 從 `tests/e2e/screenshots/` 掃描檔名含 BUG ID 或對應 test case 的圖片 |

### 5.4 有參數時的判斷規則

| 參數內容 | 判斷方式 | 動作 |
|---------|---------|------|
| 以 `.png` / `.jpg` / `.jpeg` 結尾 | 視為圖片路徑 | 上傳截圖 |
| 以 `/` 結尾 | 視為目錄路徑 | 掃描目錄下所有圖片並上傳 |
| 其他 | 視為留言文字 | 貼純文字留言 |
| 混合 | 同時有文字和圖片路徑 | 合併為一則留言（文字在上、截圖在下） |

### 5.5 執行流程

```
Step 1: 前置條件檢查（commit 存在？截圖存在？）
    ↓
Step 2: 解析參數（自動偵測 or 手動指定）
    ↓
Step 3: 若有圖片，上傳到 Azure DevOps
    POST /_apis/wit/attachments?fileName={name}&api-version=7.1
    Content-Type: application/octet-stream
    Body: {binary file data}
    → 回傳 attachment URL
    ↓
Step 4: 組合 HTML 留言（文字 + 圖片）
    ↓
Step 5: 貼 Discussion 留言
    POST /_apis/wit/workitems/{id}/comments?api-version=7.1-preview.4
    Body: { "text": "<p>留言文字</p><p><img src='{url}'></p>" }
```

### 5.6 批次上傳

若指定目錄路徑，自動掃描目錄下所有 `.png` / `.jpg` 檔案，一次全部上傳並組合成一則留言。

---

## 6. 無參數 / help（預設：顯示狀態）

> `/Bug管理 -h` 或 `/Bug管理 help` 會顯示子指令清單與用法說明。

### 7.1 執行流程

1. 讀取待辦清單中的 Azure DevOps Bug 區塊
2. 統計完成/待辦/進行中數量
3. 與 Azure DevOps 比對是否有未同步的項目
4. 輸出報告

### 7.2 輸出格式

```
【Azure DevOps Bug 狀態】

Iteration: {iteration}，截止：{deadline}

| 狀態 | 數量 | 說明 |
|------|------|------|
| ✅ 完成 | 20 | 已修正並 commit |
| 🔄 進行中 | 2 | 正在處理 |
| ⬜ 待辦 | 4 | 尚未開始 |

⚠️ 有 3 張本地已完成但 Azure DevOps 未同步，執行 `/Bug管理 sync` 同步。
```

---

## 8. Skill 串接機制

### 8.1 被動串接（其他 Skill → Bug 管理）

| 來源 Skill | 觸發條件 | 提醒內容 |
|-----------|---------|---------|
| Commit | push 完成後，commit message 含 BUG ID | 「偵測到 BUG {id} 已修正，要回寫 Azure DevOps 嗎？」 |
| 測試 | 測試完成且有截圖 | 「測試截圖已產生，要上傳到 BUG {id} 嗎？」 |
| 待辦事項 | Bug 項目標記完成 | 「BUG {id} 已完成，要同步到 Azure DevOps 嗎？」 |

### 8.2 主動串接（Bug 管理 → 其他 Skill）

| 動作 | 觸發 Skill |
|------|-----------|
| pull 完成後 | 建議 `/待辦事項` 查看清單 |
| 選擇要修的 Bug 後 | 建議 `/分析` 進行需求分析 |

---

## 9. URL 解析規則

從查詢 URL 自動解析連線參數：

```
https://dev.azure.com/{org}/{project}/_queries/query/{queryId}
                       ↑       ↑                      ↑
                      org   project               queryId
```

**支援的 URL 格式：**

| 格式 | 範例 |
|------|------|
| 查詢 URL | `https://dev.azure.com/myorg/MyProject/_queries/query/xxx-xxx` |
| Work Item URL | `https://dev.azure.com/myorg/MyProject/_workitems/edit/50021` |
| 純 Query ID | `2f824d02-6f25-4b77-a1e1-e72b8a93d45c` |
| 純 Bug ID | `50021` 或 `50021,50024,50025` |

---

## 10. 錯誤處理

| 錯誤 | 處理方式 |
|------|---------|
| PAT 未設定 | 提示使用者設定 `AZURE_DEVOPS_PAT` 環境變數或直接貼上 |
| PAT 過期（401） | 提示使用者重新產生 PAT |
| PAT 權限不足（403） | 提示需要 Work Items Read+Write scope |
| Bug ID 不存在（404） | 顯示錯誤並跳過 |
| 網路逾時 | 重試一次，仍失敗則顯示錯誤 |

---

## 11. 安全規範

| 規則 | 說明 |
|------|------|
| **禁止儲存 PAT** | 不可寫入 Memory、SKILL.md、CLAUDE.md、任何檔案 |
| **禁止 commit PAT** | git add 前檢查是否含 PAT 字串 |
| **環境變數優先** | 優先讀取 `AZURE_DEVOPS_PAT`，未設定才提示使用者輸入 |
| **對話內暫存** | 使用者在對話中貼的 PAT 僅在當次對話有效，不持久化 |
| **遮罩顯示** | 顯示 PAT 時只顯示前 4 + 後 4 字元：`abcd...wxyz` |

---

## 12. 禁止行為

- 禁止在任何檔案中儲存 PAT / Token / 密碼
- 禁止未確認就自動更新 Azure DevOps 狀態
- 禁止修改非目前使用者 Assigned 的 Bug
- 禁止刪除 Bug（只能關閉或變更狀態）
- 禁止在留言中包含敏感資訊（密碼、內部 IP）
