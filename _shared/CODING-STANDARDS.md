# 共享編碼規範

> 本文件為跨技術棧共用的編碼規範，所有「開發-*」與「CodeReview」、「重構」SKILL 皆引用此文件。
> 修改任何規範時，只需修改此檔案，所有 SKILL 將同步生效。

---

## 目錄

1. [Controller / Service 架構規範](#1-controller--service-架構規範)
2. [註解規範](#2-註解規範)
3. [Exception 處理與日誌規範](#3-exception-處理與日誌規範)
4. [資料庫異動規範](#4-資料庫異動規範)
5. [命名與檔案規範](#5-命名與檔案規範)
6. [程式碼品質要求](#6-程式碼品質要求)

---

## 1. Controller / Service 架構規範

> ⚠️ **最高原則**：每一個前端網頁功能，對應到後端必須有專用的 RestController（Java）或 Router（Python），絕對禁止跨 Controller / Router 使用，否則會造成權限控管混亂。

### 1.1 架構設計原則

| 規則 | 說明 |
|------|------|
| **專用 Controller / Router** | 每個前端頁面 / 功能模組對應一個專用的 Controller 或 Router |
| **專用 Service** | Controller 專用的方法，放在同名的 Service 中（如 `UserController` → `UserService`） |
| **禁止跨 Controller** | ❌ 禁止一個網頁功能呼叫其他 Controller 的內部 API 或 Service |
| **權限獨立** | 每個 Controller 有獨立的權限控制，不與其他功能混用 |

### 1.2 共用 Service vs 專用 Service 判斷標準

| Service 類型 | 判斷標準 | 範例 |
|-------------|---------|------|
| **共用 Service** | 被 2+ 個 Controller 使用、提供基礎功能 | `UserService`、`NotificationService` |
| **專用 Service** | 只被 1 個 Controller 使用、提供頁面特定功能 | `<功能>Service`（與 Controller 同名） |

### 1.3 開發階段檢查項目

1. **功能是否有專用 Controller**
   - ✅ 有 → 在該 Controller 中新增 / 修改 API
   - ❌ 沒有 → 建立新的專用 Controller

2. **方法應放在哪個 Service**
   - 若方法**只被此 Controller 使用** → 放在專用 Service（與 Controller 同名）
   - 若方法**被多個 Controller 共用** → 放在通用 Service

3. **是否有跨 Controller 呼叫**
   - 發現跨 Controller 呼叫 → 標記為 🔴 架構違規，必須修正

---

## 2. 註解規範

### 2.1 通則

- **語言**：所有註解使用繁體中文
- **逐行註解**：每一行具備邏輯意義的程式碼都必須附上繁體中文註解
- **註解內容**：應精確說明該行的「邏輯目的」（為什麼做），不只是描述程式碼字面意義

### 2.2 各語言註解格式

| 語言 | 行內註解 | 區塊註解 | 函式說明 |
|------|---------|---------|---------|
| **Java** | `// 註解` | `/* ... */` | `/** Javadoc */` |
| **Python** | `# 註解` | `""" ... """` | Function Docstring |
| **JSX / JS** | `// 註解` | `{/* JSX 註解 */}` | JSDoc `/** ... */` |
| **HTML / Thymeleaf** | `<!-- 註解 -->` | - | - |

### 2.3 排除項目（不需要註解）

- 空白行
- 僅有括號的行 `{`、`}`、`(`、`)`
- import / package 語句
- 註解本身

---

## 3. Exception 處理與日誌規範

> ⚠️ **最高原則**：所有 Exception 必須記錄完整的堆疊追蹤（Stack Trace），禁止吞掉例外或僅記錄訊息。

### 3.1 強制規範

| 規則 | 說明 |
|------|------|
| **必須使用 log.error / logger.error** | 捕獲 Exception 時，必須呼叫 logger 記錄錯誤 |
| **必須包含堆疊追蹤** | 錯誤訊息必須包含 Exception 物件本身 |
| **禁止僅記錄訊息** | ❌ 禁止僅記錄 `e.getMessage()` 這種遺失堆疊的寫法 |
| **禁止吞掉例外** | ❌ 禁止空的 catch 區塊或僅 `e.printStackTrace()` |

### 3.2 Java 範例

```java
// ✅ 正確：包含錯誤描述與完整堆疊追蹤
try {
    userService.createUser(userDto);
} catch (Exception e) {
    log.error("建立使用者失敗，userId: {}", userDto.getUserId(), e);
    throw new BusinessException("使用者建立失敗", e);
}

// ❌ 錯誤 1：僅記錄訊息，遺失堆疊追蹤
log.error("發生錯誤: " + e.getMessage());

// ❌ 錯誤 2:空的 catch 區塊（吞掉例外）
catch (Exception e) { /* do nothing */ }

// ❌ 錯誤 3：僅使用 printStackTrace
e.printStackTrace();
```

### 3.3 Python 範例

```python
# ✅ 正確：使用 exc_info=True 記錄完整堆疊
try:
    user = await create_user(user_data)
except IntegrityError as e:
    logger.error("建立使用者失敗，username: %s", user_data.username, exc_info=True)
    raise HTTPException(status_code=409, detail="使用者名稱已存在") from e

# ❌ 錯誤 1：bare except
try: ...
except: pass

# ❌ 錯誤 2：僅記錄訊息
except Exception as e:
    logger.error(f"失敗: {e}")

# ❌ 錯誤 3：使用 print
except Exception as e:
    print(e)
```

### 3.4 Logger 宣告

| 語言 | 標準寫法 |
|------|---------|
| **Java** | `private static final Logger log = LoggerFactory.getLogger(<ClassName>.class);` |
| **Python** | `logger = logging.getLogger(__name__)`（模組層級宣告） |

---

## 4. 資料庫異動規範

> 所有資料庫結構異動（DDL）必須透過 Migration 工具管理，禁止手動操作資料庫。

### 4.1 通則

- ❌ 禁止直接對資料庫執行 CREATE、ALTER、DROP 等 DDL 語句
- ❌ 禁止使用 DBeaver、pgAdmin、MySQL Workbench 等工具直接修改資料庫結構
- ❌ 禁止在程式碼中使用會自動建表的設定（如 `spring.jpa.hibernate.ddl-auto=update/create`）

### 4.2 Migration 工具對應

| 技術棧 | 工具 | 規範 |
|-------|------|------|
| **Java + Spring Boot** | Liquibase | 使用 XML 或 YAML changelog；每個 changeset 必須有唯一的 `id` 與 `author`；已執行的 changeset 禁止修改；必須包含 `rollback` 區塊 |
| **Python + FastAPI** | Alembic | 使用 `alembic revision --autogenerate -m "描述"`；已執行的 migration 禁止修改 |

### 4.3 命名規範

- changelog / migration 檔案以日期或版本號開頭，例：`20260411-add-user-table.xml`

---

## 5. 命名與檔案規範

### 5.1 設定檔

| 規則 | 正確 | 錯誤 |
|------|------|------|
| Spring Boot 設定統一使用 YAML | `application.yml` | ❌ `application.properties` |
| 環境變數範例檔不加前綴點（避免被檔案管理器隱藏） | `env.example` | ❌ `.env.example` |

### 5.2 各語言命名慣例

| 語言 | 類別 / 元件 | 函式 / 方法 | 常數 | 變數 |
|------|----------|----------|------|------|
| Java | PascalCase | camelCase | UPPER_SNAKE_CASE | camelCase |
| Python | PascalCase | snake_case | UPPER_SNAKE_CASE | snake_case |
| React (JSX) | PascalCase | camelCase | UPPER_SNAKE_CASE | camelCase |

---

## 6. 程式碼品質要求

### 6.1 通則

- 必須包含完整的 Null Check 與異常處理機制
- 所有對外 API 必須有輸入驗證（Input Validation）
- 確保程式碼符合資深開發者的高可讀性與穩定性要求

### 6.2 方法 / 函式長度建議

| 項目 | 建議標準 |
|------|---------|
| 單一方法 / 函式長度 | 不超過 50 行 |
| 單一類別 / 模組長度 | 不超過 500 行 |
| 方法參數數量 | 不超過 5 個 |
| if / for 巢狀深度 | 不超過 3 層 |

超出標準時，應使用「重構」SKILL 進行 Extract Method / Extract Class。
