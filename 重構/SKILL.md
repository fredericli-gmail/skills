---
name: 重構
description: "程式碼重構專家。在不改變業務邏輯的前提下，改善程式碼結構與可維護性。專注解決：(1) 方法過長 (2) 重複程式碼 (3) 高耦合低內聚 (4) 命名不清。採用小步快跑策略，每個重構步驟獨立 commit，確保可回滾。重構前必須確認測試可執行，重構後測試必須通過。"
---

# Skill: 重構（Refactoring Expert）

## Usage Trigger

- 當使用者說「重構」、「這個方法太長」、「程式碼很亂」、「重複程式碼太多」時觸發
- 當 CodeReview 發現方法過長、重複程式碼等建議項目時，可建議使用此 Skill
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

### Phase 1: 掃描與識別（必須執行）

#### 1.1 掃描目標程式碼

```bash
# 掃描 Java 檔案結構
find src/main/java -name "*.java" -type f | head -50

# 計算方法行數（識別過長方法）
grep -n "public\|private\|protected" --include="*.java" [目標檔案]

# 搜尋重複程式碼模式
grep -rn "[疑似重複的程式碼片段]" --include="*.java" src/
```

#### 1.2 識別程式碼壞味道

> 完整清單請參考 [PATTERNS.md](PATTERNS.md)

**優先處理的壞味道**：

| 優先級 | 壞味道 | 識別方式 |
|-------|-------|---------|
| 🔴 P0 | 方法過長（> 50 行） | 計算方法行數 |
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

根據掃描報告，規劃重構順序：

1. **優先處理高風險項目**：方法過長、重複程式碼
2. **由內而外**：先重構被依賴的底層方法
3. **相關項目一起處理**：同一類別的問題盡量一起重構

#### 2.2 產出重構計畫

```
【重構計畫】

📋 重構範圍：[檔案清單]

🎯 重構目標：
1. [目標 1：例如將 processUser() 從 105 行縮減到 30 行以內]
2. [目標 2：例如消除 UserService 與 OrderService 的重複程式碼]

📝 重構步驟：

Step 1: [重構項目名稱]
├── 檔案：[檔案路徑]
├── 手法：[使用的重構手法，如 Extract Method]
├── 說明：[具體做什麼]
└── 預期：[重構後的效果]

Step 2: [重構項目名稱]
├── 檔案：[檔案路徑]
├── 手法：[重構手法]
├── 說明：[具體做什麼]
└── 預期：[重構後的效果]

[更多步驟...]

⚠️ 風險提示：
- [可能的風險與應對方式]

請確認以上計畫，輸入「OKOKYES」開始執行重構。
```

⏸️ **停止點：等待使用者輸入 OKOKYES 後才開始執行重構**

---

### Phase 3: 執行重構

#### 3.1 重構前準備

```bash
# 確認目前工作區乾淨
git status

# 確認測試可執行（若有測試）
./mvnw test -Dtest=[相關測試類別]
```

#### 3.2 執行重構（小步快跑）

**每個重構步驟的執行流程**：

```
1. 執行單一重構手法
   ↓
2. 確認編譯通過
   ↓
3. 執行相關測試（若有）
   ↓
4. 立即 commit
   ↓
5. 進行下一個重構步驟
```

**Commit 訊息格式**：

```bash
git commit -m "refactor: [重構說明]

- [具體變更 1]
- [具體變更 2]

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

#### 3.3 重構手法對照表

> 完整手法請參考 [PATTERNS.md](PATTERNS.md)

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

> 完整檢查清單請參考 [CHECKLIST.md](CHECKLIST.md)

```bash
# 確認編譯通過
./mvnw compile

# 執行所有相關測試
./mvnw test

# 檢查是否有遺漏的 commit
git status
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

📋 Commit 清單：
1. refactor: [commit 訊息 1]
2. refactor: [commit 訊息 2]
```

---

## 3. 常用重構手法詳解

### 3.1 Extract Method（提取方法）

**適用情境**：方法過長、程式碼區塊可獨立命名

```java
// ❌ 重構前
public void processOrder(Order order) {
    // 驗證訂單 - 開始
    if (order == null) {
        throw new IllegalArgumentException("訂單不可為空");
    }
    if (order.getItems() == null || order.getItems().isEmpty()) {
        throw new IllegalArgumentException("訂單項目不可為空");
    }
    if (order.getCustomerId() == null) {
        throw new IllegalArgumentException("客戶編號不可為空");
    }
    // 驗證訂單 - 結束

    // 後續處理...
}

// ✅ 重構後
public void processOrder(Order order) {
    // 驗證訂單資料
    validateOrder(order);

    // 後續處理...
}

/**
 * 驗證訂單資料完整性
 * @param order 待驗證的訂單
 * @throws IllegalArgumentException 當訂單資料不完整時
 */
