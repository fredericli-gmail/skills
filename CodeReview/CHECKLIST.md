# CodeReview 完整審查檢查清單

> 本文件提供 CodeReview Skill 使用的完整審查項目與檢測規則

---

## 1. 後端 Java 審查清單

### 1.1 🔴 錯誤項目（必須修正）

#### Lombok 禁用檢查

| 註解 | 說明 | 正則表達式 |
|------|------|-----------|
| `@Data` | 自動產生 getter/setter/toString/equals/hashCode | `@Data\b` |
| `@Getter` | 自動產生 getter | `@Getter\b` |
| `@Setter` | 自動產生 setter | `@Setter\b` |
| `@Builder` | 建構者模式 | `@Builder\b` |
| `@AllArgsConstructor` | 全參數建構子 | `@AllArgsConstructor\b` |
| `@NoArgsConstructor` | 無參數建構子 | `@NoArgsConstructor\b` |
| `@RequiredArgsConstructor` | 必要參數建構子 | `@RequiredArgsConstructor\b` |
| `@ToString` | 自動產生 toString | `@ToString\b` |
| `@EqualsAndHashCode` | 自動產生 equals/hashCode | `@EqualsAndHashCode\b` |
| `@Value` | 不可變物件 | `@Value\b` |
| `@Slf4j` | 自動注入 Logger | `@Slf4j\b` |
| `@Log4j` | 自動注入 Logger | `@Log4j\b` |

**修正方式**：移除 Lombok 註解，手動撰寫對應方法

---

#### Lambda 與 Stream 禁用檢查

| 項目 | 說明 | 正則表達式 |
|------|------|-----------|
| Lambda 表達式 | `->` 運算子 | `[^-]->[^>]` |
| `.stream()` | Stream 起始 | `\.stream\s*\(` |
| `.parallelStream()` | 平行 Stream | `\.parallelStream\s*\(` |
| `.map()` | Stream 映射 | `\.map\s*\(` |
| `.filter()` | Stream 過濾 | `\.filter\s*\(` |
| `.collect()` | Stream 收集 | `\.collect\s*\(` |
| `.forEach()` | Stream 迭代 | `\.forEach\s*\(` |
| `.flatMap()` | Stream 攤平 | `\.flatMap\s*\(` |
| `.reduce()` | Stream 歸約 | `\.reduce\s*\(` |
| `.sorted()` | Stream 排序 | `\.sorted\s*\(` |
| `.distinct()` | Stream 去重 | `\.distinct\s*\(` |
| `.limit()` | Stream 限制 | `\.limit\s*\(` |
| `.skip()` | Stream 跳過 | `\.skip\s*\(` |
| `Optional.of()` | Optional 建立 | `Optional\.(of|ofNullable|empty)\s*\(` |
| `.orElse()` | Optional 預設值 | `\.orElse\s*\(` |
| `.ifPresent()` | Optional 條件執行 | `\.ifPresent\s*\(` |

**修正方式**：改用傳統 for-loop、if-else、匿名內部類別

---

#### SQL Injection 檢查

| 項目 | 說明 | 檢測模式 |
|------|------|---------|
| 字串拼接 SELECT | `"SELECT ... " + variable` | `"SELECT[^"]*"\s*\+` |
| 字串拼接 INSERT | `"INSERT ... " + variable` | `"INSERT[^"]*"\s*\+` |
| 字串拼接 UPDATE | `"UPDATE ... " + variable` | `"UPDATE[^"]*"\s*\+` |
| 字串拼接 DELETE | `"DELETE ... " + variable` | `"DELETE[^"]*"\s*\+` |
| 字串拼接 WHERE | `"WHERE ... " + variable` | `"WHERE[^"]*"\s*\+` |
| MyBatis ${} | 不安全的參數綁定 | `\$\{[^}]+\}` |

**修正方式**：
- JPA：使用 `@Query` 搭配 `:paramName`
- JDBC：使用 `PreparedStatement` 搭配 `?`
- MyBatis：使用 `#{paramName}` 取代 `${paramName}`

