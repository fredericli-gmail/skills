---
name: 重構
description: "程式碼重構專家（支援 Java 與 JavaScript）。在不改變業務邏輯的前提下，改善程式碼結構與可維護性。專注解決：(1) 方法/函式過長 (2) 重複程式碼 (3) 高耦合低內聚 (4) 命名不清。採用小步快跑策略，每個重構步驟獨立 commit，確保可回滾。重構前必須確認測試可執行，重構後測試必須通過。"
---

# Skill: 重構（Refactoring Expert）

## 支援語言

| 語言 | 支援狀態 | 說明 |
|------|---------|------|
| **Java** | ✅ 完整支援 | Spring Boot、POJO、Service 層等 |
| **JavaScript** | ✅ 完整支援 | 原生 JS、jQuery、Thymeleaf 前端 |

## Usage Trigger

- 當使用者說「重構」、「這個方法太長」、「程式碼很亂」、「重複程式碼太多」時觸發
- 當 CodeReview 發現方法過長、重複程式碼等建議項目時，可建議使用此 Skill
- 當需要在新增功能前，先整理既有程式碼時
- 當前端 JavaScript 程式碼需要整理、模組化時

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

**Java 檔案掃描**：
```bash
# 掃描 Java 檔案結構
find src/main/java -name "*.java" -type f | head -50

# 計算方法行數（識別過長方法）
grep -n "public\|private\|protected" --include="*.java" [目標檔案]

# 搜尋重複程式碼模式
grep -rn "[疑似重複的程式碼片段]" --include="*.java" src/
```

**JavaScript 檔案掃描**：
```bash
# 掃描 JavaScript 檔案結構
find src/main/resources/static/js -name "*.js" -type f | head -50

# 計算函式行數（識別過長函式）
grep -n "function\|var.*=.*function\|const.*=.*function" --include="*.js" [目標檔案]

# 檢查 JavaScript 語法正確性
node --check [目標檔案]

# 搜尋重複程式碼模式
grep -rn "[疑似重複的程式碼片段]" --include="*.js" src/main/resources/static/js/

# 統計檔案行數
wc -l src/main/resources/static/js/*.js | sort -rn | head -10
```

#### 1.2 識別程式碼壞味道

> 完整清單請參考 [PATTERNS.md](PATTERNS.md)

**Java 優先處理的壞味道**：

| 優先級 | 壞味道 | 識別方式 |
|-------|-------|---------|
| 🔴 P0 | 方法過長（> 50 行） | 計算方法行數 |
| 🔴 P0 | 重複程式碼 | 搜尋相似程式碼區塊 |
| 🟡 P1 | 過深巢狀（> 3 層） | 檢查 if/for 巢狀層數 |
| 🟡 P1 | 過長參數列（> 5 個） | 檢查方法簽章 |
| 🟢 P2 | 命名不清 | 人工審查 |
| 🟢 P2 | 註解過時或錯誤 | 人工審查 |

**JavaScript 優先處理的壞味道**：

| 優先級 | 壞味道 | 識別方式 |
|-------|-------|---------|
| 🔴 P0 | 函式過長（> 50 行） | 計算函式行數 |
| 🔴 P0 | 重複程式碼 | 搜尋相似程式碼區塊 |
| 🔴 P0 | 全域變數污染 | 搜尋 `var` 在函式外宣告 |
| 🟡 P1 | 過深巢狀（> 3 層） | 檢查 if/for/callback 巢狀層數 |
| 🟡 P1 | 回呼地獄（Callback Hell） | 檢查 `.then()` 或回呼嵌套深度 |
| 🟡 P1 | 魔術數字/字串 | 搜尋未命名的常數 |
| 🟢 P2 | 命名不清 | 人工審查 |
| 🟢 P2 | 過多 console.log | 搜尋 `console.log` 數量 |

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

**Java 專案**：
```bash
# 確認目前工作區乾淨
git status

# 確認測試可執行（若有測試）
./mvnw test -Dtest=[相關測試類別]
```

**JavaScript 檔案**：
```bash
# 確認目前工作區乾淨
git status

# 確認 JavaScript 語法正確
node --check [目標檔案]

# 若有前端測試框架（如 Jest）
npm test -- [相關測試檔案]
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

**Java 專案**：
```bash
# 確認編譯通過
./mvnw compile

# 執行所有相關測試
./mvnw test

