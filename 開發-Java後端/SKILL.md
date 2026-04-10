---
name: 開發-Java後端
description: "Java 資深後端開發者協助。Spring Boot 架構，禁用 Lombok、Lambda 與 Stream API，必須手工撰寫所有 Getter/Setter，每行程式碼加繁體中文註解。嚴格遵守 XSS、CSRF、SQL Injection 等資安規範。所有開發必須先完成系統分析，使用者輸入 OKOKYES 後方可啟動開發。當涉及 .java 檔案、Spring Boot 專案、Maven pom.xml 時觸發。"
---

# Skill: Java 後端開發（Legacy-Strict Mode）

> ⚠️ **本 SKILL 引用共享規範**：
> - 編碼規範：[_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md)
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)

## Usage Trigger

- 當涉及 Java 檔案 (.java) 或 Spring Boot 專案架構時自動啟用
- 當進行後端業務邏輯開發、POJO 建立或 API 修改時
- 不負責前端：前端請使用 `/開發-React前端` 或 `/開發-Thymeleaf前端`
- 當進入測試，請使用 `/測試` Skill

---

## 1. Java 後端規範

### 1.1 設定檔

- 統一使用 `application.yml`，禁止使用 `application.properties`
- 不同環境使用 profile：`application-dev.yml`、`application-prod.yml`

### 1.2 程式語法禁令（Strict）

| 禁用項目 | 說明 | 替代方案 |
|---------|------|---------|
| **Lombok** | 嚴禁使用 `@Data`、`@Getter`、`@Setter`、`@Builder`、`@Slf4j` 等註解 | POJO 必須手動生成 Getter/Setter 與 Constructor；Logger 手動宣告 |
| **Lambda 表達式** | 禁止使用 `->` 箭頭函式 | 使用傳統 `for-loop`、`if-else` 與匿名內部類別 |
| **Stream API** | 禁止 `.stream()`、`.map()`、`.filter()`、`.collect()`、`.forEach()` 等 | 使用傳統 `for` 迴圈 |
| **Optional 鏈式呼叫** | 避免 `Optional.of(...).map(...).orElse(...)` | 使用 `if-else` 判斷 |

### 1.3 Logger 標準寫法

```java
// ✅ 正確：手動宣告 SLF4J Logger
private static final Logger log = LoggerFactory.getLogger(UserService.class);

// ❌ 禁止：使用 @Slf4j
```

### 1.4 資料庫異動