---

#### 硬編碼敏感資訊檢查

| 項目 | 正則表達式 |
|------|-----------|
| 密碼 | `(password|passwd|pwd)\s*=\s*"[^"]+"` |
| 金鑰 | `(key|secret|apikey|api_key)\s*=\s*"[^"]+"` |
| Token | `(token|access_token|auth_token)\s*=\s*"[^"]+"` |
| 憑證 | `(credential|cert)\s*=\s*"[^"]+"` |

**修正方式**：使用環境變數、設定檔或密鑰管理服務

---

#### 不安全亂數檢查

| 項目 | 說明 | 檢測模式 |
|------|------|---------|
| `java.util.Random` | 不安全的亂數產生器 | `new Random\(\)` |
| `Math.random()` | 不安全的亂數 | `Math\.random\(\)` |

**修正方式**：安全相關場景改用 `java.security.SecureRandom`

---

### 1.2 🟡 警告項目（建議修正）

#### Null Check 檢查

檢查以下情況是否有適當的 null 檢查：
- 方法參數使用前
- 方法回傳值使用前
- 集合元素使用前
- 外部 API 回傳值使用前

**檢測提示**：
```java
// 危險：未檢查 null
String name = user.getName();

// 安全：有 null 檢查
if (user != null) {
    String name = user.getName();
}
```

---

#### 異常處理檢查

| 項目 | 說明 | 正則表達式 |
|------|------|-----------|
| 空 catch 區塊 | catch 後無處理 | `catch\s*\([^)]+\)\s*\{\s*\}` |
| 僅印出堆疊 | 沒有適當處理 | `e\.printStackTrace\(\)` |
| 吞掉異常 | catch 後無動作 | `catch\s*\([^)]+\)\s*\{[^}]*\}` 內無 throw/log |
| log.error 遺失堆疊 | 僅記錄訊息未傳入 Exception | `log\.error\s*\(\s*e\.getMessage\(\)` |
| log.error 格式錯誤 | 未將 Exception 作為最後參數 | `log\.error\s*\([^,)]+\)\s*;` |

**修正方式**：
- 記錄 Log 並拋出適當的業務異常
- 或進行適當的錯誤恢復處理
- **log.error 必須包含完整堆疊追蹤**

**正確範例**：
```java
// ✅ 正確：Exception 物件作為最後參數，自動印出完整堆疊追蹤
catch (Exception e) {
    log.error("處理失敗，userId: {}", userId, e);
    throw new BusinessException("操作失敗", e);
}
```

**錯誤範例**：
```java
// ❌ 錯誤 1：僅記錄訊息，遺失堆疊追蹤
catch (Exception e) {
    log.error("發生錯誤: " + e.getMessage());
}

// ❌ 錯誤 2：使用 {} 格式化訊息但未傳入 Exception 物件
catch (Exception e) {
    log.error("錯誤: {}", e.getMessage());
}

// ❌ 錯誤 3：空的 catch 區塊
catch (Exception e) {
    // do nothing
}

// ❌ 錯誤 4：僅使用 printStackTrace
catch (Exception e) {
    e.printStackTrace();
}
```

---

#### 註解完整度檢查

每一行具備邏輯意義的程式碼都應有繁體中文註解：
- 變數宣告與初始化
- 條件判斷
- 迴圈
- 方法呼叫
- 回傳語句

**排除項目**：
- 空白行
- 僅有括號的行 `{`, `}`, `(`，`)`
- import 語句
- package 語句
- 註解本身

---

### 1.3 🟢 建議項目（可選改善）

| 項目 | 建議標準 |
|------|---------|
| 方法長度 | 單一方法不超過 50 行 |
| 類別長度 | 單一類別不超過 500 行 |
| 參數數量 | 方法參數不超過 5 個 |
| 巢狀深度 | if/for 巢狀不超過 3 層 |
| 圈複雜度 | 單一方法不超過 10 |

---

## 2. 前端審查清單

### 2.1 🔴 錯誤項目（必須修正）