# 檢查是否有遺漏的 commit
git status
```

**JavaScript 檔案**：
```bash
# 確認所有 JS 檔案語法正確
for file in src/main/resources/static/js/*.js; do
  node --check "$file" 2>&1 || echo "❌ $file 語法錯誤"
done

# 確認專案編譯通過（若 JS 與後端整合）
./mvnw compile

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

## 4. JavaScript 重構手法詳解

### 4.1 Extract Function（提取函式）

**適用情境**：函式過長、程式碼區塊可獨立命名

```javascript
// ❌ 重構前：函式過長
function loadCases(page, size) {
    // 顯示載入中
    var loadingEl = document.getElementById('loading');
    loadingEl.style.display = 'block';
    var listEl = document.getElementById('caseList');
    listEl.innerHTML = '';

    // 建立請求參數
    var params = {
        page: page,
        size: size,
        status: document.getElementById('statusFilter').value,
        keyword: document.getElementById('searchInput').value
    };

    // 發送請求...
}

// ✅ 重構後：提取為獨立函式
function loadCases(page, size) {
    // 顯示載入狀態
    showLoading();
    // 清空列表
    clearCaseList();
    // 取得查詢參數
    var params = buildSearchParams(page, size);
    // 發送請求...
}

/**
 * 顯示載入中狀態
 */
function showLoading() {
    // 取得載入元素
    var loadingEl = document.getElementById('loading');
    // 顯示載入元素
    loadingEl.style.display = 'block';
}

/**
 * 清空案件列表
 */
function clearCaseList() {
    // 取得列表元素
    var listEl = document.getElementById('caseList');
    // 清空列表內容
    listEl.innerHTML = '';
}

/**
 * 建立搜尋參數物件
 * @param {number} page - 頁碼
 * @param {number} size - 每頁筆數
 * @returns {Object} 搜尋參數物件
 */
function buildSearchParams(page, size) {
    // 回傳參數物件
    return {
        page: page,
        size: size,
        status: document.getElementById('statusFilter').value,
        keyword: document.getElementById('searchInput').value
    };
}
```

### 4.2 Extract Module（提取模組）

**適用情境**：多個函式有相關性、需要封裝私有狀態

```javascript
// ❌ 重構前：散落的全域函式
var notificationCount = 0;
var notificationList = [];

function loadNotifications() { /* ... */ }
function markAsRead(id) { /* ... */ }
function clearAll() { /* ... */ }

// ✅ 重構後：封裝為模組（IIFE 模式）
var NotificationManager = (function() {
    // ========== 私有變數 ==========
    // 通知數量
    var count = 0;
    // 通知列表
    var list = [];

    // ========== 私有方法 ==========
    /**
     * 更新通知 Badge
     */
    function updateBadge() {
        // 取得 Badge 元素
        var badge = document.getElementById('notificationBadge');
        // 設定數量
        badge.textContent = count > 99 ? '99+' : count;
    }

    // ========== 公開方法 ==========
    return {
        /**
         * 載入通知列表
         */
        load: function() {
            // 載入邏輯...
        },

        /**
         * 標記為已讀
         * @param {number} id - 通知 ID
         */
        markAsRead: function(id) {
            // 標記邏輯...
            // 更新 Badge
            updateBadge();
        },

        /**
         * 清除所有通知
         */
        clearAll: function() {
            // 清除邏輯...
            // 重設計數
            count = 0;
            // 更新 Badge
            updateBadge();
        }
    };
})();
```

### 4.3 Replace Callback with Promise Chain（消除回呼地獄）

**適用情境**：多層巢狀回呼

```javascript
// ❌ 重構前：回呼地獄
function processCase(caseId) {
    Api.get('/cases/' + caseId, function(caseData) {
        Api.get('/users/' + caseData.assigneeId, function(userData) {
            Api.get('/categories/' + caseData.categoryId, function(categoryData) {
                // 處理資料...
                renderCase(caseData, userData, categoryData);
            }, function(error) {
                showError('載入類別失敗');
            });
        }, function(error) {
            showError('載入使用者失敗');
        });
    }, function(error) {
        showError('載入案件失敗');
    });
}

