---
name: 開發
description: Java 資深開發者協助。後端禁用 Lombok、Lambda 與 Stream API，必須手工撰寫所有 Getter/Setter，每行程式碼加繁體中文註解。前端採用 Vue 3 + Vite + Tailwind CSS 架構，使用 Composition API（script setup）開發 SFC 元件。嚴格遵守 XSS、CSRF、SQL Injection 等資安規範。所有開發必須先完成系統分析，使用者輸入 OKOKYES 後方可啟動開發。
---

# Skill: Full-Stack Developer (Java Backend + Vue Frontend)

## Usage Trigger
- 當涉及 Java 檔案 (.java) 或 Spring Boot 專案架構時自動啟用。
- 當進行後端業務邏輯開發、POJO 建立或 API 修改時。
- 當涉及 Vue 元件 (.vue)、JavaScript (.js)、CSS 或前端開發時。
- 當涉及前端頁面新增、修改或遷移時。
- 當進入測試，則請使用 測試 skill

---

## 1. 開發技術限制

### 1.1 後端 Java 規範
- **設定檔**：統一使用 `application.yml`，禁止使用 `application.properties`。
- **程式語法禁令 (Strict)**：
  - **禁用 Lombok**：嚴禁使用 `@Data`, `@Getter`, `@Setter` 等註解，POJO 必須手動生成 Getter/Setter 與 Constructor。
  - **禁用 Lambda**：禁止使用 Lambda 表達式或 Stream API，統一使用傳統 `for-loop`、`if-else` 與匿名內部類別。

### 1.2 前端技術規範
- **前端框架**：強制使用 Vue 3 + Vite + Tailwind CSS 架構。
- **元件格式**：使用 SFC（Single File Component，`.vue` 檔案）。
- **API 風格**：強制使用 Composition API（`<script setup>`），禁止使用 Options API。
- **JavaScript 語法**：Vue 前端允許使用箭頭函式 `=>`、`const/let`、解構賦值、Template Literal，禁止使用 `var`。
- **詳細規範**：參閱本文件「第 9 章 Vue 前端開發規範」。

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

## 4. 前端開發規範（Vue 3）

### 4.1 HTML / Template 規範
- 使用語意化標籤（`<header>`, `<nav>`, `<main>`, `<footer>`, `<article>`, `<section>`）。
- 所有表單元素必須有對應的 `<label>`。
- 圖片必須包含 `alt` 屬性。
- 禁止使用 inline style（`style="..."`），樣式統一使用 Tailwind CSS class 或 `<style scoped>`。
- Vue template 中使用 `{{ }}` 插值（自動 HTML Escape），禁止使用 `v-html`。

### 4.2 CSS 規範
- **優先使用 Tailwind CSS class**，減少自訂 CSS。
- 元件專屬樣式使用 `<style scoped>`，避免污染全域。
- 全域共用樣式放在 `src/assets/css/main.css`。
- 禁止使用 `!important`，除非覆蓋第三方套件樣式。

### 4.3 JavaScript 規範（Vue 前端）
- **允許箭頭函式**（Vue 生態標準寫法）。
- **允許解構賦值**、Template Literal、展開運算子。
- **禁止使用 `var`**，統一使用 `const`（優先）與 `let`。
- 禁止使用 `eval()`、`new Function()` 等動態執行程式碼的方法。
- 事件處理使用 Vue 的 `@click`、`@input` 等指令，禁止使用 `addEventListener`。

### 4.4 Vue 專用規範
- **元件格式**：強制使用 SFC（`.vue` 檔案），禁止在 JS 中用 `template` 字串定義元件。
- **API 風格**：強制使用 `<script setup>`（Composition API），禁止使用 Options API（`data/methods/computed`）。
- **詳細 Vue 規範**：參閱本文件「第 9 章 Vue 前端開發規範」。

---

## 5. 資訊安全規範

### 5.1 XSS（Cross-Site Scripting）防護
| 情境 | 強制措施 |
|------|---------|
| 輸出使用者輸入 | 使用 Vue `{{ }}` 插值（自動 HTML Escape），絕對禁止 `v-html` |
| 動態屬性綁定 | 使用 `:href`、`:src` 等 Vue 綁定，禁止手動拼接 URL |
| URL 參數 | 使用 Vue Router 的 `router.push({ query: {} })` 或 `encodeURIComponent()` |
| HTML 屬性 | 使用 Vue 動態綁定 `:attr`，禁止字串拼接 DOM 屬性 |

