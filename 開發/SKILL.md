---
name: 開發
description: Java 資深開發者協助。禁用 Lombok、Lambda 與 Stream API，必須手工撰寫所有 Getter/Setter，每行程式碼加繁體中文註解。前端採用 Thymeleaf 模板，嚴格遵守 XSS、CSRF、SQL Injection 等資安規範。所有開發必須先完成系統分析，使用者輸入 OKOKYES 後方可啟動開發。
---

# Skill: Java Senior Developer (Legacy-Strict Mode)

## Usage Trigger
- 當涉及 Java 檔案 (.java) 或 Spring Boot 專案架構時自動啟用。
- 當進行後端業務邏輯開發、POJO 建立或 API 修改時。
- 當涉及 HTML、CSS、JavaScript 或 Thymeleaf 模板開發時。
- 當進入測試，則請使用 測試 skill

---

## 1. 開發技術限制

### 1.1 後端 Java 規範
- **設定檔**：統一使用 `application.yml`，禁止使用 `application.properties`。
- **程式語法禁令 (Strict)**：
  - **禁用 Lombok**：嚴禁使用 `@Data`, `@Getter`, `@Setter` 等註解，POJO 必須手動生成 Getter/Setter 與 Constructor。
  - **禁用 Lambda**：禁止使用 Lambda 表達式或 Stream API，統一使用傳統 `for-loop`、`if-else` 與匿名內部類別。

### 1.2 前端技術規範
- **模板引擎**：強制使用 Thymeleaf，禁止使用 JSP。
- **JavaScript 框架**：若無特殊需求，優先使用原生 JavaScript 或 jQuery，禁止引入重量級前端框架（React、Vue、Angular）除非專案已採用。

### 1.3 資料庫異動規範
- **強制使用 Liquibase**：所有資料庫結構異動（DDL）必須透過 Liquibase changelog 管理。
- **禁止直接操作資料庫**：
  - ❌ 禁止直接對資料庫執行 CREATE、ALTER、DROP 等 DDL 語句
  - ❌ 禁止使用 DBeaver、pgAdmin、MySQL Workbench 等工具直接修改資料庫結構
  - ❌ 禁止在程式碼中使用 `spring.jpa.hibernate.ddl-auto=update/create`
- **Changelog 規範**：
  - 使用 XML 或 YAML 格式撰寫 changelog
  - 每個 changeset 必須有唯一的 `id` 與 `author`
  - changeset 一旦執行後禁止修改，新異動須建立新的 changeset
  - 必須包含 `rollback` 區塊以支援回滾
- **命名規範**：changelog 檔案以日期或版本號開頭（例：`20260103-add-user-table.xml`）

---

## 2. 註解規範
- **逐行註解**：每一行具備邏輯意義的程式碼都必須附上繁體中文註解。
- 註解內容應精確說明該行之邏輯目的。
- HTML 模板中的關鍵邏輯區塊亦須加上 `<!-- 註解 -->` 說明。

---

## 3. 代碼品質要求
- 必須包含完整的 Null Check 與異常處理機制。
- 確保程式碼符合資深 SD 的高可讀性與穩定性要求。
- 所有對外 API 必須有輸入驗證（Input Validation）。

### 3.1 Exception 處理與日誌規範（強制）

> ⚠️ **最高原則**：所有 Exception 必須記錄完整的堆疊追蹤（Stack Trace），禁止吞掉例外或僅記錄訊息。

**強制規範**：
| 規則 | 說明 |
|------|------|
| **必須使用 log.error** | 捕獲 Exception 時，必須呼叫 `log.error()` 記錄錯誤 |
| **必須包含堆疊追蹤** | `log.error()` 的第二個參數必須傳入 Exception 物件本身 |
| **禁止僅記錄訊息** | ❌ 禁止 `log.error(e.getMessage())` 這種遺失堆疊的寫法 |
| **禁止吞掉例外** | ❌ 禁止空的 catch 區塊或僅 `e.printStackTrace()` |

**正確範例**：
```java
try {
    // 業務邏輯
    userService.createUser(userDto);
} catch (Exception e) {
    // ✅ 正確：包含錯誤描述與完整堆疊追蹤
    log.error("建立使用者失敗，userId: {}", userDto.getUserId(), e);
    throw new BusinessException("使用者建立失敗", e);
}
```

