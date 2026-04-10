---
name: 重構
description: "程式碼重構專家（支援 Java、Python、JavaScript / React）。在不改變業務邏輯的前提下，改善程式碼結構與可維護性。專注解決：(1) 方法/函式過長 (2) 重複程式碼 (3) 高耦合低內聚 (4) 命名不清。採用小步快跑策略，每個重構步驟獨立 commit，確保可回滾。重構前必須確認測試可執行，重構後測試必須通過。確認後使用者必須輸入 OKOKYES 才會啟動執行階段。"
---

# Skill: 重構（Refactoring Expert）

> ⚠️ **本 SKILL 引用共享規範**：[_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md)

## 支援語言

| 語言 | 支援狀態 | 說明 |
|------|---------|------|
| **Java** | ✅ 完整支援 | Spring Boot、POJO、Service 層 |
| **Python** | ✅ 完整支援 | FastAPI、SQLAlchemy 模組 |
| **JavaScript / React** | ✅ 完整支援 | React 函式元件、Hooks、原生 JS |

## Usage Trigger

- 當使用者說「重構」、「這個方法太長」、「程式碼很亂」、「重複程式碼太多」時
- 當 CodeReview 發現方法過長、重複程式碼等建議項目時
- 當需要在新增功能前，先整理既有程式碼時

---

## 1. 核心原則（必須遵守）

### 1.1 重構黃金守則

| 守則 | 說明 |
|------|------|
| **行為不變** | 重構只改變程式碼結構，絕不改變業務邏輯 |
| **小步快跑** | 每次只做一個小重構，確認沒問題再繼續 |
| **頻繁提交** | 每完成一個重構步驟立即 commit |
| **測試先行** | 重構前確認測試可執行，重構後測試必須通過 |
| **可回滾** | 每個 commit 都是可獨立回滾的單位 |

### 1.2 禁止事項

- ❌ 禁止在重構過程中新增功能
- ❌ 禁止在重構過程中修復 bug（除非是重構導致的）
- ❌ 禁止一次重構太多東西
- ❌ 禁止在沒有測試覆蓋的情況下進行大規模重構
- ❌ 禁止重構後不執行測試就宣告完成

---

## 2. 重構流程

### Phase 1: 掃描與識別

#### 1.1 掃描目標程式碼

依語言選用適當的掃描指令：

```bash
# Java
find src/main/java -name "*.java" -type f | head -50
grep -n "public\|private\|protected" --include="*.java" [目標檔案]

# Python
find . -name "*.py" -type f | head -50
grep -n "^def \|^    def " --include="*.py" [目標檔案]

# JavaScript / React
find src -name "*.jsx" -o -name "*.js" -type f | head -50
wc -l src/**/*.jsx | sort -rn | head -10
```

#### 1.2 識別程式碼壞味道

> 完整清單見 [PATTERNS.md](PATTERNS.md)

**通用優先處理項目：**

| 優先級 | 壞味道 | 識別方式 |
|-------|-------|---------|
| 🔴 P0 | 方法/函式過長（> 50 行） | 計算行數 |
| 🔴 P0 | 重複程式碼 | 搜尋相似程式碼區塊 |
| 🟡 P1 | 過深巢狀（> 3 層） | 檢查 if/for 巢狀層數 |
| 🟡 P1 | 過長參數列（> 5 個） | 檢查方法簽章 |
| 🟢 P2 | 命名不清 | 人工審查 |
| 🟢 P2 | 註解過時或錯誤 | 人工審查 |

#### 1.3 產出掃描報告

```
【程式碼掃描報告】

📁 掃描範圍：[檔案/目錄]

🔴 高優先（必須重構）：
┌──────────────────────────┬──────────┬─────────────────────────┐
│ 檔案:位置                 │ 壞味道   │ 說明                    │
├──────────────────────────┼──────────┼─────────────────────────┤
│ UserService.java:45-150  │ 方法過長 │ processUser() 共 105 行 │
│ OrderService.java:23-45  │ 重複程式碼│ 與 UserService 相似     │
└──────────────────────────┴──────────┴─────────────────────────┘

🟡 中優先（建議重構）：
[同上格式]

🟢 低優先（可選重構）：
[同上格式]

📊 統計：
├── 🔴 高優先：{n} 項
├── 🟡 中優先：{n} 項
└── 🟢 低優先：{n} 項
```

---

### Phase 2: 重構規劃

#### 2.1 決定重構範圍與順序

1. **優先處理高風險項目**：方法過長、重複程式碼
2. **由內而外**：先重構被依賴的底層方法
3. **相關項目一起處理**：同一類別的問題盡量一起重構

#### 2.2 產出重構計畫

```
【重構計畫】

📋 重構範圍：[檔案清單]

🎯 重構目標：
1. [目標 1，如：將 processUser() 從 105 行縮減到 30 行以內]
2. [目標 2，如：消除 UserService 與 OrderService 的重複程式碼]

📝 重構步驟：

Step 1: [重構項目名稱]
├── 檔案：[檔案路徑]
├── 手法：[使用的重構手法，如 Extract Method]
├── 說明：[具體做什麼]
└── 預期：[重構後的效果]

Step 2: [重構項目名稱]
[同上格式]

⚠️ 風險提示：
- [可能的風險與應對方式]

請確認以上計畫，輸入「OKOKYES」開始執行重構。
```