private void validateOrder(Order order) {
    // 檢查訂單是否為空
    if (order == null) {
        throw new IllegalArgumentException("訂單不可為空");
    }
    // 檢查訂單項目是否為空
    if (order.getItems() == null || order.getItems().isEmpty()) {
        throw new IllegalArgumentException("訂單項目不可為空");
    }
    // 檢查客戶編號是否為空
    if (order.getCustomerId() == null) {
        throw new IllegalArgumentException("客戶編號不可為空");
    }
}
```

### 3.2 Extract Utility Class（提取工具類別）

**適用情境**：多個類別有相同的程式碼

```java
// ❌ 重構前：重複的驗證邏輯散落各處
public class UserService {
    public void saveUser(User user) {
        if (user == null) {
            throw new IllegalArgumentException("user 不可為 null");
        }
        // ...
    }
}

public class OrderService {
    public void saveOrder(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("order 不可為 null");
        }
        // ...
    }
}

// ✅ 重構後：提取共用工具類別
public class ValidationUtils {

    /**
     * 檢查物件不可為 null
     * @param obj 待檢查的物件
     * @param fieldName 欄位名稱（用於錯誤訊息）
     * @throws IllegalArgumentException 當物件為 null 時
     */
    public static void requireNonNull(Object obj, String fieldName) {
        // 檢查物件是否為 null
        if (obj == null) {
            // 拋出例外並附上欄位名稱
            throw new IllegalArgumentException(fieldName + " 不可為 null");
        }
    }
}

public class UserService {
    public void saveUser(User user) {
        // 驗證使用者物件不可為空
        ValidationUtils.requireNonNull(user, "user");
        // ...
    }
}
```

### 3.3 Replace Nested Conditional with Guard Clauses（衛述句）

**適用情境**：過深的 if-else 巢狀

```java
// ❌ 重構前：過深巢狀
public String getPaymentStatus(Order order) {
    String result = "";
    if (order != null) {
        if (order.getPayment() != null) {
            if (order.getPayment().isCompleted()) {
                result = "已付款";
            } else {
                result = "待付款";
            }
        } else {
            result = "無付款資訊";
        }
    } else {
        result = "訂單不存在";
    }
    return result;
}

// ✅ 重構後：使用衛述句
public String getPaymentStatus(Order order) {
    // 檢查訂單是否存在
    if (order == null) {
        return "訂單不存在";
    }
    // 檢查是否有付款資訊
    if (order.getPayment() == null) {
        return "無付款資訊";
    }
    // 檢查付款是否完成
    if (order.getPayment().isCompleted()) {
        return "已付款";
    }
    // 預設為待付款狀態
    return "待付款";
}
```

---

## 4. 技術規範遵守

重構過程中必須遵守專案的技術規範：

| 規範 | 要求 |
|------|------|
| **Lombok** | 禁止使用，必須手寫 Getter/Setter |
| **Lambda** | 禁止使用，改用傳統 for-loop |
| **Stream API** | 禁止使用，改用傳統迭代 |
| **註解** | 每行邏輯程式碼必須有繁體中文註解 |

---

## 5. Skill 串接機制（強制執行）

### 5.1 重構完成 → CodeReview

當重構完成且編譯測試通過後：

```
【Skill 串接通知】

✅ 重構階段已完成
✅ 編譯通過
✅ 測試通過（若有執行）
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

此 Skill 將協助您進行：
- 確認重構後的程式碼符合規範
- 檢查是否有遺漏的註解
- 確認無禁用語法（Lombok/Lambda/Stream）

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

**等待使用者確認**：
- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview` skill
- ❌ 使用者輸入其他內容 → 不啟動，繼續正常對話

### 5.2 從 CodeReview 進入重構

當 CodeReview 發現以下建議項目時，可提示使用重構 Skill：

- 方法超過 50 行
- 發現重複程式碼
- 巢狀層數過深
- 類別過於龐大

```
【建議使用重構 Skill】

CodeReview 發現以下可重構項目：
├── UserService.java: processUser() 方法共 85 行
└── OrderService.java: 與 UserService 有 3 處重複程式碼

建議使用「重構」Skill (/重構) 進行程式碼改善。
請輸入「OKOKYES」啟動「重構」Skill，或輸入其他內容繼續對話。
```

---

## 6. 禁止行為

- ❌ 禁止未掃描就開始重構
- ❌ 禁止未經使用者確認就執行重構
- ❌ 禁止一次重構多個不相關的項目
- ❌ 禁止重構後不執行測試
- ❌ 禁止重構過程中偷偷修改業務邏輯
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動重構

---

## 7. 相關文件

- [PATTERNS.md](PATTERNS.md) - 程式碼壞味道與重構手法對照表
- [CHECKLIST.md](CHECKLIST.md) - 重構前後檢查清單