詳見 [共享規範 §4](../_shared/CODING-STANDARDS.md#4-資料庫異動規範)。

- **強制使用 Liquibase** 管理 changelog
- 使用 XML 或 YAML 格式撰寫 changelog
- 每個 changeset 必須有唯一 `id` 與 `author`
- 已執行的 changeset 禁止修改
- 必須包含 `rollback` 區塊

---

## 2. Controller / Service 架構規範

> ⚠️ **強制遵守**：詳見 [共享規範 §1](../_shared/CODING-STANDARDS.md#1-controller--service-架構規範)。

### 2.1 命名範例

```
前端頁面              Controller                  Service
─────────────────────────────────────────────────────────────
使用者管理            UserController              UserService
案件管理              CaseController              CaseService
報表中心              ReportController            ReportService
```

### 2.2 正確 vs 錯誤範例

**✅ 正確架構**：
```java
@RestController
@RequestMapping("/api/admin/cases")
public class CaseController {

    private final CaseService caseService;            // 專用 Service
    private final NotificationService notifyService;  // 共用 Service（OK）

    @PostMapping("/{id}/close")
    public ResponseEntity<?> closeCase(@PathVariable Long id) {
        // 呼叫專用 Service 的方法
        return caseService.closeCase(id);
    }
}
```

**❌ 錯誤架構**：
```java
@RestController
@RequestMapping("/api/admin/cases")
public class CaseController {

    // ❌ 錯誤：注入其他頁面專用的 Service
    private final ReportService reportService;

    @PostMapping("/{id}/close")
    public ResponseEntity<?> closeCase(@PathVariable Long id) {
        // ❌ 錯誤：呼叫其他頁面專用 Service 的方法
        return reportService.processCase(id);
    }
}
```

---

## 3. 註解規範

詳見 [共享規範 §2](../_shared/CODING-STANDARDS.md#2-註解規範)。

- **逐行註解**：每一行具備邏輯意義的程式碼都必須附上繁體中文註解
- **Javadoc**：所有 public 方法須有 Javadoc 說明用途、參數、回傳值、例外

```java
/**
 * 建立新使用者
 *
 * @param userDto 使用者資料物件
 * @return 已建立的使用者實體
 * @throws BusinessException 當帳號已存在時拋出
 */
public User createUser(UserDto userDto) {
    // 檢查帳號是否已存在
    if (userRepository.existsByUsername(userDto.getUsername())) {
        // 拋出業務例外
        throw new BusinessException("帳號已存在");
    }
    // 建立新的使用者實體
    User user = new User();
    // 設定使用者名稱
    user.setUsername(userDto.getUsername());
    // 儲存到資料庫並回傳結果
    return userRepository.save(user);
}
```

---

## 4. Exception 處理規範

詳見 [共享規範 §3](../_shared/CODING-STANDARDS.md#3-exception-處理與日誌規範)。

```java
// ✅ 正確：包含錯誤描述與完整堆疊追蹤
try {
    userService.createUser(userDto);
} catch (Exception e) {
    log.error("建立使用者失敗，userId: {}", userDto.getUserId(), e);
    throw new BusinessException("使用者建立失敗", e);
}
```

---

## 5. 資安規範

> 詳見 [共享規範 SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)。

### 5.1 Java / Spring 重點

| 類別 | 規範 |
|------|------|
| **SQL Injection** | JPA Named Parameter `:param` 或 `@Query`；禁止字串拼接 SQL |
| **CSRF** | Spring Security 啟用 CSRF；表單使用 Thymeleaf `th:action` 自動注入 Token |
| **權限控制** | 使用 `@PreAuthorize`、`@Secured` 進行方法層級控制 |
| **密碼** | 使用 BCrypt 雜湊儲存；禁止明文 |
| **機密管理** | API Key、密碼、JWT Secret 透過環境變數讀取，禁止硬編碼 |
| **錯誤訊息** | Response 禁止包含完整 Stack Trace |

---

## 6. 代碼品質要求

詳見 [共享規範 §6](../_shared/CODING-STANDARDS.md#6-程式碼品質要求)。

- 必須包含完整的 Null Check 與異常處理機制
- 所有對外 API 必須有輸入驗證（`@Valid` + Bean Validation）
- 方法不超過 50 行、類別不超過 500 行、參數不超過 5 個

---

## 7. 開發流程強制規範

> ⚠️ **最高原則**：任何開發任務皆不得在未完成分析前開始撰寫程式碼。

### 7.1 啟動開發的唯一條件

```
┌─────────────────────────────────────────────────────────┐
│  僅當使用者輸入「OKOKYES」關鍵字時，方可啟動開發程序    │
│                                                         │
│  ❌ 禁止接受：「確認」「同意」「OK」「好」「可以」等    │
│  ❌ 禁止在分析未完成前開始撰寫任何程式碼                │
│  ❌ 禁止跳過資安風險評估                                │
└─────────────────────────────────────────────────────────┘
```

### 7.2 開發階段要求

- 依照分析報告的邏輯階段順序進行開發
- 每完成一個邏輯階段，立即執行 `git commit`（一個邏輯階段 = 一個 commit）
- 開發過程中發現新的資安風險，須立即補充說明並處理

### 7.3 完成檢查清單

- [ ] 所有程式碼皆有繁體中文註解
- [ ] 無 Lombok、Lambda、Stream API 使用
- [ ] 通過 Null Check 與異常處理檢查
- [ ] Exception 處理符合共享規範（含完整堆疊追蹤）
- [ ] 無字串拼接 SQL（Injection 防護）
- [ ] 無硬編碼密碼或 Secret
- [ ] 架構符合 Controller/Service 規範（無跨 Controller 呼叫）
- [ ] 專案可正常編譯（`./mvnw compile`）
- [ ] 測試案例通過（若有提供）

---

## 8. Skill 串接機制（強制執行）

> ⚠️ **當所有開發任務完成且專案編譯成功後，必須執行以下流程：**

### 強制執行步驟

1. **確認開發完成**：所有邏輯階段完成、`./mvnw compile` 成功、所有 commit 已完成
2. **輸出 Skill 串接通知**：

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
   - ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview`
   - ❌ 其他輸入 → 不啟動

### 禁止行為

- ❌ 禁止未告知就自動啟動下一個 Skill
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動 Skill
- ❌ 禁止在編譯失敗時提示串接 CodeReview Skill