### 5.2 CSRF（Cross-Site Request Forgery）防護
- 本專案前端採用 JWT Token 認證（透過 `Authorization: Bearer` 標頭），非 Cookie-based Session，因此天然免疫 CSRF 攻擊。
- 所有 API 請求透過 axios 攔截器自動帶入 JWT Token，無需額外處理 CSRF Token。
- **禁止將 JWT Token 存入 Cookie**，統一存放於 `localStorage`。

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
- 後端：使用 `@RequirePermission` 註解進行方法層級權限控制。
- 前端：使用 `v-permission` 自訂指令控制元素顯示/隱藏，Vue Router 使用 `beforeEach` guard 檢查頁面權限。
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

## 7. Controller / Service 架構規範（強制）

> ⚠️ **最高原則**：每一個前端網頁功能，對應到後端必須有專用的 RestController，絕對禁止跨 Controller 使用，否則會造成權限控管混亂。

### 7.1 架構設計原則

| 規則 | 說明 |
|------|------|
| **專用 Controller** | 每個前端頁面/功能模組對應一個專用的 RestController |
| **專用 Service** | Controller 專用的方法，放在同名的 Service 中（如 `UserController` → `UserService`） |
| **禁止跨 Controller** | ❌ 禁止一個網頁功能呼叫其他 Controller 的 API |
| **權限獨立** | 每個 Controller 有獨立的權限控制，不與其他功能混用 |

### 7.2 命名規範

```
前端頁面              Controller                  Service
─────────────────────────────────────────────────────────────
使用者管理            UserController              UserService
AI 知識庫管理         AiPromptController          AiPromptService / AiDatasetTextService
AI 程式知識庫         AiSourceCodeController      AiSourceCodeService（專用）
AI PDF 知識庫         AiPdfController             AiPdfService（專用）
案件管理              CaseController              CaseService
```

### 7.3 開發時的架構檢查

在開發新功能時，必須確認：

1. **功能是否有專用 Controller**
   - ✅ 有 → 在該 Controller 中新增/修改 API
   - ❌ 沒有 → 建立新的專用 Controller

2. **方法應放在哪個 Service**
   - 若方法**只被此 Controller 使用** → 放在專用 Service（與 Controller 同名）
   - 若方法**被多個 Controller 共用** → 放在通用 Service（如 `AiKnowledgeBaseService`）

3. **檢查是否有跨 Controller 呼叫**
   - ❌ 禁止 Controller A 注入 Controller B
   - ❌ 禁止 Controller A 使用 Controller B 專用的 Service

### 7.4 正確 vs 錯誤範例

**✅ 正確架構**：
```java
// AiSourceCodeController 使用自己的專用 Service
@RestController
@RequestMapping("/api/admin/ai-source-codes")
public class AiSourceCodeController {

    private final AiSourceCodeService sourceCodeService;  // 專用 Service
    private final AiKnowledgeBaseService knowledgeBaseService;  // 共用 Service（OK）

    @PostMapping("/{id}/refresh-indexing-status")
    public ResponseEntity<?> refreshIndexingStatus(@PathVariable Long id) {
        // 呼叫專用 Service 的方法
        return sourceCodeService.refreshIndexingStatus(id);
    }
}
```

**❌ 錯誤架構**：
```java
// 錯誤：AiSourceCodeController 使用其他頁面的 Service
@RestController
@RequestMapping("/api/admin/ai-source-codes")
public class AiSourceCodeController {

    // ❌ 錯誤：注入其他頁面專用的 Service
    private final AiDatasetTextService textService;  // 這是 AiPromptController 專用的！

    @PostMapping("/{id}/refresh-indexing-status")
    public ResponseEntity<?> refreshIndexingStatus(@PathVariable Long id) {
        // ❌ 錯誤：呼叫其他頁面專用 Service 的方法
        return textService.refreshIndexingStatus(id);
    }
}
```

### 7.5 共用 Service vs 專用 Service 判斷標準

| Service 類型 | 判斷標準 | 範例 |
|-------------|---------|------|
| **共用 Service** | 被 2+ 個 Controller 使用、提供基礎功能 | `AiKnowledgeBaseService`、`UserService`、`NotificationService` |
| **專用 Service** | 只被 1 個 Controller 使用、提供頁面特定功能 | `AiSourceCodeService`、`AiPromptService`、`AiPdfService` |

