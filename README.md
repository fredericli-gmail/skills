# Skill_New：開發工作流 SKILL 集合

> 完整的「分析 → 開發 → 審查 → 重構 → 版控 → 部署 → 測試」工作流 SKILL 集合。
> 所有 SKILL 統一使用 **`OKOKYES`** 作為使用者確認詞，與全域 CLAUDE.md 一致。

---

## 結構總覽

```
skill_new/
├── _shared/                    # 共享規範（被多個 SKILL 引用）
│   ├── CODING-STANDARDS.md     # 編碼規範（架構、註解、Exception、命名）
│   └── SECURITY-CHECKLIST.md   # 資安規範（XSS、CSRF、SQLi、認證、機密）
│
├── 分析/                        # 需求分析與設計規劃
│   ├── SKILL.md
│   ├── QUESTIONS.md            # 系統化問題清單
│   ├── TEMPLATES.md            # 分析報告模板
│   └── SOLUTIONS.md            # 常見問題建議方案
│
├── 開發-Java後端/               # Java + Spring Boot
│   └── SKILL.md
├── 開發-Python後端/             # Python + FastAPI
│   └── SKILL.md
├── 開發-React前端/              # React 18 + Vite + Tailwind
│   └── SKILL.md
├── 開發-Thymeleaf前端/          # Thymeleaf + 原生 JS
│   └── SKILL.md
│
├── CodeReview/                  # 程式碼品質審查
│   ├── SKILL.md
│   └── CHECKLIST.md            # 完整檢查項目與正則
│
├── 重構/                        # 程式碼重構
│   ├── SKILL.md
│   ├── PATTERNS.md             # 壞味道與重構手法對照
│   └── CHECKLIST.md
│
├── 測試/                        # Playwright E2E 測試
│   ├── SKILL.md
│   ├── PLAYWRIGHT-REFERENCE.md
│   ├── TEST-PLAN.md
│   └── TROUBLESHOOTING.md
│
├── Commit/                      # Git 規範化提交
│   └── SKILL.md
├── 上版/                        # Commit + DEV 部署一條龍（通用框架）
│   └── SKILL.md
└── 待辦事項/                    # 跨對話任務追蹤
    └── SKILL.md
```

---

## 工作流程

### 標準開發流程

```
分析  →  開發-*  →  CodeReview  →  Commit  →  測試
                    ↓ 失敗
                    重構（可選）
```

### 包含部署的流程

```
分析  →  開發-*  →  CodeReview  →  上版  →  測試
                                  ↑
                                  （包含 Commit + DEV 部署）
```

### 確認詞統一

**所有 SKILL 強制使用 `OKOKYES` 作為唯一確認詞：**

| 觸發點 | 動作 |
|-------|------|
| 分析報告完成 | 等待 `OKOKYES` → 啟動對應的「開發-*」 |
| 開發完成 | 等待 `OKOKYES` → 啟動 `/CodeReview` |
| 審查通過 | 等待 `OKOKYES` → 啟動 `/Commit` 或 `/上版` |
| 重構計畫確認 | 等待 `OKOKYES` → 執行重構 |
| Commit 分組確認 | 等待 `OKOKYES` → 執行 commit |
| 測試計畫確認 | 等待 `OKOKYES` → 執行測試 |

❌ **禁止接受**：「確認」、「同意」、「OK」、「好」、「可以」、`okgo`、`Jonatan` 等其他詞彙。

---

## SKILL 索引

| SKILL | 用途 | 觸發時機 | 串接下一步 |
|-------|------|---------|-----------|
| **分析** | 需求理解、程式碼掃描、設計規劃 | 用戶提出新需求 | 對應的「開發-*」 |
| **開發-Java後端** | Spring Boot 後端開發（無 Lombok/Lambda/Stream） | 涉及 `.java` / Maven | CodeReview |
| **開發-Python後端** | FastAPI 後端開發 | 涉及 `.py` / FastAPI | CodeReview |
| **開發-React前端** | React 18 + Vite + Tailwind 前端開發 | 涉及 `.jsx` / Vite | CodeReview |
| **開發-Thymeleaf前端** | Thymeleaf 模板 + 原生 JS / jQuery | 涉及 `templates/*.html` | CodeReview |
| **CodeReview** | 程式碼品質、資安、註解審查 | 開發完成自動觸發 | Commit / 上版（通過）或 開發-*（失敗） |
| **重構** | 改善程式碼結構（不變更行為） | 程式碼異味或 CR 建議 | CodeReview |
| **測試** | Playwright E2E 自動化測試 | Commit / 上版完成 | 流程結束 |
| **Commit** | 規範化 Git 提交（不部署） | CodeReview 通過 | 測試 |
| **上版** | Commit + DEV 部署（通用框架） | CodeReview 通過 | 測試 |
| **待辦事項** | 跨對話任務追蹤 | `/待辦事項` 或對話中提及 | - |

