---
name: 原始碼掃描
description: "原始碼資安掃描（SAST）專家，依 OWASP Top 10 2025 RC 檢查 Java、Python、React、Thymeleaf 專案。自動偵測語言選擇工具（Semgrep、Bandit、SpotBugs、ESLint Security、syft/grype），產出含嚴重度、OWASP 分類、檔案行號、修補建議的表格報告。涵蓋 Broken Access Control、Security Misconfiguration、Software Supply Chain Failures、Injection、Mishandling of Exceptional Conditions 等十大類。當用戶說「/原始碼掃描」、「/SAST」、「/OWASP掃描」、「掃描原始碼」、「資安靜態分析」時觸發。亦可於 CodeReview 通過後自動串接觸發。確認後使用者必須輸入 OKOKYES 才會執行掃描。"
---

# Skill: 原始碼掃描（SAST / OWASP Top 10 2025）

> 📌 **依據版本**：OWASP Top 10 **2025 Release Candidate**（2025-11-06 發布於 Global AppSec Conference）
> 官方連結：https://owasp.org/Top10/2025/

> ⚠️ **本 SKILL 引用共享規範**：
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)
> - 本 SKILL 屬於三層資安檢測的第一層（SAST），第二層為「網頁弱點掃描」（DAST），第三層為「滲透測試」。

---

## Usage Trigger

- 使用者輸入 `/原始碼掃描`、`/SAST`、`/OWASP掃描`
- 使用者說「掃描原始碼」、「資安靜態分析」、「檢查 OWASP 漏洞」
- `CodeReview` Skill 審查通過後自動串接觸發
- 重大資安事件發生後的全面複檢

---

## 1. 掃描前置作業

### 1.1 確認掃描範圍

1. **讀取使用者指定路徑**，預設為當前專案根目錄
2. **自動偵測技術棧**：

   | 判斷檔案 | 技術棧 |
   |---------|--------|
   | `pom.xml` / `build.gradle` + `.java` | Java（Spring Boot） |
   | `requirements.txt` / `pyproject.toml` + `.py` | Python（FastAPI） |
   | `package.json` + `.jsx` | React |
   | `templates/*.html` + Thymeleaf 語法 | Thymeleaf |

3. **輸出掃描範圍確認**：
   ```
   【原始碼掃描範圍確認】

   專案路徑：/path/to/project
   偵測到技術棧：Java + Thymeleaf
   掃描檔案數（估算）：128 個

   將使用工具：
   - Semgrep（OWASP 規則集）
   - SpotBugs + Find Security Bugs
   - 自訂 Thymeleaf XSS 規則

   是否啟動掃描？請輸入「OKOKYES」確認。
   ```

### 1.2 工具可用性檢查與自動安裝

掃描前檢查工具是否存在，若缺少則協助安裝：

| 工具 | 檢查指令 | 安裝指令 |
|------|---------|---------|
| **Semgrep** | `semgrep --version` | `pip install semgrep` |
| **Bandit** | `bandit --version` | `pip install bandit` |
| **SpotBugs** | `which spotbugs` | `brew install spotbugs`（macOS） |
| **Find Security Bugs** | 下載 plugin jar | Maven plugin 或 CLI plugin |
| **ESLint Security** | `npx eslint --version` | `npm i -D eslint eslint-plugin-security` |
| **syft**（SBOM） | `syft version` | `brew install syft` |
| **grype**（SBOM 掃描） | `grype version` | `brew install grype` |
| **cosign**（簽章驗證） | `cosign version` | `brew install cosign` |

> ⚠️ **安裝前必須告知使用者並等待 `OKOKYES`**。禁止靜默安裝第三方工具。

---

## 2. OWASP Top 10 2025 對照檢查清單

所有語言共用此邏輯清單，工具覆蓋度與特定寫法依語言調整。

> **2025 版重大變化**：
> - 🆕 A03 Software Supply Chain Failures（新增，擴大 2021 A06 範圍）
> - 🆕 A10 Mishandling of Exceptional Conditions（新增，取代 2021 A10 SSRF）
> - ♻️ SSRF 併入 A01 Broken Access Control
> - ⬆️ Security Misconfiguration 從 #5 升至 #2

---

### A01:2025 – Broken Access Control（存取控制失效，含 SSRF）

