# 程式碼壞味道與重構手法對照表

> 本文件提供常見的程式碼壞味道（Code Smells）識別方式與對應的重構手法

---

## 目錄

1. [方法層級壞味道](#1-方法層級壞味道)
2. [類別層級壞味道](#2-類別層級壞味道)
3. [重複程式碼壞味道](#3-重複程式碼壞味道)
4. [資料相關壞味道](#4-資料相關壞味道)
5. [條件邏輯壞味道](#5-條件邏輯壞味道)
6. [重構手法速查表](#6-重構手法速查表)

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