**錯誤範例**：
```java
// ❌ 錯誤 1：僅記錄訊息，遺失堆疊追蹤
catch (Exception e) {
    log.error("發生錯誤: " + e.getMessage());
}

// ❌ 錯誤 2：空的 catch 區塊（吞掉例外）
catch (Exception e) {
    // do nothing
}

// ❌ 錯誤 3：僅使用 printStackTrace（不走日誌框架）
catch (Exception e) {
    e.printStackTrace();
}

// ❌ 錯誤 4：遺失堆疊追蹤的寫法
catch (Exception e) {
    log.error("錯誤: {}", e.getMessage());
}
```

**Logger 宣告規範**：
```java
// ✅ 正確：使用 SLF4J 宣告 Logger
private static final Logger log = LoggerFactory.getLogger(UserService.class);
```

---

## 4. 前端開發規範（Thymeleaf）

### 4.1 HTML 規範
- 使用語意化標籤（`<header>`, `<nav>`, `<main>`, `<footer>`, `<article>`, `<section>`）。
- 所有表單元素必須有對應的 `<label>`。
- 圖片必須包含 `alt` 屬性。
- 禁止使用 inline style（`style="..."`），樣式統一寫入 CSS 檔案。

### 4.2 CSS 規範
- 類別命名採用 kebab-case（例：`user-profile`, `btn-primary`）。
- 禁止使用 `!important`，除非覆蓋第三方套件樣式。
- 樣式檔案須依功能模組拆分，避免單一巨型 CSS 檔案。

### 4.3 JavaScript 規範
- **禁止 inline script**：禁止在 HTML 中直接撰寫 `<script>...</script>` 或 `onclick="..."`。
- 所有 JavaScript 必須寫入獨立 `.js` 檔案，透過 Thymeleaf 引入。
- 事件綁定統一使用 `addEventListener` 或 jQuery `.on()` 方法。
- 禁止使用 `eval()`、`new Function()` 等動態執行程式碼的方法。

### 4.4 Thymeleaf 專用規範
- **輸出文字**：強制使用 `th:text`（自動 HTML Escape），禁止使用 `th:utext` 除非經過資安審核。
- **URL 建構**：使用 `@{/path}` 語法，禁止手動拼接 URL 字串。
- **表單處理**：使用 `th:action` 與 `th:object` 綁定，禁止手動組裝表單。
- **條件與迴圈**：使用 `th:if`, `th:unless`, `th:each`，保持模板可讀性。

---

## 5. 資訊安全規範

### 5.1 XSS（Cross-Site Scripting）防護
| 情境 | 強制措施 |
|------|---------|
| 輸出使用者輸入 | 必須使用 `th:text`，絕對禁止 `th:utext` |
| JavaScript 變數賦值 | 使用 `th:inline="javascript"` 搭配 `[[${var}]]`（自動 escape） |
| URL 參數 | 使用 `@{/path(param=${var})}` 自動編碼 |
| HTML 屬性 | 使用 `th:attr` 或專屬屬性如 `th:href`, `th:src` |

### 5.2 CSRF（Cross-Site Request Forgery）防護
- 所有 POST、PUT、DELETE、PATCH 請求的表單必須包含 CSRF Token。
- Thymeleaf 表單使用 `th:action` 時會自動注入 CSRF Token（需確認 Spring Security 已啟用）。
- AJAX 請求必須在 Header 中帶入 CSRF Token：
  ```javascript
  // 從 meta 標籤取得 Token
  var token = $("meta[name='_csrf']").attr("content");
  var header = $("meta[name='_csrf_header']").attr("content");
  ```
- 頁面 `<head>` 區塊必須包含：
  ```html
  <meta name="_csrf" th:content="${_csrf.token}"/>
  <meta name="_csrf_header" th:content="${_csrf.headerName}"/>
  ```

### 5.3 SQL Injection 防護
- **絕對禁止**字串拼接 SQL 語句。
- 強制使用：
  - JPA Named Parameter：`@Query("SELECT u FROM User u WHERE u.name = :name")`
  - PreparedStatement 的 `?` 佔位符
  - MyBatis 的 `#{}` 參數綁定（禁止使用 `${}`）

### 5.4 敏感資料保護
- Response 禁止包含完整的 Exception Stack Trace，統一使用自訂錯誤頁面。
- 禁止在前端 JavaScript 或 HTML 註解中暴露：
  - API 金鑰、密碼、Token
  - 內部 IP 位址或伺服器資訊
  - 資料庫連線資訊
- Log 記錄禁止輸出密碼、身分證字號、信用卡號等敏感資訊。

### 5.5 權限控制
- 所有資源存取必須驗證使用者權限（不可僅依賴前端隱藏按鈕）。
- 使用 Spring Security 的 `@PreAuthorize` 或 `@Secured` 進行方法層級權限控制。
- 禁止透過修改 URL 參數（如 ID）存取他人資料（IDOR 防護）。