⏸️ **停止點：等待使用者輸入 `OKOKYES` 後才開始執行重構**

---

### Phase 3: 執行重構

#### 3.1 重構前準備

```bash
# 確認目前工作區乾淨
git status

# Java：確認測試可執行
./mvnw test -Dtest=[相關測試類別]

# Python：確認語法正確
python -m py_compile [目標檔案]

# JavaScript：確認語法正確
node --check [目標檔案]
```

#### 3.2 執行重構（小步快跑）

```
1. 執行單一重構手法
   ↓
2. 確認編譯/語法通過
   ↓
3. 執行相關測試（若有）
   ↓
4. 立即 commit
   ↓
5. 進行下一個重構步驟
```

**Commit 訊息格式**：

```bash
git commit -m "$(cat <<'EOF'
refactor: [重構說明]

- [具體變更 1]
- [具體變更 2]

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

#### 3.3 重構手法對照表

> 完整手法見 [PATTERNS.md](PATTERNS.md)

| 壞味道 | 重構手法 | 說明 |
|-------|---------|------|
| 方法過長 | Extract Method | 將程式碼區塊提取為獨立方法 |
| 重複程式碼 | Extract Method → Move Method | 提取後移到共用位置 |
| 過深巢狀 | Replace Nested Conditional with Guard Clauses | 用衛述句取代巢狀 |
| 過長參數列 | Introduce Parameter Object | 將參數封裝為物件 |
| 資料泥團 | Extract Class | 將相關欄位提取為新類別 |

---

### Phase 4: 驗證與完成

#### 4.1 重構後檢查

```bash
# Java
./mvnw compile && ./mvnw test

# Python
python -m py_compile [目標檔案]
pytest [相關測試]

# JavaScript / React
npm run build
npm test
```

#### 4.2 產出重構報告

```
【重構完成報告】

✅ 重構完成

📊 重構統計：
├── 重構檔案數：{n}
├── 重構步驟數：{n}
├── Commit 數量：{n}
└── 測試結果：通過/失敗

📝 重構摘要：

| 項目 | 重構前 | 重構後 | 改善 |
|-----|-------|-------|------|
| processUser() 行數 | 105 行 | 28 行 | -73% |
| 重複程式碼區塊 | 5 處 | 0 處 | -100% |

🔧 主要變更：
1. [變更說明 1]
2. [變更說明 2]
```

---

## 3. 技術規範遵守

重構過程中必須遵守對應「開發-*」SKILL 的規範（透過共享規範統一）：

### 3.1 Java 規範

| 規範 | 要求 |
|------|------|
| **Lombok** | 禁止使用，必須手寫 Getter/Setter |
| **Lambda** | 禁止使用，改用傳統 for-loop |
| **Stream API** | 禁止使用，改用傳統迭代 |
| **註解** | 每行邏輯程式碼必須有繁體中文註解 |

### 3.2 Python 規範

| 規範 | 要求 |
|------|------|
| **type hints** | 所有函式必須有 |
| **bare except** | 禁止使用，必須指定例外類型 |
| **註解** | 每行邏輯程式碼必須有繁體中文註解 |

### 3.3 React / JavaScript 規範

| 規範 | 要求 |
|------|------|
| **函式元件** | 禁止 Class Component |
| **var** | 禁止使用，必須用 `const` / `let` |
| **註解** | 每行邏輯程式碼必須有繁體中文註解 |
| **dangerouslySetInnerHTML** | 禁止使用 |

> **註**：本 SKILL 不對 React/ES6+ 語法（箭頭函式、`const`/`let`、Template Literal）做禁用要求，這些是現代 React 標準寫法。

---

## 4. Skill 串接機制

### 4.1 重構完成 → CodeReview

當重構完成且編譯/測試通過後：

```
【Skill 串接通知】

✅ 重構階段已完成
✅ 編譯通過
✅ 測試通過（若有執行）
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

此 Skill 將協助您進行：
- 確認重構後的程式碼符合規範
- 檢查是否有遺漏的註解
- 確認無禁用語法

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview`
- ❌ 其他輸入 → 不啟動

### 4.2 從 CodeReview 進入重構

當 CodeReview 發現以下建議項目時，可提示使用重構 Skill：

- 方法/函式超過 50 行
- 發現重複程式碼
- 巢狀層數過深
- 類別/模組過於龐大

---

## 5. 禁止行為

- ❌ 禁止未掃描就開始重構
- ❌ 禁止未經使用者確認就執行重構
- ❌ 禁止一次重構多個不相關的項目
- ❌ 禁止重構後不執行測試
- ❌ 禁止重構過程中偷偷修改業務邏輯
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動執行階段

---

## 6. 相關文件

- [PATTERNS.md](PATTERNS.md) — 程式碼壞味道與重構手法對照表
- [CHECKLIST.md](CHECKLIST.md) — 重構前後檢查清單
- [_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md) — 共享編碼規範