| 檢查項目 | Java | Python | React/JS | Thymeleaf |
|---------|------|--------|----------|-----------|
| 缺少 `@PreAuthorize` / 權限檢查 | ✅ | ✅ | - | - |
| 路由未驗證登入狀態 | ✅ | ✅ | ✅ | ✅ |
| IDOR（直接物件引用）未校驗擁有者 | ✅ | ✅ | - | - |
| 前端越權路由（僅藏不擋） | - | - | ✅ | ✅ |
| **SSRF**：`HttpClient` / `requests.get` 使用使用者可控 URL | ✅ | ✅ | - | - |
| **SSRF**：未限制目標為內網 IP / metadata endpoint（`169.254.169.254`） | ✅ | ✅ | - | - |
| CORS 設定為 `*` 且允許 credentials | ✅ | ✅ | - | - |
| JWT 未驗證 `aud`、`iss`、`exp` | ✅ | ✅ | - | - |

### A02:2025 – Security Misconfiguration（設定錯誤，2025 升至第 2）

- Debug 模式開啟（Spring `debug=true`、FastAPI `debug=True`、React dev build）
- 預設帳號密碼未改（`admin/admin`、`root/root`）
- 錯誤訊息暴露 stack trace 給使用者
- `.env`、`application.properties` 含明文密鑰且被 commit
- HTTP Security Headers 缺失（CSP、HSTS、X-Frame-Options）
- Spring Actuator 未加保護暴露 `/actuator/env`、`/actuator/heapdump`
- 雲端資源設定錯誤（S3 bucket public、IAM 權限過大）
- 未使用的功能 / 端點仍啟用

### A03:2025 – Software Supply Chain Failures（供應鏈失效，新類別）

**比 2021 A06「Vulnerable Components」範圍更廣**，涵蓋整個軟體供應鏈。

| 檢查項目 | 工具 |
|---------|------|
| 依賴套件含已知 CVE | `mvn dependency-check`、`pip-audit`、`npm audit`、`grype` |
| **SBOM 未產生或未簽章** | `syft`（產 SBOM）、`cosign`（簽章） |
| **Typosquatting**（套件拼錯名攻擊） | 人工 + Semgrep 規則 |
| **CI/CD 管線不安全** | `.github/workflows/` 使用未 pin 版本的 action |
| Lock file 缺失或過期 | `package-lock.json`、`poetry.lock` |
| 套件來源未限制（允許任意 registry） | `.npmrc`、`pip.conf` |
| 缺少 SLSA 供應鏈層級要求 | SLSA 驗證工具 |
| 內部 mirror 被污染 | Nexus / Artifactory 設定檢查 |

**重點指令**：
```bash
# 產出 SBOM
syft packages dir:. -o cyclonedx-json > sbom.json

# 依 SBOM 檢查漏洞
grype sbom:sbom.json -o json > grype-report.json

# 簽章驗證
cosign verify <image>
```

### A04:2025 – Cryptographic Failures（加密失誤，原 2021 A02）

- 硬編碼密鑰、Token、密碼（`password = "xxx"`、`API_KEY = "xxx"`）
- 使用弱雜湊（MD5、SHA1）於密碼場景
- 密碼使用非 KDF 雜湊（應用 bcrypt、argon2、PBKDF2）
- 使用 `new Random()`（Java）、`random` 模組（Python）於安全場景，應改 `SecureRandom` / `secrets`
- HTTPS 驗證被繞過（`verify=False`、`TrustAllCerts`）
- 使用已破解演算法（DES、RC4、SHA1-with-RSA）
- IV / Nonce 重複使用

### A05:2025 – Injection（注入，含 XSS，原 2021 A03）

| 類型 | 檢查重點 |
|------|---------|
| **SQL Injection** | 字串拼接 SQL、f-string / `%` 拼接、`@Query` 用 `${}` 而非 `:param` |
| **Command Injection** | `Runtime.exec`、`os.system`、`subprocess` 使用使用者輸入 |
| **LDAP Injection** | 拼接 LDAP filter |
| **XSS** | `th:utext`、`dangerouslySetInnerHTML`、`innerHTML = userInput` |
| **Template Injection** | Thymeleaf、Jinja2 動態模板字串 |
| **NoSQL Injection** | MongoDB `$where`、未驗證 JSON 結構 |
| **XPath / Expression Language Injection** | SpEL、OGNL 拼接 |

### A06:2025 – Insecure Design（不安全設計，原 2021 A04）