### 5.6 HTTP 安全標頭
確保 Response Header 包含以下安全設定：
```
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY 或 SAMEORIGIN
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

---

## 6. Fortify / 弱掃常見問題清單

| 問題類型 | 說明 | 解決方案 |
|---------|------|---------|
| **Path Traversal** | 使用者輸入直接用於檔案路徑 | 驗證並正規化路徑，使用白名單機制 |
| **Insecure Randomness** | 使用 `java.util.Random` 產生安全相關數值 | 改用 `java.security.SecureRandom` |
| **Privacy Violation** | 敏感資料寫入 Log | 過濾或遮罩敏感欄位 |
| **Log Forging** | 使用者輸入直接寫入 Log | 過濾換行字元與特殊字元 |
| **Hardcoded Password** | 密碼寫死在程式碼中 | 使用環境變數或加密設定檔 |
| **Null Dereference** | 未檢查 Null 即呼叫方法 | 加入 Null Check |
| **Resource Injection** | 使用者輸入用於資源識別 | 驗證輸入並使用白名單 |
| **Open Redirect** | 未驗證的重導向 URL | 僅允許白名單內的 URL |
| **Weak Cryptography** | 使用 MD5、SHA1 等弱雜湊 | 改用 SHA-256 以上，密碼使用 BCrypt |
| **Cookie Security** | Cookie 未設定安全屬性 | 設定 `Secure`, `HttpOnly`, `SameSite` |
| **Missing HSTS** | 未強制 HTTPS | 加入 Strict-Transport-Security Header |
| **Clickjacking** | 頁面可被嵌入 iframe | 設定 X-Frame-Options: DENY |

---

## 7. 開發流程強制規範

> ⚠️ **最高原則**：任何開發任務皆不得在未完成分析前開始撰寫程式碼。

### 7.1 分析階段要求
1. **需求評估**：評估需求合理性，識別潛在風險。
2. **系統分析報告**必須包含：
   - 受影響檔案清單
   - 邏輯變更點說明
   - 每個邏輯階段包含的檔案（作為 commit 依據）
   - **資安風險評估**：列出可能涉及的安全風險與對應防護措施
3. **等待使用者確認**。

### 7.2 啟動開發的唯一條件

```
┌─────────────────────────────────────────────────────────┐
│  僅當使用者輸入「OKOKYES」關鍵字時，方可啟動開發程序    │
│                                                         │
│  ❌ 禁止接受：「確認」「同意」「OK」「好」「可以」等    │
│  ❌ 禁止在分析未完成前開始撰寫任何程式碼                │
│  ❌ 禁止跳過資安風險評估                                │
└─────────────────────────────────────────────────────────┘
```

### 7.3 開發階段要求
- 依照分析報告的邏輯階段順序進行開發。
- 每完成一個邏輯階段，立即執行 `git commit`。
- 開發過程中發現新的資安風險，須立即補充說明並處理。

### 7.4 完成檢查清單
- [ ] 所有程式碼皆有繁體中文註解
- [ ] 無 Lombok、Lambda、Stream API 使用
- [ ] 通過 Null Check 與異常處理檢查
- [ ] XSS 防護：所有輸出使用 `th:text`
- [ ] CSRF 防護：所有表單包含 Token
- [ ] SQL Injection 防護：無字串拼接 SQL
- [ ] 無敏感資料外洩風險
- [ ] 專案可正常編譯
- [ ] 測試案例通過（若有提供）

---

## Skill 串接機制（強制執行）

> ⚠️ **當所有開發任務完成且專案編譯成功後，必須執行以下流程：**

### 強制執行步驟

1. **確認開發完成**：
   - 所有邏輯階段的程式碼已撰寫完成
   - 專案可正常編譯（`./mvnw compile` 成功）
   - 所有 commit 已完成

2. **明確告知找到下一個 Skill**：必須輸出以下格式的訊息

```
【Skill 串接通知】

✅ 開發階段已完成
✅ 專案編譯成功
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

此 Skill 將協助您進行：
- 程式碼品質審查
- 檢查禁用語法（Lombok/Lambda/Stream）
- 資安規範審查（XSS、CSRF、SQL Injection）
- 繁體中文註解完整度檢查

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

3. **等待使用者確認**：
   - ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview` skill
   - ❌ 使用者輸入其他內容 → 不啟動，繼續正常對話

### 禁止行為

- ❌ 禁止未告知就自動啟動下一個 Skill
- ❌ 禁止跳過「已找到 Skill」的通知
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動 Skill
- ❌ 禁止在編譯失敗時提示串接 CodeReview Skill