### 7.6 發現架構問題時的處理方式

若發現現有程式碼違反此規範：

1. **標記為架構問題**：在分析報告中標記 🔴 架構違規
2. **建立專用 Service**：將方法移動或複製到專用 Service
3. **更新 Controller**：改為注入專用 Service
4. **測試驗證**：確保功能正常、權限控制正確

---

## 8. 開發流程強制規範

> ⚠️ **最高原則**：任何開發任務皆不得在未完成分析前開始撰寫程式碼。

### 8.1 分析階段要求
1. **需求評估**：評估需求合理性，識別潛在風險。
2. **系統分析報告**必須包含：
   - 受影響檔案清單
   - 邏輯變更點說明
   - 每個邏輯階段包含的檔案（作為 commit 依據）
   - **資安風險評估**：列出可能涉及的安全風險與對應防護措施
3. **等待使用者確認**。

### 8.2 啟動開發的唯一條件

```
┌─────────────────────────────────────────────────────────┐
│  僅當使用者輸入「OKOKYES」關鍵字時，方可啟動開發程序    │
│                                                         │
│  ❌ 禁止接受：「確認」「同意」「OK」「好」「可以」等    │
│  ❌ 禁止在分析未完成前開始撰寫任何程式碼                │
│  ❌ 禁止跳過資安風險評估                                │
└─────────────────────────────────────────────────────────┘
```

### 8.3 開發階段要求
- 依照分析報告的邏輯階段順序進行開發。
- 每完成一個邏輯階段，立即執行 `git commit`。
- 開發過程中發現新的資安風險，須立即補充說明並處理。

### 8.4 完成檢查清單

**後端 Java**：
- [ ] 所有程式碼皆有繁體中文註解
- [ ] 無 Lombok、Lambda、Stream API 使用
- [ ] 通過 Null Check 與異常處理檢查
- [ ] SQL Injection 防護：無字串拼接 SQL
- [ ] 無敏感資料外洩風險
- [ ] 專案可正常編譯（`./mvnw compile`）

**前端 Vue**：
- [ ] 所有程式碼皆有繁體中文註解
- [ ] 使用 `<script setup>`（Composition API），無 Options API
- [ ] XSS 防護：使用 `{{ }}` 插值，無 `v-html`
- [ ] 元件命名符合 PascalCase 規範
- [ ] Props 皆有型別定義
- [ ] 前端可正常建置（`npm run build`）
- [ ] 測試案例通過（若有提供）

---

## 9. Vue 前端開發規範

> **核心原則**：前端全面採用 Vue 3 + Vite + Tailwind CSS 架構，Spring Boot 僅負責提供 REST API。

### 9.1 技術選型

| 項目 | 選型 | 版本 |
|------|------|------|
| 前端框架 | Vue 3 | 3.x（Composition API） |
| 建置工具 | Vite | 6.x |
| 路由 | Vue Router | 4.x |
| HTTP 客戶端 | axios | 1.x |
| CSS 框架 | Tailwind CSS | 4.x（Vite 整合，非 CDN） |
| 圖表 | Chart.js | 既有版本 |
| 狀態管理 | 不使用 Pinia | 各頁面獨立，composable 足夠 |

### 9.2 專案目錄結構

