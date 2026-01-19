# 程式碼壞味道與重構手法對照表

> 本文件提供常見的程式碼壞味道（Code Smells）識別方式與對應的重構手法
>
> **支援語言**：Java、JavaScript

---

## 目錄

1. [方法層級壞味道](#1-方法層級壞味道)
2. [類別層級壞味道](#2-類別層級壞味道)
3. [重複程式碼壞味道](#3-重複程式碼壞味道)
4. [資料相關壞味道](#4-資料相關壞味道)
5. [條件邏輯壞味道](#5-條件邏輯壞味道)
6. [重構手法速查表](#6-重構手法速查表)
7. [JavaScript 專用壞味道](#7-javascript-專用壞味道)

---

## 1. 方法層級壞味道

### 1.1 Long Method（過長方法）

**識別標準**：
- 方法超過 50 行
- 需要捲動才能看完
- 方法內有多個邏輯區塊
- 方法做了太多事情

**檢測指令**：
```bash
# 列出 Java 檔案中的方法定義，人工檢視行數
grep -n "public\|private\|protected" --include="*.java" [檔案路徑]
```

**重構手法**：

| 手法 | 適用情境 |
|------|---------|
| Extract Method | 將邏輯區塊提取為獨立方法 |
| Replace Temp with Query | 將暫存變數改為方法呼叫 |
| Decompose Conditional | 將複雜條件式提取為方法 |

**範例**：

```java
// ❌ 過長方法
public void processOrder(Order order) {
    // 第一段：驗證（15行）
    // 第二段：計算（20行）
    // 第三段：儲存（15行）
    // 第四段：通知（10行）
    // 總共 60 行
}

// ✅ 重構後
public void processOrder(Order order) {
    // 驗證訂單資料
    validateOrder(order);
    // 計算訂單金額
    calculateOrderAmount(order);
    // 儲存訂單到資料庫
    saveOrder(order);
    // 發送訂單通知
    sendOrderNotification(order);
}
```

---

### 1.2 Long Parameter List（過長參數列）

**識別標準**：
- 方法參數超過 5 個
- 參數之間有關聯性
- 多個方法有相似的參數組合

**重構手法**：

| 手法 | 適用情境 |
|------|---------|
| Introduce Parameter Object | 將相關參數封裝為物件 |
| Preserve Whole Object | 傳遞整個物件而非個別屬性 |

**範例**：

```java
// ❌ 過長參數列
public void createUser(String name, String email, String phone,
                       String address, String city, String zipCode) {
    // ...
}

// ✅ 重構後：使用參數物件
public void createUser(UserCreateRequest request) {
    // 取得使用者姓名
    String name = request.getName();
    // 取得使用者信箱
    String email = request.getEmail();
    // ...
}
```

---

### 1.3 Too Many Local Variables（過多區域變數）

**識別標準**：
- 方法內有超過 7 個區域變數
- 變數只用於暫存中間結果
- 變數的生命週期過長

**重構手法**：

| 手法 | 說明 |
|------|------|
| Extract Method | 將使用這些變數的程式碼提取為方法 |
| Replace Temp with Query | 將暫存變數改為方法呼叫 |
| Split Temporary Variable | 將多用途變數拆分 |

---

## 2. 類別層級壞味道

### 2.1 Large Class（龐大類別）

**識別標準**：
- 類別超過 500 行
- 類別有太多欄位
- 類別有太多方法
- 類別承擔太多職責

**檢測指令**：
```bash
# 計算檔案行數
wc -l [檔案路徑]

# 列出類別的所有方法
grep -n "public\|private\|protected" --include="*.java" [檔案路徑] | grep -v "class"
```

**重構手法**：

| 手法 | 適用情境 |
|------|---------|
| Extract Class | 將相關欄位和方法提取為新類別 |
| Extract Subclass | 將部分行為提取為子類別 |
| Extract Interface | 將共用介面提取出來 |

**範例**：

```java
// ❌ 龐大類別：一個類別做太多事
public class UserService {
    // 使用者 CRUD（100行）
    // 使用者驗證（80行）
    // 使用者權限（70行）
    // 使用者通知（60行）
    // 總共 310 行
}

// ✅ 重構後：拆分為多個類別
public class UserService {
    // 使用者 CRUD（100行）
}

public class UserAuthService {
    // 使用者驗證（80行）
}

public class UserPermissionService {
    // 使用者權限（70行）
}

public class UserNotificationService {
    // 使用者通知（60行）
}
```

---

### 2.2 God Class（上帝類別）

**識別標準**：
- 類別知道太多其他類別的細節
- 類別被太多其他類別依賴
- 修改任何功能都要改這個類別

**重構手法**：

| 手法 | 說明 |
|------|------|
| Extract Class | 依職責拆分為多個類別 |
| Move Method | 將方法移到更適合的類別 |
| Move Field | 將欄位移到更適合的類別 |

---

### 2.3 Feature Envy（依戀情結）

**識別標準**：
- 方法大量使用其他類別的資料
- 方法對自己類別的資料使用很少
- 方法更像是屬於另一個類別

**重構手法**：

| 手法 | 說明 |
|------|------|
| Move Method | 將方法移到它最常使用的類別 |
| Extract Method + Move Method | 先提取再移動 |

**範例**：

```java
// ❌ 依戀情結：方法過度使用 Order 的資料
public class ReportService {
    public String generateOrderReport(Order order) {
        // 取得訂單編號
        String orderId = order.getId();
        // 取得訂單日期
        Date orderDate = order.getDate();
        // 取得訂單金額
        BigDecimal amount = order.getAmount();
        // 取得訂單狀態
        String status = order.getStatus();
        // 組裝報表...
        return orderId + "|" + orderDate + "|" + amount + "|" + status;
    }
}

// ✅ 重構後：將方法移到 Order 類別
public class Order {
    public String toReportString() {
        // 組裝報表格式
        return this.id + "|" + this.date + "|" + this.amount + "|" + this.status;
    }
}
```

---

## 3. 重複程式碼壞味道

### 3.1 Duplicated Code（重複程式碼）

**識別標準**：
- 相同程式碼出現 2 次以上
- 相似程式碼只有些微差異
- 複製貼上後只改參數

**檢測指令**：
```bash
# 搜尋可能重複的程式碼片段
grep -rn "[疑似重複的關鍵字或程式碼]" --include="*.java" src/
```

**重構手法依情境**：

| 情境 | 重構手法 |
|------|---------|
| 同一類別內重複 | Extract Method |
| 兄弟類別間重複 | Extract Method → Pull Up Method（移到父類別） |
| 不相關類別間重複 | Extract Class（建立共用工具類別） |

**範例**：

```java
// ❌ 重複程式碼：驗證邏輯散落各處
public class UserController {
    public void createUser(User user) {
        if (user == null) {
            throw new IllegalArgumentException("參數不可為空");
        }
        if (user.getName() == null || user.getName().trim().isEmpty()) {
            throw new IllegalArgumentException("姓名不可為空");
        }
        // ...
    }
}

public class OrderController {
    public void createOrder(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("參數不可為空");
        }
        if (order.getItems() == null || order.getItems().isEmpty()) {
            throw new IllegalArgumentException("訂單項目不可為空");
        }
        // ...
    }
}

// ✅ 重構後：提取共用工具類別
public class ValidationUtils {

    /**
     * 驗證物件不可為 null
     */
    public static void requireNonNull(Object obj, String message) {
        if (obj == null) {
            throw new IllegalArgumentException(message);
        }
    }

    /**
     * 驗證字串不可為空
     */
    public static void requireNonEmpty(String str, String message) {
        if (str == null || str.trim().isEmpty()) {
            throw new IllegalArgumentException(message);
        }
    }

    /**
     * 驗證集合不可為空
     */
    public static void requireNonEmpty(Collection<?> collection, String message) {
        if (collection == null || collection.isEmpty()) {
            throw new IllegalArgumentException(message);
        }
    }
}
```

---

### 3.2 Similar Code（相似程式碼）

**識別標準**：
- 程式碼結構相似但有部分差異
- 只有個別變數或方法呼叫不同
- 可以用參數化來統一

**重構手法**：

| 手法 | 說明 |
|------|------|
| Form Template Method | 將相同部分提取為模板方法 |
| Parameterize Method | 將差異部分參數化 |

**範例**：

```java
// ❌ 相似程式碼
public void sendEmailNotification(User user, String message) {
    // 檢查使用者是否有信箱
    if (user.getEmail() == null) {
        return;
    }
    // 建立信件內容
    String content = "親愛的 " + user.getName() + "，" + message;
    // 發送 Email
    emailService.send(user.getEmail(), content);
    // 記錄日誌
    logger.info("已發送 Email 通知給 " + user.getName());
}

public void sendSmsNotification(User user, String message) {
    // 檢查使用者是否有手機
    if (user.getPhone() == null) {
        return;
    }
    // 建立簡訊內容
    String content = "親愛的 " + user.getName() + "，" + message;
    // 發送簡訊
    smsService.send(user.getPhone(), content);
    // 記錄日誌
    logger.info("已發送簡訊通知給 " + user.getName());
}

// ✅ 重構後：使用策略模式或模板方法
public interface NotificationSender {
    boolean canSend(User user);
    void send(User user, String content);
    String getType();
}

public void sendNotification(User user, String message, NotificationSender sender) {
    // 檢查是否可發送
    if (!sender.canSend(user)) {
        return;
    }
    // 建立通知內容
    String content = "親愛的 " + user.getName() + "，" + message;
    // 發送通知
    sender.send(user, content);
    // 記錄日誌
    logger.info("已發送" + sender.getType() + "通知給 " + user.getName());
}
```

---

## 4. 資料相關壞味道

### 4.1 Data Clumps（資料泥團）

**識別標準**：
- 多個欄位總是一起出現
- 多個參數總是一起傳遞
- 這些資料有邏輯關聯

**重構手法**：

| 手法 | 說明 |
|------|------|
| Extract Class | 將相關資料提取為新類別 |
| Introduce Parameter Object | 將相關參數封裝為物件 |

**範例**：

```java
// ❌ 資料泥團：地址相關欄位總是一起出現
public class User {
    private String name;
    private String streetAddress;  // 總是一起使用
    private String city;           // 總是一起使用
    private String zipCode;        // 總是一起使用
    private String country;        // 總是一起使用
}

public void printAddress(String street, String city, String zip, String country) {
    // ...
}

// ✅ 重構後：提取 Address 類別
public class Address {
    private String streetAddress;
    private String city;
    private String zipCode;
    private String country;

    // Getter/Setter...

    public String getFullAddress() {
        return this.zipCode + " " + this.city + " " + this.streetAddress;
    }
}

public class User {
    private String name;
    private Address address;  // 封裝為物件
}
```

---

### 4.2 Primitive Obsession（基本型別偏執）

**識別標準**：
- 使用基本型別（String, int）表達業務概念
- 同一個型別在不同地方代表不同意義
- 需要額外的驗證邏輯來確保資料正確

**重構手法**：

| 手法 | 說明 |
|------|------|
| Replace Type Code with Class | 用類別取代型別代碼 |
| Replace Data Value with Object | 用物件取代基本型別 |

**範例**：

```java
// ❌ 基本型別偏執
public class Order {
    private String status;  // "PENDING", "PAID", "SHIPPED", "CANCELLED"

    public void setStatus(String status) {
        // 需要額外驗證
        if (!"PENDING".equals(status) && !"PAID".equals(status)
            && !"SHIPPED".equals(status) && !"CANCELLED".equals(status)) {
            throw new IllegalArgumentException("無效的訂單狀態");
        }
        this.status = status;
    }
}

// ✅ 重構後：使用列舉
public enum OrderStatus {
    PENDING("待處理"),
    PAID("已付款"),
    SHIPPED("已出貨"),
    CANCELLED("已取消");

    private String displayName;

    OrderStatus(String displayName) {
        this.displayName = displayName;
    }

    public String getDisplayName() {
        return this.displayName;
    }
}

public class Order {
    private OrderStatus status;  // 型別安全
}
```

---

## 5. 條件邏輯壞味道

### 5.1 Nested Conditionals（巢狀條件式）

**識別標準**：
- if-else 巢狀超過 3 層
- 條件邏輯難以閱讀
- 難以理解程式流程

**重構手法**：

| 手法 | 說明 |
|------|------|
| Replace Nested Conditional with Guard Clauses | 使用衛述句提前返回 |
| Decompose Conditional | 將條件提取為有意義的方法 |

**範例**：

```java
// ❌ 過深巢狀
public double calculateDiscount(Order order) {
    double discount = 0;
    if (order != null) {
        if (order.getCustomer() != null) {
            if (order.getCustomer().isVip()) {
                if (order.getAmount() > 1000) {
                    discount = 0.2;
                } else {
                    discount = 0.1;
                }
            } else {
                if (order.getAmount() > 1000) {
                    discount = 0.05;
                }
            }
        }
    }
    return discount;
}

// ✅ 重構後：使用衛述句
public double calculateDiscount(Order order) {
    // 檢查訂單是否有效
    if (order == null) {
        return 0;
    }
    // 檢查客戶是否存在
    if (order.getCustomer() == null) {
        return 0;
    }
    // 判斷是否為 VIP 客戶
    boolean isVip = order.getCustomer().isVip();
    // 判斷是否為大額訂單
    boolean isLargeOrder = order.getAmount() > 1000;

    // VIP 大額訂單：8折
    if (isVip && isLargeOrder) {
        return 0.2;
    }
    // VIP 一般訂單：9折
    if (isVip) {
        return 0.1;
    }
    // 一般客戶大額訂單：95折
    if (isLargeOrder) {
        return 0.05;
    }
    // 無折扣
    return 0;
}
```

---

### 5.2 Complex Conditional（複雜條件式）

**識別標準**：
- 單一條件式過長
- 使用多個 && 或 ||
- 條件意義不明確

**重構手法**：

| 手法 | 說明 |
|------|------|
| Decompose Conditional | 將條件提取為具名方法 |
| Consolidate Conditional Expression | 合併相同結果的條件 |

**範例**：

```java
// ❌ 複雜條件式
if (user != null && user.getStatus() != null
    && "ACTIVE".equals(user.getStatus())
    && user.getRole() != null
    && ("ADMIN".equals(user.getRole()) || "SUPER_ADMIN".equals(user.getRole()))
    && user.getLastLoginDate() != null
    && user.getLastLoginDate().after(thirtyDaysAgo)) {
    // 允許存取
}

// ✅ 重構後：提取為有意義的方法
if (isActiveAdmin(user) && hasRecentLogin(user)) {
    // 允許存取
}

private boolean isActiveAdmin(User user) {
    // 檢查使用者是否為空
    if (user == null) {
        return false;
    }
    // 檢查使用者是否啟用
    if (!"ACTIVE".equals(user.getStatus())) {
        return false;
    }
    // 檢查是否為管理員角色
    String role = user.getRole();
    return "ADMIN".equals(role) || "SUPER_ADMIN".equals(role);
}

private boolean hasRecentLogin(User user) {
    // 檢查最後登入日期是否存在
    if (user.getLastLoginDate() == null) {
        return false;
    }
    // 檢查是否在 30 天內登入過
    Date thirtyDaysAgo = getThirtyDaysAgo();
    return user.getLastLoginDate().after(thirtyDaysAgo);
}
```

---

## 6. 重構手法速查表

### 6.1 依壞味道查詢

| 壞味道 | 首選手法 | 備選手法 |
|-------|---------|---------|
| 方法過長 | Extract Method | Decompose Conditional |
| 過長參數列 | Introduce Parameter Object | Preserve Whole Object |
| 龐大類別 | Extract Class | Extract Subclass |
| 重複程式碼（同類別） | Extract Method | - |
| 重複程式碼（跨類別） | Extract Class | Pull Up Method |
| 資料泥團 | Extract Class | Introduce Parameter Object |
| 巢狀條件式 | Replace with Guard Clauses | Decompose Conditional |
| 複雜條件式 | Decompose Conditional | Consolidate Conditional |
| 依戀情結 | Move Method | Extract Method + Move |
| 基本型別偏執 | Replace with Class/Enum | - |

### 6.2 依手法查詢

| 重構手法 | 適用情境 | 風險等級 |
|---------|---------|---------|
| Extract Method | 方法過長、程式碼重複 | 低 |
| Extract Class | 類別過大、資料泥團 | 中 |
| Move Method | 方法放錯類別、依戀情結 | 中 |
| Inline Method | 方法過於瑣碎、間接層過多 | 低 |
| Replace Temp with Query | 暫存變數過多 | 低 |
| Introduce Parameter Object | 參數列過長 | 中 |
| Replace Conditional with Guard Clauses | 巢狀條件式 | 低 |
| Decompose Conditional | 複雜條件式 | 低 |

### 6.3 風險等級說明

| 等級 | 說明 | 建議 |
|------|------|------|
| 低 | 局部變更，不影響其他程式碼 | 可較快速執行 |
| 中 | 可能影響其他類別或介面 | 需確認測試覆蓋 |
| 高 | 影響範圍大，可能有副作用 | 需謹慎規劃，分步執行 |

---

## 7. JavaScript 專用壞味道

### 7.1 Global Variable Pollution（全域變數污染）

**識別標準**：
- 在函式外使用 `var` 宣告變數
- 未使用 `var` 宣告變數（隱式全域）
- 多個 JS 檔案共用相同的全域變數名稱

**檢測指令**：
```bash
# 搜尋可能的全域變數（簡易檢測）
grep -n "^var \|^let \|^const " --include="*.js" [檔案路徑]
```

**重構手法**：

| 手法 | 說明 |
|------|------|
| Wrap in IIFE | 將程式碼包在立即執行函式中 |
| Extract Module | 使用模組模式封裝 |
| Namespace Object | 將相關變數放入命名空間物件 |

**範例**：

```javascript
// ❌ 全域變數污染
var userName = '';
var userEmail = '';
var isLoggedIn = false;

function login(name, email) {
    userName = name;
    userEmail = email;
    isLoggedIn = true;
}

// ✅ 重構後：使用模組模式封裝
var UserModule = (function() {
    // 私有變數
    var userName = '';
    var userEmail = '';
    var isLoggedIn = false;

    // 公開方法
    return {
        login: function(name, email) {
            userName = name;
            userEmail = email;
            isLoggedIn = true;
        },
        isLoggedIn: function() {
            return isLoggedIn;
        },
        getUserName: function() {
            return userName;
        }
    };
})();
```

---

### 7.2 Callback Hell（回呼地獄）

**識別標準**：
- 回呼函式嵌套超過 3 層
- 程式碼向右縮排過深
- 難以追蹤錯誤處理

**檢測指令**：
```bash
# 搜尋多層嵌套的 function
grep -n "function.*function.*function" --include="*.js" [檔案路徑]

# 搜尋 .then 嵌套
grep -n "\.then.*\.then" --include="*.js" [檔案路徑]
```

**重構手法**：

| 手法 | 說明 |
|------|------|
| Extract Named Functions | 將回呼提取為具名函式 |
| Use Promise Chain | 使用 Promise 鏈扁平化 |
| Early Return | 提前返回減少嵌套 |

**範例**：

```javascript
// ❌ 回呼地獄
function processOrder(orderId) {
    getOrder(orderId, function(order) {
        getCustomer(order.customerId, function(customer) {
            getInventory(order.items, function(inventory) {
                processPayment(order, customer, function(result) {
                    sendConfirmation(customer.email, function() {
                        // 完成...
                    });
                });
            });
        });
    });
}

// ✅ 重構後：提取具名函式 + Promise 鏈
function processOrder(orderId) {
    // 宣告資料暫存
    var orderData = null;
    var customerData = null;

    // 取得訂單
    getOrder(orderId)
        .then(function(order) {
            // 儲存訂單資料
            orderData = order;
            // 取得客戶
            return getCustomer(order.customerId);
        })
        .then(function(customer) {
            // 儲存客戶資料
            customerData = customer;
            // 檢查庫存
            return getInventory(orderData.items);
        })
        .then(function(inventory) {
            // 處理付款
            return processPayment(orderData, customerData);
        })
        .then(function(result) {
            // 發送確認信
            return sendConfirmation(customerData.email);
        })
        .then(function() {
            // 完成處理
        })
        .catch(function(error) {
            // 統一錯誤處理
            console.error('訂單處理失敗:', error);
        });
}
```

---

### 7.3 Magic Numbers/Strings（魔術數字/字串）

**識別標準**：
- 程式碼中直接使用未命名的數字
- 程式碼中直接使用字串字面值
- 相同的值在多處重複出現

**檢測指令**：
```bash
# 搜尋可能的魔術數字
grep -n "[^a-zA-Z0-9_][0-9]\{2,\}[^a-zA-Z0-9_]" --include="*.js" [檔案路徑]

# 搜尋重複的字串
grep -on "'[^']\{5,\}'" --include="*.js" [檔案路徑] | sort | uniq -c | sort -rn
```

**重構手法**：

| 手法 | 說明 |
|------|------|
| Extract Constant | 提取為具名常數 |
| Configuration Object | 集中管理設定值 |

**範例**：

```javascript
// ❌ 魔術數字/字串散落各處
function validatePassword(password) {
    if (password.length < 8) {
        return '密碼太短';
    }
    if (password.length > 20) {
        return '密碼太長';
    }
    return null;
}

function setAutoRefresh() {
    setInterval(refreshData, 30000);
}

function showToast(message) {
    // 顯示 3 秒
    setTimeout(hideToast, 3000);
}

// ✅ 重構後：提取為常數
// ========== 常數定義 ==========
// 密碼最小長度
var PASSWORD_MIN_LENGTH = 8;
// 密碼最大長度
var PASSWORD_MAX_LENGTH = 20;
// 自動更新間隔（毫秒）
var AUTO_REFRESH_INTERVAL = 30000;
// Toast 顯示時間（毫秒）
var TOAST_DISPLAY_DURATION = 3000;

function validatePassword(password) {
    // 檢查最小長度
    if (password.length < PASSWORD_MIN_LENGTH) {
        return '密碼太短';
    }
    // 檢查最大長度
    if (password.length > PASSWORD_MAX_LENGTH) {
        return '密碼太長';
    }
    // 驗證通過
    return null;
}

function setAutoRefresh() {
    // 設定自動更新定時器
    setInterval(refreshData, AUTO_REFRESH_INTERVAL);
}

function showToast(message) {
    // 設定 Toast 自動隱藏
    setTimeout(hideToast, TOAST_DISPLAY_DURATION);
}
```

---

### 7.4 Long Function（過長函式）

**識別標準**：
- 函式超過 50 行
- 函式內有多個邏輯區塊
- 函式做了太多事情
- 需要捲動才能看完

**檢測指令**：
```bash
# 統計 JS 檔案行數
wc -l src/main/resources/static/js/*.js | sort -rn

# 列出函式定義
grep -n "function " --include="*.js" [檔案路徑]
```

**重構手法**：

| 手法 | 說明 |
|------|------|
| Extract Function | 將程式碼區塊提取為獨立函式 |
| Decompose Conditional | 將複雜條件式提取為函式 |

**範例**：

```javascript
// ❌ 過長函式（60+ 行）
function initializePage() {
    // 載入使用者資訊（15行）
    // ...

    // 設定事件監聽（20行）
    // ...

    // 初始化表格（15行）
    // ...

    // 載入資料（15行）
    // ...
}

// ✅ 重構後：拆分為多個函式
function initializePage() {
    // 載入使用者資訊
    loadUserInfo();
    // 設定事件監聽
    setupEventListeners();
    // 初始化表格
    initializeTable();
    // 載入初始資料
    loadInitialData();
}

/**
 * 載入使用者資訊
 */
function loadUserInfo() {
    // 使用者載入邏輯...
}

/**
 * 設定事件監聽器
 */
function setupEventListeners() {
    // 事件綁定邏輯...
}

/**
 * 初始化資料表格
 */
function initializeTable() {
    // 表格初始化邏輯...
}

/**
 * 載入初始資料
 */
function loadInitialData() {
    // 資料載入邏輯...
}
```

---

### 7.5 Tight DOM Coupling（DOM 耦合過緊）

**識別標準**：
- 硬編碼的 DOM 選擇器散落各處
- 相同的 `getElementById` 多次呼叫
- 業務邏輯與 DOM 操作混在一起

**重構手法**：

| 手法 | 說明 |
|------|------|
| Cache DOM References | 快取 DOM 元素參照 |
| Extract DOM Operations | 將 DOM 操作提取為獨立函式 |
| Separate Concerns | 分離業務邏輯與 UI 操作 |

**範例**：

```javascript
// ❌ DOM 耦合過緊
function updateUser(userId) {
    var name = document.getElementById('userName').value;
    var email = document.getElementById('userEmail').value;

    // 驗證
    if (!name) {
        document.getElementById('nameError').style.display = 'block';
        return;
    }

    // 發送請求
    Api.put('/users/' + userId, {name: name, email: email})
        .then(function(response) {
            document.getElementById('successMsg').style.display = 'block';
            document.getElementById('userName').value = response.data.name;
        });
}

// ✅ 重構後：快取 DOM 參照 + 分離關注點
// ========== DOM 元素快取 ==========
var elements = {
    userName: document.getElementById('userName'),
    userEmail: document.getElementById('userEmail'),
    nameError: document.getElementById('nameError'),
    successMsg: document.getElementById('successMsg')
};

/**
 * 取得表單資料
 * @returns {Object} 表單資料物件
 */
function getFormData() {
    return {
        name: elements.userName.value,
        email: elements.userEmail.value
    };
}

/**
 * 顯示錯誤訊息
 * @param {string} field - 欄位名稱
 */
function showError(field) {
    // 取得對應的錯誤元素
    var errorEl = elements[field + 'Error'];
    // 顯示錯誤
    if (errorEl) {
        errorEl.style.display = 'block';
    }
}

/**
 * 顯示成功訊息
 */
function showSuccess() {
    // 顯示成功訊息
    elements.successMsg.style.display = 'block';
}

/**
 * 更新使用者
 * @param {number} userId - 使用者 ID
 */
function updateUser(userId) {
    // 取得表單資料
    var formData = getFormData();

    // 驗證姓名
    if (!formData.name) {
        showError('name');
        return;
    }

    // 發送更新請求
    Api.put('/users/' + userId, formData)
        .then(function(response) {
            // 顯示成功訊息
            showSuccess();
            // 更新表單值
            elements.userName.value = response.data.name;
        });
}
```

---

### 7.6 JavaScript 重構手法速查表

| 壞味道 | 首選手法 | 備選手法 |
|-------|---------|---------|
| 全域變數污染 | Wrap in IIFE | Extract Module |
| 回呼地獄 | Extract Named Functions | Use Promise Chain |
| 魔術數字/字串 | Extract Constant | Configuration Object |
| 函式過長 | Extract Function | Decompose Conditional |
| DOM 耦合過緊 | Cache DOM References | Separate Concerns |
| 重複程式碼 | Extract Function | Extract Module |
| 過深巢狀 | Guard Clauses | Early Return |

### 7.7 JavaScript 重構風險等級

| 重構手法 | 適用情境 | 風險等級 |
|---------|---------|---------|
| Extract Function | 函式過長 | 低 |
| Extract Constant | 魔術數字 | 低 |
| Wrap in IIFE | 全域變數 | 低 |
| Extract Module | 相關函式群組 | 中 |
| Cache DOM References | DOM 操作 | 低 |
| Promise Chain Refactor | 回呼地獄 | 中 |
| Separate Concerns | 業務邏輯混雜 | 中 |
