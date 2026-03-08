---
name: 分析
description: "專業需求分析與設計規劃工具。當用戶提出新功能需求、系統改進、或任何開發任務時觸發。執行完整的需求理解、現有程式碼掃描、問題識別、風險評估，並產出結構化的分析報告與設計方案。適用於：(1) 新功能開發需求分析 (2) 系統重構評估 (3) 技術可行性分析 (4) 架構設計規劃 (5) 需求釐清與確認流程。當用戶說「分析這個需求」、「我需要做 XXX 功能」、「幫我規劃」、「評估可行性」時觸發此 skill。"
---

# 需求分析與設計規劃

## Overview

當用戶提出開發需求時，使用此流程進行系統化分析。核心原則：先理解後動手、主動質疑、全面掃描、風險前置、等待確認。

## Workflow Decision Tree

### 收到新需求
執行完整的「五階段分析流程」

### 需求不明確
使用「Phase 2: 疑問識別」提出問題

### 需要評估影響範圍
使用「Phase 3: 程式碼掃描」

### 需要完整設計文件
使用「Phase 4: 設計規劃」並參考 [TEMPLATES.md](TEMPLATES.md)

## 五階段分析流程

### Phase 1: 需求接收與初步理解

收到需求後，立即執行：

1. **複述需求** - 用自己的話重新描述
2. **識別關鍵字** - 提取核心概念、實體、行為
3. **界定範圍** - 明確什麼在/不在範圍內

輸出格式：
```
【需求理解摘要】
核心目標：[一句話描述]
主要功能：[功能清單]
涉及範圍：[影響的模組/系統]
預期產出：[最終交付物]
```

### Phase 2: 疑問與問題識別

針對需求提出問題。完整問題清單見 [QUESTIONS.md](QUESTIONS.md)。

#### 必須涵蓋的問題類別

**功能性疑問**
- 邊界條件處理？異常情況處理？預設值？

**非功能性疑問**
- 效能要求？安全性考量？可擴展性需求？

**業務邏輯疑問**
- 業務規則完整性？特殊案例？與現有流程關係？

**技術可行性疑問**
- 現有架構支援度？需要新技術？技術債處理？

#### 問題分級輸出格式

```
【疑問清單】

🔴 關鍵問題（必須回答才能繼續）：
1. [問題]
   建議方案：[Claude 的建議]

🟡 重要問題（影響設計決策）：
1. [問題]
   建議方案：[建議處理方式]

🟢 確認事項（可使用預設值）：
1. [問題]
   預設方案：[預設處理]
```

⏸️ **停止點：等待用戶回覆後才繼續 Phase 3**

### Phase 3: 現有程式碼掃描

執行系統化掃描：

```bash
# 了解專案結構
find . -type f \( -name "*.java" -o -name "*.jsx" -o -name "*.js" -o -name "*.css" -o -name "*.yml" \) | head -100

# 搜尋相關程式碼
grep -rn "關鍵字" --include="*.java" --include="*.jsx" --include="*.js"
```

掃描重點：
- 目錄結構與技術棧
- 需修改的檔案
- 可能受影響的檔案
- 現有測試覆蓋
- 技術債識別

輸出格式：
```
【程式碼掃描報告】

📁 專案架構：
[技術棧與架構描述]

📝 相關檔案：
[需修改]
├── path/file1.jsx - [修改原因]
└── path/file2.js - [修改原因]

[可能受影響]
├── path/related1.jsx - [影響原因]
└── path/related2.js - [影響原因]

⚠️ 注意事項：
- [技術債或問題]
- [架構限制]
```

### Phase 4: 完整分析與設計規劃

整合資訊產出設計文件。詳細模板見 [TEMPLATES.md](TEMPLATES.md)。

#### 設計文件結構

```
【需求分析與設計規劃書】

1. 執行摘要
   - 目標、方法、預估工時

2. 需求分析
   2.1 功能需求（含驗收標準）
   2.2 非功能需求
   2.3 限制條件

3. 技術設計
   3.1 架構設計
   3.2 資料設計
   3.3 介面設計
   3.4 實作計畫

4. 風險評估
   4.1 技術風險
   4.2 業務風險
   4.3 緩解策略

5. 實作計畫
   5.1 任務拆解
   5.2 預估工時
   5.3 里程碑

6. 待確認事項
```

⏸️ **停止點：等待用戶確認設計方案後才開始實作**

### Phase 5: 確認與調整

```
請確認以下事項：
✅ 需求理解是否正確？
✅ 設計方案是否符合期望？
✅ 是否有需要調整的部分？
✅ 是否可以開始實作？

請回覆「OKOKYES」開始實作，或告訴我需要調整的部分。
```