```
frontend/                           # Vue 前端專案根目錄
├── index.html                      # SPA 入口 HTML
├── vite.config.js                  # Vite 設定（含 API proxy）
├── package.json                    # 前端依賴
├── tailwind.config.js              # Tailwind CSS 設定
├── postcss.config.js               # PostCSS 設定
├── src/
│   ├── main.js                     # Vue App 入口
│   ├── App.vue                     # 根元件（含 <router-view>）
│   │
│   ├── router/                     # 路由設定
│   │   └── index.js                # Vue Router（含 route guard 權限檢查）
│   │
│   ├── api/                        # API 層（模組化）
│   │   ├── index.js                # axios 實例 + JWT 攔截器
│   │   ├── auth.js                 # 認證 API（登入、登出、刷新 Token）
│   │   ├── users.js                # 使用者管理 API
│   │   └── [模組名].js             # 各功能模組 API
│   │
│   ├── composables/                # 組合式函式（共用邏輯）
│   │   ├── useAuth.js              # 認證狀態 + Token 管理
│   │   ├── usePermission.js        # 權限檢查
│   │   ├── useToast.js             # Toast 通知
│   │   ├── usePagination.js        # 分頁邏輯
│   │   ├── useSort.js              # 排序邏輯
│   │   ├── useFilter.js            # 篩選邏輯
│   │   └── useDarkMode.js          # 深色模式切換
│   │
│   ├── components/                 # 共用元件
│   │   ├── layout/                 # 版型元件
│   │   │   ├── AppLayout.vue       # 主版型（header + sidebar + router-view）
│   │   │   ├── AppHeader.vue       # 頂部導航列
│   │   │   ├── AppSidebar.vue      # 側邊選單（權限驅動）
│   │   │   └── AppToast.vue        # Toast 通知容器
│   │   ├── common/                 # 通用 UI 元件
│   │   │   ├── DataTable.vue       # 資料表格（排序、選取）
│   │   │   ├── AppPagination.vue   # 分頁元件
│   │   │   ├── FilterPanel.vue     # 篩選面板（可折疊）
│   │   │   ├── AppModal.vue        # Modal 對話框
│   │   │   ├── ActionDropdown.vue  # 操作下拉選單
│   │   │   ├── StatusBadge.vue     # 狀態標籤
│   │   │   └── ConfirmDialog.vue   # 確認對話框
│   │   └── form/                   # 表單元件
│   │       ├── FormField.vue       # 表單欄位（label + input + error）
│   │       ├── FormSelect.vue      # 下拉選擇
│   │       └── FormCheckbox.vue    # 核取方塊
│   │
│   ├── directives/                 # 自訂指令
│   │   └── permission.js           # v-permission 權限指令
│   │
│   ├── views/                      # 頁面元件（對應路由）
│   │   ├── auth/                   # 認證頁面（無 AppLayout）
│   │   │   ├── LoginView.vue
│   │   │   ├── ForgotPasswordView.vue
│   │   │   ├── ResetPasswordView.vue
│   │   │   └── ChangePasswordView.vue
│   │   ├── DashboardView.vue       # 營運總覽儀表板
│   │   ├── PhoneSupportView.vue    # AI 智慧查詢
│   │   ├── admin/                  # 管理頁面
│   │   │   ├── UsersView.vue
│   │   │   ├── RolesView.vue
│   │   │   ├── IpWhitelistView.vue
│   │   │   └── TextToSqlTrainView.vue
│   │   └── ai/                     # AI 功能頁面
│   │       ├── AiPromptsView.vue
│   │       ├── AiKnowledgeBasesView.vue
│   │       ├── AiSourceCodesView.vue
│   │       ├── AiPdfView.vue
│   │       ├── AiDatabasesView.vue
│   │       └── AiLangfuseLogsView.vue
│   │
│   └── assets/                     # 靜態資源
│       ├── css/
│       │   └── main.css            # 全域樣式（自訂按鈕、表格等）
│       └── images/                 # 圖片資源
```

### 9.3 開發與建置流程

#### 9.3.1 開發模式

```
┌──────────────────┐     /api/* proxy     ┌──────────────────┐
│ Vite Dev Server  │ ──────────────────→  │ Spring Boot      │
│ localhost:5173   │                      │ localhost:8080   │
│ (HMR + Vue)     │ ←──────────────────  │ (REST API)       │
└──────────────────┘     JSON 回應        └──────────────────┘
```

**Vite 設定（vite.config.js）**：
```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      }
    }
  },
  build: {
    outDir: '../src/main/resources/static',
    emptyOutDir: true
  }
})
```

#### 9.3.2 建置指令

| 指令 | 用途 |
|------|------|
| `cd frontend && npm install` | 安裝前端依賴 |
| `cd frontend && npm run dev` | 啟動開發伺服器（HMR） |
| `cd frontend && npm run build` | 建置正式版（輸出至 `static/`） |
| `cd frontend && npm run preview` | 預覽正式版建置結果 |

#### 9.3.3 Spring Boot 端配置

**PageController 簡化**：僅保留 SPA 入口路由，所有前端路由由 Vue Router 處理。

```java
// 所有非 API 路由轉發至 index.html（Vue Router 接手）
@Controller
public class SpaForwardController {

    // SPA 入口：所有非 API、非靜態資源的請求轉發至 index.html
    @GetMapping(value = {"/", "/{path:^(?!api|static|assets).*$}/**"})
    public String forward() {
        // 轉發至 Vue SPA 入口頁面
        return "forward:/index.html";
    }
}
```

