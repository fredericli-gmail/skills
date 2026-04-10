---
name: 開發-Thymeleaf前端
description: "Thymeleaf 前端開發者協助。Spring Boot + Thymeleaf 模板，禁止使用 JSP，優先使用原生 JavaScript 或 jQuery。每行程式碼加繁體中文註解。嚴格遵守 XSS、CSRF 等資安規範。所有開發必須先完成系統分析，使用者輸入 OKOKYES 後方可啟動開發。當涉及 templates/*.html、Spring Boot SSR 模板時觸發。"
---

# Skill: Thymeleaf 前端開發

> ⚠️ **本 SKILL 引用共享規範**：
> - 編碼規範：[_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md)
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)

## Usage Trigger

- 當涉及 `src/main/resources/templates/*.html`、Thymeleaf 模板時自動啟用
- 當涉及前端頁面新增、修改時（且專案使用 Thymeleaf 而非 React）
- 後端 Java 開發請使用 `/開發-Java後端`
- 當進入測試，請使用 `/測試` Skill

---

## 1. 前端技術規範

### 1.1 技術棧

| 項目 | 選用 |
|------|------|
| **模板引擎** | Thymeleaf（**禁止 JSP**） |
| **JavaScript** | 原生 JavaScript 或 jQuery（**禁止重量級框架** 如 React、Vue、Angular，除非專案已採用） |
| **CSS** | 模組化 CSS 檔案 |
| **後端** | Spring Boot + Spring Security |

---

## 2. HTML 規範

- 使用語意化標籤（`<header>`、`<nav>`、`<main>`、`<footer>`、`<article>`、`<section>`）
- 所有表單元素必須有對應的 `<label>`
- 圖片必須包含 `alt` 屬性
- **禁止 inline style**（`style="..."`），樣式統一寫入 CSS 檔案

---

## 3. CSS 規範

- 類別命名採用 **kebab-case**（例：`user-profile`、`btn-primary`）
- **禁止 `!important`**，除非覆蓋第三方套件樣式
- 樣式檔案須依功能模組拆分，避免單一巨型 CSS 檔案

---

## 4. JavaScript 規範

- **禁止 inline script**：禁止在 HTML 中直接撰寫 `<script>...</script>` 或 `onclick="..."`
- 所有 JavaScript 必須寫入獨立 `.js` 檔案，透過 Thymeleaf 引入
- 事件綁定統一使用 `addEventListener` 或 jQuery `.on()` 方法
- 禁止使用 `eval()`、`new Function()` 等動態執行程式碼的方法

---

## 5. Thymeleaf 專用規範

| 情境 | 規範 |
|------|------|
| **輸出文字** | 強制使用 `th:text`（自動 HTML Escape），**禁止 `th:utext`** 除非經過資安審核 |
| **URL 建構** | 使用 `@{/path}` 語法，禁止手動拼接 URL 字串 |
| **表單處理** | 使用 `th:action` 與 `th:object` 綁定，禁止手動組裝表單 |
| **條件與迴圈** | 使用 `th:if`、`th:unless`、`th:each`，保持模板可讀性 |
| **片段重用** | 使用 `th:fragment` 與 `th:replace` 重用共用片段 |

### 5.1 範例

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <!-- CSRF Meta（資安規範要求） -->
    <meta name="_csrf" th:content="${_csrf.token}"/>
    <meta name="_csrf_header" th:content="${_csrf.headerName}"/>
    <title>使用者管理</title>
    <!-- 載入外部 CSS -->
    <link rel="stylesheet" th:href="@{/css/users.css}"/>
</head>
<body>
    <!-- 頁面標題 -->
    <h1 th:text="${pageTitle}">使用者管理</h1>

    <!-- 使用者列表（迴圈） -->
    <table>
        <thead>
            <tr>
                <th>帳號</th>
                <th>姓名</th>
            </tr>
        </thead>
        <tbody>
            <!-- 逐筆迭代使用者 -->
            <tr th:each="user : ${users}">
                <td th:text="${user.username}">admin</td>
                <td th:text="${user.name}">管理員</td>
            </tr>
        </tbody>
    </table>

    <!-- 表單（自動注入 CSRF Token） -->
    <form th:action="@{/users}" th:object="${userForm}" method="post">
        <label for="username">帳號</label>
        <input type="text" id="username" th:field="*{username}"/>

        <label for="name">姓名</label>
        <input type="text" id="name" th:field="*{name}"/>

        <button type="submit">建立</button>
    </form>

    <!-- 載入外部 JS -->
    <script th:src="@{/js/users.js}"></script>
</body>
</html>
```

---

## 6. 資安規範

> 詳見 [共享規範 SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)。

### 6.1 XSS 防護

| 情境 | 強制措施 |
|------|---------|
| 輸出使用者輸入 | 必須使用 `th:text`，**絕對禁止 `th:utext`** |
| JavaScript 變數賦值 | 使用 `th:inline="javascript"` 搭配 `[[${var}]]`（自動 escape） |
| URL 參數 | 使用 `@{/path(param=${var})}` 自動編碼 |
| HTML 屬性 | 使用 `th:attr` 或專屬屬性如 `th:href`、`th:src` |

### 6.2 CSRF 防護

- 所有 POST、PUT、DELETE、PATCH 請求的表單必須包含 CSRF Token
- Thymeleaf 表單使用 `th:action` 時會自動注入 CSRF Token（需確認 Spring Security 已啟用）
- AJAX 請求必須在 Header 中帶入 CSRF Token：

```javascript
// 從 meta 標籤取得 Token
var token = $("meta[name='_csrf']").attr("content");
var header = $("meta[name='_csrf_header']").attr("content");

// jQuery AJAX 全域設定
$.ajaxSetup({
    beforeSend: function(xhr) {
        xhr.setRequestHeader(header, token);
    }
});
```

頁面 `<head>` 區塊必須包含：
```html
<meta name="_csrf" th:content="${_csrf.token}"/>
<meta name="_csrf_header" th:content="${_csrf.headerName}"/>
```

---

## 7. 註解規範

詳見 [共享規範 §2](../_shared/CODING-STANDARDS.md#2-註解規範)。

- HTML 模板中的關鍵邏輯區塊必須加上 `<!-- 註解 -->` 說明
- JavaScript 檔案中每行邏輯程式碼必須附上繁體中文 `// 註解`

---

## 8. 開發流程強制規範

### 8.1 啟動開發的唯一條件

```
┌─────────────────────────────────────────────────────────┐
│  僅當使用者輸入「OKOKYES」關鍵字時，方可啟動開發程序    │
│  ❌ 禁止接受其他確認詞彙                                │
└─────────────────────────────────────────────────────────┘
```

### 8.2 完成檢查清單

- [ ] 所有程式碼皆有繁體中文註解
- [ ] 所有輸出使用 `th:text`（無 `th:utext`）
- [ ] 所有表單包含 CSRF Token
- [ ] 無 inline style、無 inline script
- [ ] 表單元素有對應 `<label>`
- [ ] 圖片有 `alt` 屬性
- [ ] 專案可正常編譯（`./mvnw compile`）

---

## 9. Skill 串接機制（強制執行）

開發完成且編譯成功後，必須輸出：

```
【Skill 串接通知】

✅ 開發階段已完成
✅ 專案編譯成功
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

此 Skill 將協助您進行：
- HTML / Thymeleaf 程式碼品質審查
- XSS、CSRF 防護檢查
- 繁體中文註解完整度檢查

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview`
- ❌ 其他輸入 → 不啟動