// ✅ 重構後：使用 Promise 鏈（保持傳統語法）
function processCase(caseId) {
    // 宣告資料暫存變數
    var caseData = null;
    var userData = null;

    // 載入案件資料
    Api.get('/cases/' + caseId)
        .then(function(response) {
            // 儲存案件資料
            caseData = response.data;
            // 載入使用者資料
            return Api.get('/users/' + caseData.assigneeId);
        })
        .then(function(response) {
            // 儲存使用者資料
            userData = response.data;
            // 載入類別資料
            return Api.get('/categories/' + caseData.categoryId);
        })
        .then(function(response) {
            // 取得類別資料
            var categoryData = response.data;
            // 渲染案件
            renderCase(caseData, userData, categoryData);
        })
        .catch(function(error) {
            // 顯示錯誤訊息
            console.error('處理案件失敗:', error);
            showError('處理案件失敗');
        });
}
```

### 4.4 Extract Constants（提取常數）

**適用情境**：魔術數字、魔術字串

```javascript
// ❌ 重構前：魔術數字散落各處
function checkPassword(password) {
    if (password.length < 8) {
        return '密碼長度至少 8 個字元';
    }
    if (password.length > 20) {
        return '密碼長度不可超過 20 個字元';
    }
    // ...
}

function setPollingInterval() {
    setInterval(checkStatus, 30000);  // 30 秒
}

// ✅ 重構後：提取為常數
// ========== 常數定義 ==========
// 密碼最小長度
var PASSWORD_MIN_LENGTH = 8;
// 密碼最大長度
var PASSWORD_MAX_LENGTH = 20;
// 狀態檢查間隔（毫秒）
var STATUS_CHECK_INTERVAL = 30000;

function checkPassword(password) {
    // 檢查密碼最小長度
    if (password.length < PASSWORD_MIN_LENGTH) {
        return '密碼長度至少 ' + PASSWORD_MIN_LENGTH + ' 個字元';
    }
    // 檢查密碼最大長度
    if (password.length > PASSWORD_MAX_LENGTH) {
        return '密碼長度不可超過 ' + PASSWORD_MAX_LENGTH + ' 個字元';
    }
    // ...
}

function setPollingInterval() {
    // 設定狀態檢查定時器
    setInterval(checkStatus, STATUS_CHECK_INTERVAL);
}
```

---

## 5. 技術規範遵守

### 5.1 Java 技術規範

重構過程中必須遵守專案的技術規範：

| 規範 | 要求 |
|------|------|
| **Lombok** | 禁止使用，必須手寫 Getter/Setter |
| **Lambda** | 禁止使用，改用傳統 for-loop |
| **Stream API** | 禁止使用，改用傳統迭代 |
| **註解** | 每行邏輯程式碼必須有繁體中文註解 |

### 5.2 JavaScript 技術規範

| 規範 | 要求 |
|------|------|
| **Lambda/箭頭函式** | 禁止使用 `=>` 箭頭函式，改用傳統 `function` |
| **ES6+ 語法** | 避免使用 `let`、`const`、解構賦值等，使用 `var` |
| **模板字串** | 禁止使用 `` ` ` `` 模板字串，改用字串拼接 `+` |
| **註解** | 每行邏輯程式碼必須有繁體中文註解 |
| **console.log** | 正式環境應移除或改用 console.error |
| **全域變數** | 避免污染全域命名空間，使用 IIFE 或模組模式封裝 |

**JavaScript 禁用語法檢查**：
```bash
# 檢查是否誤用箭頭函式
grep -rn "=>" --include="*.js" src/main/resources/static/js/ | grep -v "//\|/\*"

# 檢查是否使用 let/const
grep -rn "\blet\b\|\bconst\b" --include="*.js" src/main/resources/static/js/ | grep -v "//\|/\*"

# 檢查是否使用模板字串
grep -rn '`' --include="*.js" src/main/resources/static/js/ | grep -v "//\|/\*"
```

---

## 6. Skill 串接機制（強制執行）

### 6.1 重構完成 → CodeReview

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

### 6.2 從 CodeReview 進入重構

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

## 7. 禁止行為

- ❌ 禁止未掃描就開始重構
- ❌ 禁止未經使用者確認就執行重構
- ❌ 禁止一次重構多個不相關的項目
- ❌ 禁止重構後不執行測試
- ❌ 禁止重構過程中偷偷修改業務邏輯
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動重構

---

## 8. 相關文件

- [PATTERNS.md](PATTERNS.md) - 程式碼壞味道與重構手法對照表
- [CHECKLIST.md](CHECKLIST.md) - 重構前後檢查清單