### 9.4 SFC 元件結構規範

#### 9.4.1 區塊順序（強制）

```vue
<!-- 1. Template 區塊（HTML 結構） -->
<template>
  <!-- 模板內容 -->
</template>

<!-- 2. Script 區塊（邏輯） -->
<script setup>
// JavaScript 邏輯
</script>

<!-- 3. Style 區塊（樣式） -->
<style scoped>
/* 元件專屬樣式 */
</style>
```

#### 9.4.2 `<script setup>` 內部結構順序（強制）

```vue
<script setup>
// ==================== 1. 匯入區 ====================
// Vue 核心
import { ref, reactive, computed, onMounted, watch } from 'vue'
import { useRouter, useRoute } from 'vue-router'

// API 模組
import { userApi } from '@/api/users'

// Composables
import { useToast } from '@/composables/useToast'
import { usePermission } from '@/composables/usePermission'

// 元件
import DataTable from '@/components/common/DataTable.vue'
import AppModal from '@/components/common/AppModal.vue'

// ==================== 2. Props / Emits 定義 ====================
// 定義元件屬性
const props = defineProps({
  // 使用者 ID
  userId: {
    type: Number,
    required: true
  },
  // 是否為編輯模式
  isEdit: {
    type: Boolean,
    default: false
  }
})

// 定義事件
const emit = defineEmits(['save', 'cancel'])

// ==================== 3. Composables 呼叫 ====================
// 初始化路由
const router = useRouter()
// 初始化當前路由
const route = useRoute()
// 初始化 Toast 通知
const { showToast } = useToast()
// 初始化權限檢查
const { hasPermission } = usePermission()

// ==================== 4. 響應式狀態 ====================
// 載入中狀態
const loading = ref(false)
// 使用者列表
const users = ref([])
// 表單資料
const formData = reactive({
  username: '',
  displayName: '',
  email: ''
})

// ==================== 5. 計算屬性 ====================
// 是否可提交表單
const canSubmit = computed(() => {
  return formData.username && formData.displayName
})

// ==================== 6. 方法 ====================
/**
 * 載入使用者列表
 */
const loadUsers = async () => {
  // 設定載入狀態
  loading.value = true
  try {
    // 呼叫 API 取得使用者列表
    const response = await userApi.getUsers()
    // 更新使用者列表
    users.value = response.data
  } catch (error) {
    // 顯示錯誤通知
    showToast('載入使用者列表失敗', 'error')
  } finally {
    // 結束載入狀態
    loading.value = false
  }
}

/**
 * 儲存使用者資料
 */
const handleSave = async () => {
  // 呼叫 API 儲存
  await userApi.saveUser(formData)
  // 顯示成功通知
  showToast('儲存成功', 'success')
  // 觸發儲存事件
  emit('save')
}

// ==================== 7. Watch 監聽 ====================
// 監聽路由參數變化
watch(() => route.params.id, (newId) => {
  // 路由參數變更時重新載入
  if (newId) {
    loadUsers()
  }
})

// ==================== 8. 生命週期 ====================
// 元件掛載時載入資料
onMounted(() => {
  loadUsers()
})
</script>
```

### 9.5 命名規範

| 項目 | 規範 | 範例 |
|------|------|------|
| **元件檔名** | PascalCase | `UserList.vue`、`AppModal.vue` |
| **頁面元件** | PascalCase + `View` 後綴 | `UsersView.vue`、`DashboardView.vue` |
| **Composable** | camelCase + `use` 前綴 | `useAuth.js`、`useToast.js` |
| **API 模組** | kebab-case | `users.js`、`ai-knowledge-bases.js` |
| **Props** | camelCase | `userId`、`isEdit` |
| **Emits** | kebab-case | `@update-user`、`@close-modal` |
| **CSS class** | Tailwind class 優先，自訂用 kebab-case | `user-profile`、`btn-primary` |
| **路由 name** | PascalCase | `UserManagement`、`AiPrompts` |
| **路由 path** | kebab-case | `/users`、`/ai-knowledge-bases` |

### 9.6 API 層規範

#### 9.6.1 axios 實例（`api/index.js`）