- 缺少速率限制（登入、OTP、API）
- 密碼重設流程缺少身分確認
- 敏感動作缺少二次驗證
- 業務邏輯漏洞（例如：優惠券可無限使用）
- 缺少威脅模型與安全設計審查

### A07:2025 – Authentication Failures（認證失誤，原 2021 A07）

- JWT 驗證使用 `none` algorithm
- Session Fixation（登入後未重新產生 Session ID）
- 密碼策略過弱（無長度、無複雜度要求）
- 記住我 Token 明文儲存
- MFA 缺失或可繞過
- 帳號列舉（登入失敗訊息區分「帳號不存在」與「密碼錯誤」）

### A08:2025 – Software or Data Integrity Failures（資料完整性失誤，原 2021 A08）

- 不安全的反序列化（`ObjectInputStream`、`pickle.loads`、`yaml.load`）
- JavaScript 未驗證 Subresource Integrity（SRI）
- 自動更新機制未驗證簽章
- CI/CD artifact 未簽章
- 資料庫備份未校驗完整性

### A09:2025 – Security Logging and Alerting Failures（日誌與告警失誤，2025 加「Alerting」）

**2025 版強調「告警」而不只是記錄**：

- 敏感操作（登入、權限變更、資料刪除）未記錄
- 日誌記錄密碼、Token、信用卡等敏感資料
- `log.error` 未帶 `Exception` / `exc_info=True`
- **異常行為無即時告警**（例如：短時間多次登入失敗、權限越權）
- 日誌未集中存放（無 SIEM）
- 日誌保留期過短，無法事後鑑識

### A10:2025 – Mishandling of Exceptional Conditions（例外處理不當，新類別）

**2025 新增**，專注於錯誤處理失控導致的資安問題。

| 檢查項目 | Java | Python | React/JS |
|---------|------|--------|----------|
| 空 catch 吞例外 `catch (Exception e) {}` | ✅ | ✅ | ✅ |
| `printStackTrace()` 暴露於回應 | ✅ | ✅ | - |
| `bare except:` 未指定例外類型 | - | ✅ | - |
| 錯誤訊息洩漏系統路徑 / SQL 語句 / stack trace | ✅ | ✅ | ✅ |
| 例外處理中權限繞過（fail-open 而非 fail-secure） | ✅ | ✅ | ✅ |
| 交易回滾失敗未處理 | ✅ | ✅ | - |
| `try-catch-ignore` 掩蓋資安檢查失敗 | ✅ | ✅ | - |
| Promise rejection 未處理 | - | - | ✅ |
| Global error handler 回傳完整錯誤物件給前端 | ✅ | ✅ | ✅ |

**重點 Semgrep 規則**：
```yaml
rules:
  - id: java-empty-catch
    pattern: |
      try { ... } catch ($EX $E) { }
    message: "空 catch 區塊可能掩蓋資安問題，違反 A10:2025"
    severity: ERROR
  - id: python-bare-except
    pattern: |
      try:
          ...
      except:
          ...
    message: "bare except 可能掩蓋資安例外"
    severity: ERROR
```

---

## 3. 工具執行流程（依語言）

### 3.1 Java + Spring Boot

```bash
# 1. Semgrep OWASP 規則
semgrep --config=p/owasp-top-ten --config=p/java --sarif --output=sast-java.sarif .

# 2. SpotBugs + Find Security Bugs（需先編譯）
mvn compile
spotbugs -textui -pluginList findsecbugs-plugin.jar \
         -effort:max -low -xml:withMessages \
         -output spotbugs-report.xml target/classes

# 3. A03 Supply Chain：OWASP Dependency Check
mvn org.owasp:dependency-check-maven:check

# 4. A03 Supply Chain：SBOM 產生 + 漏洞比對
syft packages dir:. -o cyclonedx-json > sbom.json
grype sbom:sbom.json -o json > grype-java.json
```

**Thymeleaf 補充（自訂 Semgrep 規則）**：
- 偵測 `th:utext="${...}"` 用法
- 偵測手動拼接 `@{${...}}` URL
- 偵測缺少 `th:action` 的 POST form

### 3.2 Python + FastAPI

