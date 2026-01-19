# Claude Code Skills

此目錄包含 Claude Code 的自訂 Skill，用於標準化開發流程。

---

## Skill 總覽

| Skill | 指令 | 說明 |
|-------|------|------|
| 分析 | `/分析` | 需求分析與設計規劃 |
| 開發 | `/開發` | Java 開發（禁用 Lombok/Lambda） |
| CodeReview | `/CodeReview` | 程式碼審查 |
| 測試 | `/測試` | Selenium 自動化測試 |
| 重構 | `/重構` | 程式碼重構 |
| **隔離開發** | `/隔離開發` | **Worktree 隔離開發模式** |

---

## 隔離開發 Skill 使用說明

### 核心理念

- **主目錄**：保持穩定，僅用於測試已完成的功能
- **Worktree**：每個開發任務在獨立環境中進行
- **同步機制**：透過 push/pull 同步變更

### 快速開始

```bash
# 1. 在主目錄啟動 Claude
cd /Users/fredericli/Git/Danova/Danova-woms
claude

# 2. 觸發隔離開發 Skill
> /隔離開發 我想要新增使用者匯出功能
```

### 完整工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│ 第 1 步：主目錄啟動                                              │
│ $ claude                                                        │
│ > /隔離開發 我想要新增 XXX 功能                                   │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼ Claude 分析需求 → 您確認 OKOKYES
┌─────────────────────────────────────────────────────────────────┐
│ 第 2 步：建立開發環境                                            │
│ Claude 自動執行：                                                │
│ - git worktree add ./sub-danova-worm/feature-xxx                │
│ - 產生 TASK.md（含完整執行計畫）                                  │
│ - 輸出切換指令                                                   │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼ 您執行：cd ./sub-danova-worm/feature-xxx && claude
┌─────────────────────────────────────────────────────────────────┐
│ 第 3 步：Worktree 開發                                          │
│ > 開始                                                          │
│                                                                 │
│ Claude 自動執行：                                                │
│ - 讀取 TASK.md                                                  │
│ - 依計畫開發                                                     │
│ - 每階段 commit + push                                          │
│ - 通知您可以測試                                                 │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼ 您在主目錄測試（可選）
┌─────────────────────────────────────────────────────────────────┐
│ 測試方式（在另一個終端機）                                        │
│ $ cd /Users/fredericli/Git/Danova/Danova-woms                   │
│ $ git fetch origin                                               │
│ $ git checkout feature-xxx                                       │
│ $ ./mvnw spring-boot:run                                         │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼ 開發完成 → 您確認 OKOKYES
┌─────────────────────────────────────────────────────────────────┐
│ 第 4 步：CodeReview                                             │
│ - 程式碼品質審查                                                 │
│ - 禁用語法檢查（Lombok/Lambda/Stream）                           │
│ - 資安規範審查（XSS/CSRF/SQL Injection）                         │
│ - 繁體中文註解完整度檢查                                          │
│                                                                 │
│ ✅ 通過 → 進入測試                                               │
│ ❌ 失敗 → 回到開發修正                                           │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼ 審查通過 → 您確認 OKOKYES
┌─────────────────────────────────────────────────────────────────┐
│ 第 5 步：測試                                                    │
│ - Selenium 自動化測試                                            │
│ - 使用實體瀏覽器（非 headless）                                   │
│ - 功能驗證                                                       │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼ 測試通過
┌─────────────────────────────────────────────────────────────────┐
│ 第 6 步：合併與清理                                              │
│ $ cd /Users/fredericli/Git/Danova/Danova-woms                   │
│ $ git checkout develop                                           │
│ $ git merge feature-xxx                                          │
│ $ git worktree remove ./sub-danova-worm/feature-xxx             │
│ $ git branch -d feature-xxx  # 可選：刪除本地分支                 │
└─────────────────────────────────────────────────────────────────┘
```

### 目錄結構

```
/Users/fredericli/Git/Danova/Danova-woms/
├── src/                          # 主目錄（穩定測試環境）
├── sub-danova-worm/              # Worktree 容器（已加入 .gitignore）
│   ├── feature-user-export/      # 任務 A 的開發環境
│   │   ├── src/
│   │   └── TASK.md               # 任務說明與執行計畫
│   └── fix-login-bug/            # 任務 B 的開發環境（可同時進行）
│       ├── src/
│       └── TASK.md
```

### 關鍵確認點

整個流程中，每個關鍵步驟都需要您輸入 `OKOKYES` 確認後才會繼續：

| 階段 | 確認時機 | 確認後動作 |
|------|---------|-----------|
| 分析完成 | 需求分析與執行計畫產出後 | 建立 Worktree |
| 開發完成 | 所有階段程式碼完成後 | 啟動 CodeReview |
| 審查通過 | CodeReview 無錯誤時 | 啟動測試 |
| 審查失敗 | CodeReview 有錯誤時 | 回到開發修正 |

### 常見問題

**Q: 如果開發到一半想放棄怎麼辦？**
```bash
cd /Users/fredericli/Git/Danova/Danova-woms
git worktree remove ./sub-danova-worm/feature-xxx --force
git branch -D feature-xxx  # 強制刪除未合併的分支
```

**Q: 如何同時進行多個任務？**
- 每個任務使用獨立的 worktree
- 每個 worktree 需要獨立的終端機視窗執行 Claude

**Q: Worktree 目錄會被 git 追蹤嗎？**
- 不會，`sub-danova-worm/` 已加入 `.gitignore`

---

## 標準 Skill 串接流程

```
分析 Skill ─→ 開發 Skill ─→ CodeReview Skill ─→ 測試 Skill
    │             │              │                  │
    │             │              │                  └─ 失敗 → 開發
    │             │              └─ 失敗 → 開發
    │             └─ 開發完成
    └─ OKOKYES 確認

隔離開發 Skill = 分析 + Worktree 建立 + 開發 + CodeReview + 測試
```

---

## 檔案結構

```
/Users/fredericli/.claude/skills/
├── README.md                 # 本檔案
├── 分析/
│   ├── SKILL.md
│   ├── QUESTIONS.md
│   ├── TEMPLATES.md
│   └── SOLUTIONS.md
├── 開發/
│   └── SKILL.md
├── CodeReview/
│   ├── SKILL.md
│   └── CHECKLIST.md
├── 測試/
│   ├── SKILL.md
│   ├── SELENIUM-REFERENCE.md
│   ├── TEST-PLAN.md
│   └── TROUBLESHOOTING.md
├── 重構/
│   ├── SKILL.md
│   ├── CHECKLIST.md
│   └── PATTERNS.md
└── 隔離開發/
    ├── SKILL.md
    └── TASK_TEMPLATE.md
```