```javascript
import axios from 'axios'

// 建立 axios 實例
const api = axios.create({
  // API 基礎路徑
  baseURL: '/api',
  // 請求逾時時間
  timeout: 30000,
  // 預設標頭
  headers: {
    'Content-Type': 'application/json'
  }
})

// ==================== 請求攔截器 ====================
api.interceptors.request.use((config) => {
  // 從 localStorage 取得 Token
  const token = localStorage.getItem('accessToken')
  // 若有 Token 則加入 Authorization 標頭
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  // 產生 W3C Trace Context 追蹤標頭
  config.headers['traceparent'] = generateTraceparent()
  return config
})

// ==================== 回應攔截器 ====================
api.interceptors.response.use(
  // 成功回應直接回傳
  (response) => response,
  // 錯誤回應處理
  async (error) => {
    const originalRequest = error.config
    // 401 且非重試請求：嘗試刷新 Token
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true
      try {
        // 呼叫刷新 Token API
        const refreshToken = localStorage.getItem('refreshToken')
        const { data } = await axios.post('/api/auth/refresh', { refreshToken })
        // 儲存新 Token
        localStorage.setItem('accessToken', data.accessToken)
        localStorage.setItem('refreshToken', data.refreshToken)
        // 使用新 Token 重試原始請求
        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`
        return api(originalRequest)
      } catch (refreshError) {
        // 刷新失敗：清除 Token 並導向登入頁
        localStorage.removeItem('accessToken')
        localStorage.removeItem('refreshToken')
        window.location.href = '/login'
        return Promise.reject(refreshError)
      }
    }
    return Promise.reject(error)
  }
)

export default api
```

#### 9.6.2 功能模組 API（範例：`api/users.js`）

```javascript
import api from './index'

// 使用者管理 API
export const userApi = {
  /**
   * 取得使用者列表
   * @param {Object} params - 查詢參數（page、size、keyword）
   */
  getUsers: (params) => api.get('/users', { params }),

  /**
   * 取得單一使用者
   * @param {Number} id - 使用者 ID
   */
  getUser: (id) => api.get(`/users/${id}`),

  /**
   * 建立使用者
   * @param {Object} data - 使用者資料
   */
  createUser: (data) => api.post('/users', data),

  /**
   * 更新使用者
   * @param {Number} id - 使用者 ID
   * @param {Object} data - 更新資料
   */
  updateUser: (id, data) => api.put(`/users/${id}`, data),

  /**
   * 刪除使用者
   * @param {Number} id - 使用者 ID
   */
  deleteUser: (id) => api.delete(`/users/${id}`)
}
```

### 9.7 Composable 規範

#### 9.7.1 設計原則

| 原則 | 說明 |
|------|------|
| **單一職責** | 每個 composable 只處理一個關注點 |
| **命名前綴** | 必須以 `use` 開頭 |
| **回傳物件** | 統一回傳具名物件（便於解構） |
| **響應式** | 內部狀態使用 `ref` 或 `reactive` |
| **可組合** | composable 可以呼叫其他 composable |

#### 9.7.2 範例：`useToast.js`

```javascript
import { ref } from 'vue'

// 全域 Toast 狀態（跨元件共享）
const toastVisible = ref(false)
const toastMessage = ref('')
const toastType = ref('info')

// Toast 計時器 ID
let toastTimer = null

/**
 * Toast 通知組合式函式
 */
export const useToast = () => {
  /**
   * 顯示 Toast 通知
   * @param {String} message - 通知訊息
   * @param {String} type - 通知類型（success / error / warning / info）
   * @param {Number} duration - 顯示時間（毫秒，預設 3000）
   */
  const showToast = (message, type = 'info', duration = 3000) => {
    // 設定通知內容
    toastMessage.value = message
    // 設定通知類型
    toastType.value = type
    // 顯示通知
    toastVisible.value = true
    // 清除既有計時器
    if (toastTimer) {
      clearTimeout(toastTimer)
    }
    // 設定自動隱藏計時器
    toastTimer = setTimeout(() => {
      toastVisible.value = false
    }, duration)
  }

  return {
    toastVisible,
    toastMessage,
    toastType,
    showToast
  }
}
```

#### 9.7.3 範例：`usePermission.js`

```javascript
import { computed } from 'vue'
import { useAuth } from './useAuth'

/**
 * 權限檢查組合式函式
 */