---

## 共享規範

為避免重複，跨 SKILL 共用的規範統一放在 `_shared/`：

### `_shared/CODING-STANDARDS.md`

被以下 SKILL 引用：
- 開發-Java後端 / 開發-Python後端 / 開發-React前端 / 開發-Thymeleaf前端
- CodeReview / 重構

包含：
1. Controller / Service 架構規範（強制）
2. 註解規範（逐行繁體中文）
3. Exception 處理與日誌規範
4. 資料庫異動規範（Liquibase / Alembic）
5. 命名與檔案規範
6. 程式碼品質要求

### `_shared/SECURITY-CHECKLIST.md`

被以下 SKILL 引用：
- 開發-* / CodeReview

包含：
1. XSS 防護（依技術棧）
2. CSRF 防護（Session-Based / Token-Based）
3. SQL Injection 防護（依 ORM）
4. 認證與授權
5. 敏感資料保護
6. HTTP 安全標頭
7. Fortify / 弱掃常見問題

> 修改規範時，**只需修改 `_shared/` 內的檔案**，所有 SKILL 自動同步生效。

---

## 部署設定（針對「上版」SKILL）

`/上版` SKILL 為**通用框架**，不包含任何專案特定資訊。
專案特定的部署資訊（IP、SSH、路徑、健康檢查 URL 等）必須由使用者在**專案層級 CLAUDE.md** 中提供。

範本見 [上版/SKILL.md §10](上版/SKILL.md#10-專案-claudemd-部署設定範本)。

### 安全規範（強制）

- ✅ **強制使用 SSH key 認證**
- ❌ **禁止**在 SKILL 或專案 CLAUDE.md 中寫入明文密碼
- ❌ **禁止**使用 `sshpass -p '密碼'`
- ✅ 敏感設定透過環境變數注入

---

## 與舊版 SKILL 的差異

本目錄取代舊版 `skill/` 與 `skills 4/`。主要改進：

| 改進項目 | 舊版問題 | 新版做法 |
|---------|---------|---------|
| **確認詞統一** | 舊 `skill/` 用 `OKOKYES`、`skills 4/` 用 `okgo`、重構用 `Jonatan` | 全部統一為 `OKOKYES` |
| **密碼移除** | 多處硬編碼明文密碼 | 完全移除，強制 SSH key |
| **專案資訊外移** | 特定公司專案名稱與內網 IP 寫死在 SKILL | 上版 SKILL 改為通用框架，專案資訊移到專案層級 CLAUDE.md |
| **開發 SKILL 拆分** | 一個「開發」SKILL 混雜 Java + Thymeleaf + React + Liquibase | 拆成 4 個獨立 SKILL（Java後端 / Python後端 / React前端 / Thymeleaf前端） |
| **規範重複** | Controller/Service 架構規範被複製到 4 個 SKILL | 抽到 `_shared/CODING-STANDARDS.md`，所有 SKILL 引用 |
| **測試方向** | `skill/` 用 Playwright，`skills 4/` 用 Selenium | 統一使用 Playwright（依使用者選擇） |
| **隔離開發** | 額外的 worktree 工作流 SKILL | 移除（依使用者選擇） |
| **重構奇怪規範** | 舊版要求「禁止 `let`/`const`，使用 `var`」 | 移除此違反現代 JS 標準的規範 |

---

## 安裝與啟用

1. 將整個 `skill_new/` 複製或符號連結到 Claude Code 的 skill 目錄
2. 或將各 SKILL 的 `SKILL.md` 註冊到 Claude Code 設定
3. 確保 `_shared/` 與各 SKILL 的相對路徑保持完整