## 問題提出原則

### 何時必須提出問題

- 模糊需求（如「讓它更好用」）
- 缺少邊界條件
- 可能的衝突
- 隱含假設
- 影響範圍不明

### 問題格式要求

每個問題必須包含：
1. **問題本身** - 清楚描述疑問
2. **為什麼重要** - 對設計的影響
3. **建議方案** - Claude 的建議處理方式

常見問題的建議方案見 [SOLUTIONS.md](SOLUTIONS.md)。

## Controller / Service 架構規範（強制）

> ⚠️ **最高原則**：每一個前端網頁功能，對應到後端必須有專用的 RestController，絕對禁止跨 Controller 使用，否則會造成權限控管混亂。

### 架構設計原則

| 規則 | 說明 |
|------|------|
| **專用 Controller** | 每個前端頁面/功能模組對應一個專用的 RestController |
| **專用 Service** | Controller 專用的方法，放在同名的 Service 中（如 `UserController` → `UserService`） |
| **禁止跨 Controller** | ❌ 禁止一個網頁功能呼叫其他 Controller 的 API |
| **權限獨立** | 每個 Controller 有獨立的權限控制，不與其他功能混用 |

### 命名規範

```
前端頁面              Controller                  Service
─────────────────────────────────────────────────────────────
使用者管理            UserController              UserService
AI 知識庫管理         AiPromptController          AiPromptService / AiDatasetTextService
AI 程式知識庫         AiSourceCodeController      AiSourceCodeService（專用）
AI PDF 知識庫         AiPdfController             AiPdfService（專用）
```

### 分析階段檢查項目

在 Phase 3 程式碼掃描時，必須檢查：

1. **功能是否有專用 Controller**
   - ✅ 有 → 在該 Controller 中新增/修改 API
   - ❌ 沒有 → 建立新的專用 Controller

2. **方法是否應放在專用 Service**
   - 若方法只被此 Controller 使用 → 放在專用 Service（與 Controller 同名）
   - 若方法被多個 Controller 共用 → 放在通用 Service（如 `AiKnowledgeBaseService`）

3. **是否有跨 Controller 呼叫**
   - 發現跨 Controller 呼叫 → 標記為 🔴 架構問題，必須修正

### 範例：正確 vs 錯誤

```
✅ 正確架構：
AiSourceCodeController
    └─→ AiSourceCodeService.refreshIndexingStatus()  （專用方法）
    └─→ AiKnowledgeBaseService.findById()            （共用方法）

❌ 錯誤架構：
AiSourceCodeController
    └─→ AiPromptController.refreshIndexingStatus()   （跨 Controller 呼叫）
    └─→ AiDatasetTextService.refreshIndexingStatus() （使用其他頁面的 Service）
```

---

## 輸出品質標準

✓ 所有需求都有明確的驗收標準
✓ 所有疑問都有建議的處理方案
✓ 設計方案可直接轉換為開發任務
✓ 風險都有對應的緩解策略
✓ 實作計畫具有可執行性
✓ **架構設計符合 Controller/Service 規範**

## 常見陷阱

❌ 假設用戶知道技術細節
❌ 跳過問題直接給解決方案
❌ 需求不明確時開始設計
❌ 忽略非功能性需求
❌ 低估變更的影響範圍

## Next Steps

- 完整問題清單：[QUESTIONS.md](QUESTIONS.md)
- 分析報告模板：[TEMPLATES.md](TEMPLATES.md)
- 建議處理方案：[SOLUTIONS.md](SOLUTIONS.md)

---

## Skill 串接機制（強制執行）

> ⚠️ **當使用者輸入 `OKOKYES` 確認分析報告後，必須執行以下流程：**

### 強制執行步驟

1. **確認分析完成**：使用者已輸入 `OKOKYES` 同意分析報告
2. **明確告知找到下一個 Skill**：必須輸出以下格式的訊息

```
【Skill 串接通知】

✅ 分析階段已完成
✅ 已找到對應的 Skill：「開發」(/開發)

此 Skill 將協助您進行：
- Java 後端開發（禁用 Lombok、Lambda）
- React 18 + Vite + Tailwind CSS 前端開發（JSX）
- 資安規範檢查

請輸入「OKOKYES」啟動「開發」Skill，或輸入其他內容繼續對話。
```

3. **等待使用者確認**：
   - ✅ 使用者輸入 `OKOKYES` → 立即執行 `/開發` skill
   - ❌ 使用者輸入其他內容 → 不啟動，繼續正常對話

### 禁止行為

- ❌ 禁止未告知就自動啟動下一個 Skill
- ❌ 禁止跳過「已找到 Skill」的通知
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動 Skill