#### XSS 風險檢查

| 項目 | 說明 | 正則表達式 |
|------|------|-----------|
| `th:utext` | 不安全的 HTML 輸出 | `th:utext` |
| 手動拼接 URL | 未使用 Thymeleaf URL 語法 | `href\s*=\s*"[^"]*\$\{` |
| `eval()` | 動態執行程式碼 | `eval\s*\(` |
| `new Function()` | 動態建立函式 | `new\s+Function\s*\(` |
| `innerHTML` | 不安全的 DOM 操作 | `\.innerHTML\s*=` |
| `document.write` | 不安全的輸出 | `document\.write\s*\(` |

---

#### CSRF 風險檢查

| 項目 | 說明 | 檢測方式 |
|------|------|---------|
| POST 表單缺少 Token | 未使用 `th:action` | `<form[^>]*method\s*=\s*["']post["'][^>]*>` 且無 `th:action` |
| AJAX 缺少 Header | POST 請求未帶 CSRF | 檢查 `$.ajax`, `$.post`, `fetch` 的 POST 請求 |

**修正方式**：
- 表單：使用 `th:action` 自動注入 Token
- AJAX：在 Header 加入 `X-CSRF-TOKEN`

---

### 2.2 🟡 警告項目（建議修正）

| 項目 | 說明 | 正則表達式 |
|------|------|-----------|
| Inline Style | HTML 內嵌樣式 | `style\s*=\s*"` |
| Inline Script | HTML 內嵌事件 | `on(click|change|submit|load|error)\s*=\s*"` |
| 缺少 Label | 表單元素無標籤 | `<input` 無對應 `<label>` |
| 缺少 Alt | 圖片無替代文字 | `<img(?![^>]*alt\s*=)` |
| `!important` | CSS 強制覆蓋 | `!important` |

---

### 2.3 🟢 建議項目（可選改善）

| 項目 | 建議 |
|------|------|
| 語意化標籤 | 使用 `<header>`, `<nav>`, `<main>`, `<footer>`, `<article>`, `<section>` |
| CSS 命名 | 使用 kebab-case（如 `user-profile`） |
| JS 檔案分離 | JavaScript 寫入獨立 `.js` 檔案 |
| CSS 檔案分離 | 樣式寫入獨立 `.css` 檔案 |

---

## 3. Fortify / 弱掃常見問題

| 問題類型 | 檢測重點 | 修正方向 |
|---------|---------|---------|
| Path Traversal | 檔案路徑包含使用者輸入 | 正規化路徑、白名單驗證 |
| Insecure Randomness | `Random` 用於安全場景 | 改用 `SecureRandom` |
| Privacy Violation | 敏感資料寫入 Log | 遮罩處理 |
| Log Forging | 使用者輸入直接寫 Log | 過濾換行字元 |
| Null Dereference | 未檢查 Null | 加入 Null Check |
| Resource Injection | 使用者輸入用於資源 | 白名單驗證 |
| Open Redirect | 未驗證重導向 URL | 白名單驗證 |
| Weak Cryptography | MD5/SHA1 | 改用 SHA-256+ |
| Cookie Security | 缺少安全屬性 | 設定 Secure/HttpOnly/SameSite |

---

## 4. 審查執行指令參考

```bash
# 搜尋 Lombok 註解
grep -rn "@Data\|@Getter\|@Setter\|@Builder" --include="*.java" src/

# 搜尋 Lambda 表達式
grep -rn "\->" --include="*.java" src/

# 搜尋 Stream API
grep -rn "\.stream()\|\.map(\|\.filter(\|\.collect(" --include="*.java" src/

# 搜尋 SQL 字串拼接
grep -rn '"SELECT\|"INSERT\|"UPDATE\|"DELETE' --include="*.java" src/ | grep "+"

# 搜尋 th:utext
grep -rn "th:utext" --include="*.html" src/

# 搜尋 eval
grep -rn "eval\s*(" --include="*.js" src/

# 搜尋 inline style
grep -rn 'style\s*=' --include="*.html" src/
```