```bash
# 1. Bandit 掃描
bandit -r app/ -f json -o bandit-report.json

# 2. Semgrep OWASP 規則
semgrep --config=p/owasp-top-ten --config=p/python --sarif --output=sast-python.sarif .

# 3. A03 Supply Chain：pip-audit 檢查已知 CVE
pip-audit -r requirements.txt -f json -o pip-audit.json

# 4. A03 Supply Chain：SBOM + grype
syft packages dir:. -o cyclonedx-json > sbom.json
grype sbom:sbom.json -o json > grype-python.json
```

### 3.3 React（JSX，非 TypeScript）

```bash
# 1. ESLint + security plugin
npx eslint --ext .jsx,.js \
    --plugin security \
    --config .eslintrc.security.json \
    -f json -o eslint-security.json src/

# 2. Semgrep React 規則
semgrep --config=p/react --config=p/javascript --sarif --output=sast-react.sarif .

# 3. A03 Supply Chain：npm audit
npm audit --json > npm-audit.json

# 4. A03 Supply Chain：SBOM + grype
syft packages dir:. -o cyclonedx-json > sbom.json
grype sbom:sbom.json -o json > grype-react.json
```

**重點檢查項目**：
- `dangerouslySetInnerHTML`
- `eval` / `new Function`
- `innerHTML = userInput`
- `window.location = userInput`（Open Redirect）
- localStorage 儲存 JWT / 密碼

### 3.4 Thymeleaf

```bash
# 使用 Semgrep 自訂規則
semgrep --config=./semgrep-rules/thymeleaf-xss.yml \
        --config=./semgrep-rules/thymeleaf-csrf.yml \
        src/main/resources/templates/
```

**自訂規則範例**（`thymeleaf-xss.yml`）：
```yaml
rules:
  - id: thymeleaf-utext-xss
    pattern: th:utext="$X"
    message: "使用 th:utext 可能導致 XSS，應改用 th:text"
    languages: [generic]
    severity: ERROR
    paths:
      include: ["*.html"]
```

---

## 4. 掃描報告格式

所有工具結果彙整為統一表格：

```
【原始碼掃描報告 — OWASP Top 10 2025】

════════════════════════════════════════════════════════════════

📊 掃描摘要
├── 掃描範圍：Java + Thymeleaf
├── 掃描檔案：128 個
├── 工具：Semgrep + SpotBugs + Dependency Check
├── 掃描耗時：45 秒
│
├── 🔴 嚴重（Critical）：2 項
├── 🟠 高（High）：5 項
├── 🟡 中（Medium）：8 項
└── 🟢 低（Low）：12 項

════════════════════════════════════════════════════════════════

🔴 嚴重（Critical）

┌──────┬────────────────────────┬─────┬─────────────────────────┬──────────────────────┐
│ 編號 │ 檔案:行號              │ OWASP│ 問題類型               │ 修補建議             │
├──────┼────────────────────────┼─────┼─────────────────────────┼──────────────────────┤
│ C-01 │ UserDao.java:78        │ A05 │ SQL Injection          │ 改用 PreparedStatement│
│ C-02 │ login.html:23          │ A05 │ XSS (th:utext)         │ 改為 th:text          │
└──────┴────────────────────────┴─────┴─────────────────────────┴──────────────────────┘

【C-01 詳情】
檔案：src/main/java/com/example/UserDao.java
行號：78
OWASP：A05:2025 – Injection
CWE：CWE-89
工具：Semgrep (rule: java.lang.security.audit.formatted-sql-string)
風險說明：
  String sql = "SELECT * FROM users WHERE name = '" + username + "'";
  使用字串拼接產生 SQL，惡意輸入可導致注入攻擊。

修補方案：
  使用 JPA Named Parameter：
    @Query("SELECT u FROM User u WHERE u.name = :username")
    List<User> findByUsername(@Param("username") String username);

════════════════════════════════════════════════════════════════

🟠 高（High）
[同上格式]

════════════════════════════════════════════════════════════════

🟡 中（Medium）
[同上格式]

════════════════════════════════════════════════════════════════

🟢 低（Low）
[同上格式]

════════════════════════════════════════════════════════════════

📁 報告檔案位置
├── sast-java.sarif
├── spotbugs-report.xml
├── dependency-check-report.html
└── 原始碼掃描-summary.md（本報告）
```

---

## 5. 嚴重度分級定義

| 等級 | 定義 | 處理原則 |
|------|------|---------|
| 🔴 **Critical** | 可直接被利用、影響範圍廣、無前置條件 | **必須立即修補**，阻擋後續流程 |
| 🟠 **High** | 需特定條件或權限才能利用 | 本週內修補 |
| 🟡 **Medium** | 需多重條件、影響有限 | 排入 Backlog |
| 🟢 **Low** | 最佳實踐問題、加固建議 | 可選改善 |

