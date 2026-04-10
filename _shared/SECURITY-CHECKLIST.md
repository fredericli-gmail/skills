# 共享資安規範清單

> 本文件為跨技術棧共用的資安規範，所有「開發-*」與「CodeReview」SKILL 皆引用此文件。

---

## 目錄

1. [XSS 防護](#1-xss跨站腳本防護)
2. [CSRF 防護](#2-csrf跨站請求偽造防護)
3. [SQL Injection 防護](#3-sql-injection防護)
4. [認證與授權](#4-認證與授權)
5. [敏感資料保護](#5-敏感資料保護)
6. [HTTP 安全標頭](#6-http-安全標頭)
7. [Fortify / 弱掃常見問題](#7-fortify--弱掃常見問題)

---

## 1. XSS（跨站腳本）防護

### 1.1 通則

| 情境 | 規範 |
|------|------|
| 輸出使用者輸入 | 必須使用具自動 escape 的 API，禁止手動拼接 HTML |
| 動態 HTML 插入 | 禁止使用 `innerHTML`、`document.write` |
| 動態程式碼執行 | 禁止使用 `eval()`、`new Function()` |

### 1.2 各技術棧 XSS 規範

| 技術 | 安全做法 | 禁止做法 |
|------|---------|---------|
| **Thymeleaf** | `th:text`（自動 escape）、`@{/path}` URL 建構 | ❌ `th:utext`（除非經資安審核） |
| **React JSX** | JSX 自動 escape；Markdown 用 `react-markdown` 自動 sanitize | ❌ `dangerouslySetInnerHTML` |
| **JavaScript（前端）** | `textContent`、模板引擎 | ❌ `innerHTML = userInput`、`eval()` |
| **URL 參數** | `encodeURIComponent()` 或框架的 URL 建構 API | ❌ 手動字串拼接 URL |

---

## 2. CSRF（跨站請求偽造）防護

### 2.1 Session-Based（傳統 Web）

- 所有 POST、PUT、DELETE、PATCH 請求的表單必須包含 CSRF Token
- Spring Security 啟用 CSRF 保護
- Thymeleaf 表單使用 `th:action` 自動注入 CSRF Token
- AJAX 請求必須在 Header 中帶入 CSRF Token

```html
<!-- Thymeleaf：頁面 head 包含 CSRF meta -->
<meta name="_csrf" th:content="${_csrf.token}"/>
<meta name="_csrf_header" th:content="${_csrf.headerName}"/>
```

```javascript
// jQuery AJAX 取得 token
var token = $("meta[name='_csrf']").attr("content");
var header = $("meta[name='_csrf_header']").attr("content");
```

### 2.2 Token-Based（SPA / API）

- JWT Token 透過 `Authorization: Bearer <token>` Header 傳遞，天然免疫 CSRF
- Cookie 若使用，必須設定 `SameSite=Strict`
- Spring Security / FastAPI 可關閉 CSRF middleware（前提：Token 不存於 Cookie 或設定 SameSite）

---

## 3. SQL Injection 防護

> ⚠️ **絕對禁止字串拼接 SQL 語句。**

### 3.1 各技術棧安全做法

| 技術 | 安全做法 | 禁止做法 |
|------|---------|---------|
| **JPA** | `@Query("SELECT u FROM User u WHERE u.name = :name")` Named Parameter | ❌ 字串拼接 SQL |
| **JDBC** | `PreparedStatement` 的 `?` 佔位符 | ❌ `Statement` + 字串拼接 |
| **MyBatis** | `#{paramName}` 參數綁定 | ❌ `${paramName}`（除非為靜態欄位名） |
| **SQLAlchemy ORM** | `session.query(User).filter(User.id == user_id)` | ❌ `session.execute(f"SELECT ... {user_id}")` |
| **SQLAlchemy Raw** | `text("SELECT ... WHERE id = :id"), {"id": user_id}` | ❌ f-string / `%` 拼接 |

### 3.2 範例

```java
// ✅ 正確
@Query("SELECT u FROM User u WHERE u.name = :name")
User findByName(@Param("name") String name);

// ❌ 錯誤
String sql = "SELECT * FROM users WHERE name = '" + name + "'";
```

```python
# ✅ 正確
result = session.query(User).filter(User.username == username).first()
result = session.execute(text("SELECT * FROM users WHERE id = :id"), {"id": user_id})

# ❌ 錯誤
result = session.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

---

## 4. 認證與授權

### 4.1 認證機制建議

| 場景 | 建議 | 說明 |
|------|------|------|
| 傳統 SSR Web | Session + Cookie | Spring Security Session、Cookie + Redis Session |
| SPA / API | JWT | Access Token 15-30 分鐘、Refresh Token 7-30 天 |
| 第三方登入 | OAuth 2.0 | 使用成熟套件 |

### 4.2 密碼處理

- 密碼必須使用 **bcrypt** 或 **argon2** 雜湊儲存
- ❌ 禁止使用 MD5、SHA1 等弱雜湊
- 登入失敗不洩露「帳號是否存在」（統一回傳「帳號或密碼錯誤」）
- 登入失敗達到上限須鎖定帳號或啟用延遲

### 4.3 權限控制

- 所有資源存取必須驗證使用者權限（不可僅依賴前端隱藏按鈕）
- Spring Security：使用 `@PreAuthorize` 或 `@Secured` 進行方法層級權限控制
- FastAPI：使用 `Depends(require_auth)` 或 `Depends(require_role(...))`
- ❌ 禁止透過修改 URL 參數（如 ID）存取他人資料（IDOR 防護）

---

## 5. 敏感資料保護

### 5.1 機密管理

| 規則 | 說明 |
|------|------|
| **環境變數** | API 金鑰、資料庫密碼、JWT Secret 必須透過環境變數讀取 |
| **禁止硬編碼** | ❌ 禁止在程式碼、設定檔、註解中寫死任何密碼、Token、金鑰 |
| **env.example** | 提供範例設定檔（不含真實密碼），實際 `.env` 不進版控 |
| **預設值檢查** | 啟動時若關鍵環境變數未設定，必須拋出明確錯誤 |

### 5.2 Log 與 Response 保護

- ❌ Response 禁止包含完整的 Exception Stack Trace
- ❌ Log 禁止輸出密碼、身分證字號、信用卡號、Token 等敏感資訊
- 統一使用自訂錯誤頁面 / 統一錯誤回應格式
- 敏感欄位寫入 Log 前必須遮罩

### 5.3 前端禁止暴露

- ❌ JavaScript / HTML 註解中禁止包含：
  - API 金鑰、密碼、Token
  - 內部 IP 位址或伺服器資訊
  - 資料庫連線資訊

---

## 6. HTTP 安全標頭

確保 Response Header 包含以下安全設定：

```
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY 或 SAMEORIGIN
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Cookie 屬性

- `Secure`：僅 HTTPS 傳輸
- `HttpOnly`：禁止 JavaScript 存取
- `SameSite=Strict` 或 `Lax`：防止 CSRF

---

## 7. Fortify / 弱掃常見問題

| 問題類型 | 說明 | 解決方向 |
|---------|------|---------|
| **Path Traversal** | 使用者輸入直接用於檔案路徑 | 驗證並正規化路徑，使用白名單機制 |
| **Insecure Randomness** | 使用 `java.util.Random` / `Math.random()` 產生安全相關數值 | 改用 `java.security.SecureRandom` / Python `secrets` 模組 |
| **Privacy Violation** | 敏感資料寫入 Log | 過濾或遮罩敏感欄位 |
| **Log Forging** | 使用者輸入直接寫入 Log | 過濾換行字元與特殊字元 |
| **Hardcoded Password** | 密碼寫死在程式碼中 | 使用環境變數或加密設定檔 |
| **Null Dereference** | 未檢查 Null 即呼叫方法 | 加入 Null Check |
| **Resource Injection** | 使用者輸入用於資源識別 | 驗證輸入並使用白名單 |
| **Open Redirect** | 未驗證的重導向 URL | 僅允許白名單內的 URL |
| **Weak Cryptography** | 使用 MD5、SHA1 等弱雜湊 | 改用 SHA-256 以上，密碼使用 BCrypt |
| **Cookie Security** | Cookie 未設定安全屬性 | 設定 `Secure`、`HttpOnly`、`SameSite` |
| **Missing HSTS** | 未強制 HTTPS | 加入 Strict-Transport-Security Header |
| **Clickjacking** | 頁面可被嵌入 iframe | 設定 `X-Frame-Options: DENY` |
