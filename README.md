# Skill_New：開發工作流 SKILL 集合

> 完整的「分析 → 開發 → 審查 → 資安檢測 → 重構 → 版控 → 測試」工作流 SKILL 集合。
> 所有 SKILL 統一使用 **`OKOKYES`** 作為使用者確認詞，與全域 CLAUDE.md 一致。
> 資安檢測依據 **OWASP Top 10 2025 Release Candidate**（2025-11-06 發布）。

---

## 結構總覽

```
skills/
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
├── 原始碼掃描/                  # SAST：OWASP Top 10 2025 靜態資安掃描
│   └── SKILL.md
├── 網頁弱點掃描/                # DAST：OWASP ZAP 動態弱點掃描（本機限定）
│   └── SKILL.md
├── 滲透測試/                    # 本機服務滲透測試（五階段）
│   └── SKILL.md
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
└── 待辦事項/                    # 跨對話任務追蹤
    └── SKILL.md
```

---

## 工作流程

### 標準開發流程

```
分析  →  開發-*  →  CodeReview  →  原始碼掃描  →  Commit  →  測試
                    ↓ 失敗                ↓ 失敗
                    重構（可選）          開發-* 修補
```

### 完整資安檢測流程（三層縱深防禦）

```
開發 ─► CodeReview ─► 原始碼掃描(SAST) ─► 網頁弱點掃描(DAST) ─► 滲透測試
                            │                     │                │
                            └─────────────────────┴────────────────┘
                               任一層發現漏洞 → 回「開發-*」修補後重跑
```

| 層級 | 時機 | 檢測方式 | 工具 |
|------|------|---------|------|
| **SAST** | 程式碼階段 | 靜態原始碼分析 | Semgrep、Bandit、SpotBugs、ESLint Security、syft/grype |
| **DAST** | 部署測試環境後 | 黑箱動態掃描 | OWASP ZAP、nikto |
| **Pentest** | 深度驗證 | 主動攻擊驗證 | nmap、sqlmap、ffuf、httpx、jwt_tool |

### 確認詞統一

**所有 SKILL 強制使用 `OKOKYES` 作為唯一確認詞：**

| 觸發點 | 動作 |
|-------|------|
| 分析報告完成 | 等待 `OKOKYES` → 啟動對應的「開發-*」 |
| 開發完成 | 等待 `OKOKYES` → 啟動 `/CodeReview` |
| 審查通過 | 等待 `OKOKYES` → 啟動 `/原始碼掃描` |
| 原始碼掃描通過 | 等待 `OKOKYES` → 啟動 `/Commit` 或 `/網頁弱點掃描` |
| 網頁弱點掃描通過 | 等待 `OKOKYES` → 啟動 `/滲透測試` |
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
| **CodeReview** | 程式碼品質、資安、註解審查 | 開發完成自動觸發 | 原始碼掃描（通過）或 開發-*（失敗） |
| **原始碼掃描** | OWASP Top 10 2025 SAST 靜態資安掃描 | CodeReview 通過後 | Commit / 網頁弱點掃描（通過）或 開發-*（失敗） |
| **網頁弱點掃描** | OWASP ZAP 動態弱點掃描（本機限定） | 部署測試環境後 | 滲透測試（通過）或 開發-*（失敗） |
| **滲透測試** | 本機服務五階段滲透（資訊蒐集→後滲透） | 需深度資安驗證時 | 產出滲透報告 / 開發-*（修補） |
| **重構** | 改善程式碼結構（不變更行為） | 程式碼異味或 CR 建議 | CodeReview |
| **測試** | Playwright E2E 自動化測試 | Commit 完成 | 流程結束 |
| **Commit** | 規範化 Git 提交 | 原始碼掃描通過 | 測試 |
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
- 開發-* / CodeReview / 原始碼掃描 / 網頁弱點掃描 / 滲透測試

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

## 資安檢測規範（OWASP Top 10 2025）

三個資安 Skill 皆依據 **OWASP Top 10 2025 Release Candidate**（2025-11-06 發布）。

### OWASP Top 10 2025 完整清單

| 編號 | 類別 | 與 2021 差異 |
|------|------|-------------|
| A01 | Broken Access Control | 吸收 2021 A10 SSRF |
| A02 | Security Misconfiguration | ⬆️ 從 #5 升至 #2 |
| A03 | **Software Supply Chain Failures** | 🆕 新增，擴大 2021 A06 範圍 |
| A04 | Cryptographic Failures | ⬇️ 從 #2 降至 #4 |
| A05 | Injection（含 XSS） | ⬇️ 從 #3 降至 #5 |
| A06 | Insecure Design | ⬇️ 從 #4 降至 #6 |
| A07 | Authentication Failures | 不變 |
| A08 | Software or Data Integrity Failures | 不變 |
| A09 | Security Logging and **Alerting** Failures | 強調告警 |
| A10 | **Mishandling of Exceptional Conditions** | 🆕 新增 |

官方參考：https://owasp.org/Top10/2025/

### 本機測試安全紅線（網頁弱點掃描 / 滲透測試）

- ✅ **僅限本機 / RFC1918 私有 IP**：localhost、127.0.0.1、192.168.x.x、10.x.x.x、172.16-31.x.x
- ❌ 禁止外網目標（公網域名 / 公網 IP / 雲端主機）
- ❌ 禁止 DoS、大流量壓測
- ❌ 禁止真實資料破壞（DROP、TRUNCATE、rm -rf）
- ❌ 禁止後門植入、社交工程、C2 框架
- ✅ 所有危險操作需二次 `OKOKYES` 確認

---

## 與舊版 SKILL 的差異

本目錄取代舊版 `skill/` 與 `skills 4/`。主要改進：

| 改進項目 | 舊版問題 | 新版做法 |
|---------|---------|---------|
| **確認詞統一** | 舊 `skill/` 用 `OKOKYES`、`skills 4/` 用 `okgo`、重構用 `Jonatan` | 全部統一為 `OKOKYES` |
| **密碼移除** | 多處硬編碼明文密碼 | 完全移除，強制 SSH key |
| **開發 SKILL 拆分** | 一個「開發」SKILL 混雜 Java + Thymeleaf + React + Liquibase | 拆成 4 個獨立 SKILL |
| **規範重複** | Controller/Service 架構規範被複製到 4 個 SKILL | 抽到 `_shared/CODING-STANDARDS.md`，所有 SKILL 引用 |
| **測試方向** | `skill/` 用 Playwright，`skills 4/` 用 Selenium | 統一使用 Playwright |
| **重構奇怪規範** | 舊版要求「禁止 `let`/`const`，使用 `var`」 | 移除此違反現代 JS 標準的規範 |
| **資安檢測** | 僅靠 CodeReview 人工檢查 | 新增三層縱深（SAST + DAST + Pentest），依 OWASP Top 10 2025 |

---

## 安裝與啟用

1. 將整個 `skills/` 複製或符號連結到 Claude Code 的 skill 目錄
2. 或將各 SKILL 的 `SKILL.md` 註冊到 Claude Code 設定
3. 確保 `_shared/` 與各 SKILL 的相對路徑保持完整
4. 資安 Skill 首次使用前需安裝工具（Semgrep、Bandit、OWASP ZAP、nmap、sqlmap 等），各 SKILL 內含自動安裝指引