---

## 6. 掃描結果處理

### 6.1 掃描通過（無 🔴 Critical、無 🟠 High）

```
【原始碼掃描結果】

✅ 掃描通過！

📊 掃描摘要
├── 🔴 Critical：0
├── 🟠 High：0
├── 🟡 Medium：{n}
└── 🟢 Low：{n}

原始碼靜態分析未發現嚴重資安問題。您可以選擇：
1. 輸入「OKOKYES」→ 啟動「網頁弱點掃描」Skill 進行 DAST
2. 輸入「繼續」→ 進入下一階段（Commit / 部署）
3. 輸入其他內容 → 繼續對話
```

### 6.2 掃描失敗（有 🔴 或 🟠）

```
【原始碼掃描結果】

❌ 掃描未通過

📊 掃描摘要
├── 🔴 Critical：{n} 項
├── 🟠 High：{n} 項
├── 🟡 Medium：{n}
└── 🟢 Low：{n}

發現必須修補的資安問題，請先完成修補後再繼續流程。

請輸入「OKOKYES」啟動對應的「開發-*」Skill 進行修補。
```

---

## 7. Skill 串接機制

### 7.1 串接上游：CodeReview

`CodeReview` 審查通過後，自動詢問是否執行原始碼掃描：
```
✅ CodeReview 通過 → 是否啟動「原始碼掃描」Skill？
請輸入「OKOKYES」啟動。
```

### 7.2 串接下游：掃描通過 → 網頁弱點掃描

```
【Skill 串接通知】

✅ 原始碼掃描通過
✅ 已找到對應的 Skill：「網頁弱點掃描」(/網頁弱點掃描)

此 Skill 將協助您進行：
- 啟動 OWASP ZAP 對本機服務進行黑箱掃描
- 補充 SAST 無法發現的執行階段漏洞

請輸入「OKOKYES」啟動，或輸入其他內容繼續對話。
```

### 7.3 串接下游：掃描失敗 → 開發修補

```
【Skill 串接通知】

❌ 原始碼掃描未通過
❌ 發現 {n} 項必須修補的資安問題
✅ 已找到對應的 Skill：「<開發 SKILL 名稱>」

請輸入「OKOKYES」啟動修補流程。
```

---

## 8. 安全與誤報處理

### 8.1 誤報標記

部分規則可能對特定情境誤報，允許使用者在報告中標記 `false-positive`：
- 必須註明原因
- 記錄於 `.semgrepignore` 或 `// nosec` 註解
- 下次掃描會略過但保留在報告的「已忽略」區塊

### 8.2 禁止行為

- ❌ 禁止未執行實際掃描就宣告通過
- ❌ 禁止忽略 🔴 Critical 項目
- ❌ 禁止在未告知的情況下自動安裝第三方工具
- ❌ 禁止將原始碼上傳至第三方雲端 SaaS 掃描服務（避免外洩）
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動 Skill

---

## 9. 與 CodeReview 的差異

| 面向 | CodeReview | 原始碼掃描 |
|------|-----------|-----------|
| 焦點 | 程式碼風格、可維護性、架構 | OWASP Top 10 資安漏洞 |
| 工具 | 人工 + 規則 | SAST 工具（Semgrep 等） |
| 觸發時機 | 開發完成後 | CodeReview 通過後 |
| 檢查深度 | 廣但淺 | 窄但深（資安專注） |
| 報告輸出 | 錯誤/警告/建議 | Critical/High/Medium/Low + OWASP 分類 |

兩者互補而非取代。

---

## 10. 相關文件

- [_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md) — 共享資安規範
- [網頁弱點掃描/SKILL.md](../網頁弱點掃描/SKILL.md) — 下一階段 DAST
- [滲透測試/SKILL.md](../滲透測試/SKILL.md) — 本機滲透測試
- OWASP Top 10 2025（RC）：https://owasp.org/Top10/2025/
- OWASP Top 10 2025 Introduction：https://owasp.org/Top10/2025/0x00_2025-Introduction/
- CWE-1436（OWASP Top Ten 2025 類別對照）：https://cwe.mitre.org/data/definitions/1436.html
- CWE 分類：https://cwe.mitre.org/