export const usePermission = () => {
  // 取得當前使用者資訊
  const { currentUser } = useAuth()

  /**
   * 檢查使用者是否擁有指定權限
   * @param {String} permissionCode - 權限代碼
   * @returns {Boolean} 是否有權限
   */
  const hasPermission = (permissionCode) => {
    // 若使用者未載入則無權限
    if (!currentUser.value || !currentUser.value.permissions) {
      return false
    }
    // 檢查權限清單中是否包含指定權限
    return currentUser.value.permissions.includes(permissionCode)
  }

  /**
   * 檢查使用者是否擁有任一指定權限
   * @param {Array} permissionCodes - 權限代碼陣列
   * @returns {Boolean} 是否有任一權限
   */
  const hasAnyPermission = (permissionCodes) => {
    return permissionCodes.some((code) => hasPermission(code))
  }

  return {
    hasPermission,
    hasAnyPermission
  }
}
```

### 9.8 Vue Router 與權限控制

#### 9.8.1 路由設定（`router/index.js`）

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import AppLayout from '@/components/layout/AppLayout.vue'

// 路由設定
const routes = [
  // ==================== 認證頁面（無 Layout） ====================
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/auth/LoginView.vue'),
    meta: { requiresAuth: false }
  },
  // ==================== 主應用頁面（含 Layout） ====================
  {
    path: '/',
    component: AppLayout,
    meta: { requiresAuth: true },
    children: [
      {
        path: '',
        name: 'Dashboard',
        component: () => import('@/views/DashboardView.vue'),
        meta: { permission: 'DASHBOARD_V3_HEALTH_VIEW' }
      },
      {
        path: 'users',
        name: 'UserManagement',
        component: () => import('@/views/admin/UsersView.vue'),
        meta: { permission: 'USER_VIEW' }
      }
      // ... 其他路由
    ]
  }
]

// 建立路由實例
const router = createRouter({
  history: createWebHistory(),
  routes
})

// ==================== 路由守衛（權限檢查） ====================
router.beforeEach((to, from, next) => {
  // 取得 Token
  const token = localStorage.getItem('accessToken')
  // 需要認證但無 Token：導向登入頁
  if (to.meta.requiresAuth !== false && !token) {
    next({ name: 'Login' })
    return
  }
  // 不需認證（登入頁等）：直接放行
  if (to.meta.requiresAuth === false) {
    next()
    return
  }
  // 有權限要求：檢查使用者權限
  // （權限資料由 useAuth composable 管理）
  next()
})

export default router
```

#### 9.8.2 自訂權限指令（`directives/permission.js`）

```javascript
import { usePermission } from '@/composables/usePermission'

/**
 * v-permission 自訂指令
 * 用法：<button v-permission="'USER_CREATE'">新增</button>
 * 無權限時自動移除元素
 */
export const vPermission = {
  mounted(el, binding) {
    const { hasPermission } = usePermission()
    // 檢查權限
    if (!hasPermission(binding.value)) {
      // 無權限：移除元素
      el.parentNode && el.parentNode.removeChild(el)
    }
  }
}
```

**在 template 中使用**：
```vue
<template>
  <!-- 僅有 USER_CREATE 權限時顯示新增按鈕 -->
  <button v-permission="'USER_CREATE'" class="btn-primary">
    新增使用者
  </button>
</template>
```

### 9.9 共用元件使用規範

#### 9.9.1 DataTable 元件

```vue
<template>
  <!-- 資料表格 -->
  <DataTable
    :columns="columns"
    :data="users"
    :loading="loading"
    :sortable="true"
    @sort-change="handleSortChange"
  >
    <!-- 自訂欄位插槽 -->
    <template #cell-status="{ row }">
      <StatusBadge :status="row.enabled ? 'active' : 'inactive'" />
    </template>
    <!-- 操作欄插槽 -->
    <template #cell-actions="{ row }">
      <button v-permission="'USER_EDIT'" @click="editUser(row)">
        編輯
      </button>
    </template>
  </DataTable>
</template>
```

#### 9.9.2 AppModal 元件

```vue
<template>
  <!-- Modal 對話框 -->
  <AppModal
    :visible="showModal"
    :title="isEdit ? '編輯使用者' : '新增使用者'"
    @close="showModal = false"
    @confirm="handleSave"
  >
    <!-- Modal 內容 -->
    <FormField label="帳號" v-model="formData.username" required />
    <FormField label="顯示名稱" v-model="formData.displayName" required />
    <FormSelect label="角色" v-model="formData.role" :options="roleOptions" />
  </AppModal>
</template>
```

### 9.10 資安規範（Vue 版）

| 攻擊類型 | 防護措施 |
|---------|---------|
| **XSS** | Vue `{{ }}` 自動 escape；禁止使用 `v-html`；禁止使用 `innerHTML` |
| **CSRF** | JWT Token 透過 `Authorization` 標頭傳送（非 Cookie），天然免疫 |
| **敏感資料** | 禁止在前端程式碼中硬編碼 API 金鑰、密碼；使用 `.env` 環境變數 |
| **Token 安全** | Token 存放 `localStorage`；頁面關閉不自動清除（使用者手動登出） |
| **路由安全** | 所有需認證路由設定 `meta.requiresAuth: true` + route guard 檢查 |
| **輸入驗證** | 前端驗證為使用者體驗；後端 `@Valid` 為安全保障，兩者皆須實作 |

### 9.11 註解規範（Vue 前端）

與後端 Java 一致，**每行具備邏輯意義的程式碼都必須附上繁體中文註解**。

**Template 區塊**：
```vue
<template>
  <!-- 使用者管理頁面主容器 -->
  <div class="p-6">
    <!-- 頁面標題列 -->
    <div class="flex justify-between items-center mb-6">
      <!-- 標題文字 -->
      <h1 class="text-2xl font-bold">使用者管理</h1>
      <!-- 新增按鈕（需 USER_CREATE 權限） -->
      <button v-permission="'USER_CREATE'" @click="showCreateModal">
        新增使用者
      </button>
    </div>
  </div>
</template>
```

**Script 區塊**：
```vue
<script setup>
// 匯入 Vue 響應式 API
import { ref, onMounted } from 'vue'
// 匯入使用者 API 模組
import { userApi } from '@/api/users'

// 使用者列表資料
const users = ref([])
// 載入中狀態
const loading = ref(false)

/**
 * 載入使用者列表
 */
const loadUsers = async () => {
  // 設定載入狀態為進行中
  loading.value = true
  try {
    // 呼叫後端 API 取得使用者列表
    const response = await userApi.getUsers()
    // 將回應資料存入響應式變數
    users.value = response.data
  } catch (error) {
    // 載入失敗時記錄錯誤
    console.error('[使用者管理] 載入失敗', error)
  } finally {
    // 無論成功失敗都結束載入狀態
    loading.value = false
  }
}

// 元件掛載時載入使用者列表
onMounted(() => {
  loadUsers()
})
</script>
```

### 9.12 環境變數管理

| 檔案 | 用途 | 範例 |
|------|------|------|
| `.env` | 所有環境共用 | `VITE_APP_TITLE=Danova-Mind` |
| `.env.development` | 開發環境 | `VITE_API_BASE_URL=http://localhost:8080` |
| `.env.production` | 正式環境 | `VITE_API_BASE_URL=` |

**使用方式**：
```javascript
// 取得環境變數（必須以 VITE_ 前綴）
const appTitle = import.meta.env.VITE_APP_TITLE
```

**禁止事項**：
- ❌ 禁止在 `.env` 中存放密碼、API 金鑰等敏感資訊
- ❌ 禁止將 `.env.local` 提交至版本控制

### 9.13 禁止事項總表

| 禁止項目 | 說明 |
|---------|------|
| ❌ Options API | 禁止使用 `data()`、`methods`、`computed` 等 Options API 語法 |
| ❌ `v-html` | 禁止使用，防止 XSS 攻擊 |
| ❌ `var` 宣告 | 統一使用 `const`（優先）與 `let` |
| ❌ jQuery | 禁止引入 jQuery，使用 Vue 響應式系統 |
| ❌ `document.getElementById` | 禁止直接操作 DOM，使用 `ref` 模板引用 |
| ❌ `addEventListener` | 禁止手動綁定事件，使用 Vue `@event` 指令 |
| ❌ `eval()` / `new Function()` | 禁止動態執行程式碼 |
| ❌ Pinia / Vuex | 本專案不使用全域狀態管理，composable 足夠 |
| ❌ Mixin | 禁止使用 Mixin，改用 composable |
| ❌ `this` | `<script setup>` 中無 `this`，直接使用變數 |

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
