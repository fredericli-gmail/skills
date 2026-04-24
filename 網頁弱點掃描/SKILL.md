---
name: 網頁弱點掃描
description: "網頁應用動態弱點掃描（DAST）專家，依 OWASP Top 10 2025 RC，使用 OWASP ZAP + nikto 對本機運行中的服務進行黑箱掃描，涵蓋 Broken Access Control（含 SSRF）、Security Misconfiguration、Injection（XSS / SQLi）、Cryptographic Failures、Authentication Failures、Logging & Alerting Failures 等執行階段漏洞。強制僅掃描本機 / 私有 IP 目標，限速避免服務崩潰，危險路徑黑名單保護。當用戶說「/網頁弱點掃描」、「/DAST」、「/ZAP掃描」、「掃描網站」、「弱點掃描」時觸發。確認後使用者必須輸入 OKOKYES 才會執行掃描。"
---

# Skill: 網頁弱點掃描（DAST / OWASP Top 10 2025）

> 📌 **依據版本**：OWASP Top 10 **2025 Release Candidate**（2025-11-06 發布）
> 官方連結：https://owasp.org/Top10/2025/

> ⚠️ **本 SKILL 屬於三層資安檢測的第二層（DAST）**
> - 第一層：[原始碼掃描](../原始碼掃描/SKILL.md)（SAST）
> - 第三層：[滲透測試](../滲透測試/SKILL.md)
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)

---

## Usage Trigger

- 使用者輸入 `/網頁弱點掃描`、`/DAST`、`/ZAP掃描`
- 使用者說「掃描網站」、「弱點掃描」、「黑箱掃描」
- 原始碼掃描通過後，串接觸發 DAST 補強
- 部署至測試環境後的驗收檢查

---

## 1. 前置安全檢查（強制執行）

### 1.1 目標白名單檢查

**只允許掃描本機與私有 IP 範圍**：

| 允許 ✅ | 拒絕 ❌ |
|--------|--------|
| `localhost` | 任何公網域名（`.com`、`.net`、`.tw` 等） |
| `127.0.0.1` / `::1` | 公網 IP |
| `192.168.0.0/16` | 第三方 SaaS |
| `10.0.0.0/8` | 其他人的伺服器 |
| `172.16.0.0/12` | |
| `*.local`、`*.test`、`*.localhost` | |

**檢查邏輯**：
```bash
# 從使用者輸入的 URL 提取 host
# 解析 host 對應的 IP
# 比對是否屬於上述 RFC1918 私有範圍
```

若目標不在白名單，**拒絕掃描並結束流程**：
```
❌ 目標不在白名單內

目標：https://example.com
理由：此為公網域名，非本機 / 私有環境。

本 Skill 僅允許掃描您擁有的本機或私有網路服務，
以避免誤對未授權目標進行掃描（可能違法）。

若您確認此目標屬於您自有且授權測試，請聯繫維護者調整白名單設定。
```

### 1.2 服務存活檢查

```bash
# 確認目標可連線
curl -I --max-time 5 http://localhost:8080/
```

若服務未啟動，提示使用者先啟動服務。

### 1.3 危險路徑黑名單

**掃描過程中自動排除以下路徑，避免觸發破壞性操作**：

| 路徑樣式 | 原因 |
|---------|------|
| `/logout` | 避免頻繁登出導致 session 中斷 |
| `/delete`、`*/delete/*` | 避免誤刪資料 |
| `/admin/reset` | 避免重置系統 |
| `/shutdown` | 避免關閉服務 |
| `/api/*/delete` | REST 刪除端點 |
| `/truncate`、`/drop` | 資料庫危險操作 |

使用者可在執行前補充額外黑名單。

### 1.4 服務備份提醒

```
⚠️ 掃描前提醒

即將對本機服務執行弱點掃描，掃描過程會送出大量請求。
雖然已設定速率限制與危險路徑黑名單，仍強烈建議：

1. 使用測試環境資料庫，而非正式資料庫
2. 掃描前備份資料庫（若在開發機）
3. 關閉與正式環境的連線（Webhook、第三方 API）

是否繼續？請輸入「OKOKYES」確認。
```

---

## 2. 工具可用性檢查與安裝

| 工具 | 檢查指令 | 安裝方式 |
|------|---------|---------|
| **OWASP ZAP**（推薦 Docker） | `docker pull ghcr.io/zaproxy/zaproxy:stable` | Docker（免 GUI） |
| **nikto** | `nikto -Version` | `brew install nikto` |
| **wapiti**（備援） | `wapiti --version` | `pip install wapiti3` |
| **curl** | 內建 | - |

**建議使用 Docker 版 ZAP**，無須處理 JRE 與 GUI，啟動簡單：
```bash
docker run --rm -t \
  --network host \
  -v $(pwd):/zap/wrk \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py -t http://localhost:8080 \
  -r zap-baseline-report.html
```

---

## 3. 掃描模式與流程

### 3.1 四階段流程

```
Step 1: 確認目標與白名單 → 必須 OKOKYES
Step 2: 處理認證（若需登入）
Step 3: ZAP Baseline Scan（被動掃描，安全）
Step 4: ZAP Full Scan（主動掃描，需再次 OKOKYES）
Step 5: nikto 補充（伺服器層級設定）
Step 6: 產出彙整報告
```

### 3.2 Step 2：認證處理

若目標需要登入，詢問使用者提供：

| 認證類型 | 提供方式 |
|---------|---------|
| **Session Cookie** | 使用者貼上 Cookie（從瀏覽器 DevTools 複製） |
| **Bearer Token** | 提供 JWT 或 API Key |
| **Basic Auth** | 提供帳號密碼（僅限本機） |
| **表單登入** | 提供登入 URL、欄位名稱、帳密 |

ZAP 透過 `-z "replacer.full_list(0).description=auth1;replacer.full_list(0).enabled=true;replacer.full_list(0).matchtype=REQ_HEADER;replacer.full_list(0).matchstr=Authorization;replacer.full_list(0).replacement=Bearer xxx"` 注入。

### 3.3 Step 3：Baseline Scan（被動）

**特性**：
- 只做爬蟲 + 被動分析，不送惡意 payload
- 時間短（約 1-5 分鐘）
- 對服務影響最小

```bash
docker run --rm -t \
  --network host \
  -v $(pwd)/reports:/zap/wrk \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://localhost:8080 \
  -r baseline.html \
  -J baseline.json \
  -x baseline.xml \
  -I  # 遇到警告不 fail
```

**檢查項目**：
- Missing Security Headers（CSP、HSTS、X-Frame-Options、X-Content-Type-Options）
- Cookie 未設 Secure / HttpOnly / SameSite
- 資訊洩漏（Server header、版本號、stack trace）
- 敏感檔案暴露（`.git/`、`.env`、`backup.zip`）
- Cross-Domain 設定錯誤（CORS *）

### 3.4 Step 4：Full Scan（主動）

**特性**：
- 送出惡意 payload 測試漏洞
- 時間較長（10 分鐘～數小時）
- 可能影響服務

**再次要求 `OKOKYES`**：
```
⚠️ 即將啟動 ZAP Full Scan（主動掃描）

此掃描將送出以下類型的測試請求：
- SQL Injection payloads
- XSS payloads
- Command Injection payloads
- Path Traversal payloads
- LDAP Injection payloads

預估時間：10-60 分鐘（依端點數量）
預估請求數：1,000 - 10,000+

是否繼續？請輸入「OKOKYES」確認。
```

```bash
docker run --rm -t \
  --network host \
  -v $(pwd)/reports:/zap/wrk \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-full-scan.py \
  -t http://localhost:8080 \
  -r full.html \
  -J full.json \
  -I \
  -z "-config spider.maxDuration=5 -config scanner.maxRuleDurationInMins=3"
```

**限速參數**（避免打爆本機服務）：
- `spider.maxChildren=30`
- `spider.maxDuration=5`（分鐘）
- `scanner.threadPerHost=2`
- `scanner.maxRuleDurationInMins=3`

### 3.5 Step 5：nikto 補充

```bash
nikto -h http://localhost:8080 \
      -Format json \
      -output nikto-report.json \
      -maxtime 300s
```

**檢查項目**：
- 伺服器版本已知漏洞
- 預設頁面（`/phpinfo.php`、`/server-status`）
- 過時的套件
- robots.txt 暴露的敏感路徑

---

## 4. 掃描報告格式

彙整 ZAP + nikto 結果為統一表格：

```
【網頁弱點掃描報告 — DAST / OWASP Top 10 2025】

════════════════════════════════════════════════════════════════

📊 掃描摘要
├── 目標：http://localhost:8080
├── 掃描模式：Baseline + Full Scan
├── 工具：OWASP ZAP 2.15.0 + nikto 2.5
├── 掃描開始：2026-04-11 14:00:00
├── 掃描結束：2026-04-11 14:45:22
├── 總請求數：8,342
│
├── 🔴 嚴重（Critical）：1 項
├── 🟠 高（High）：3 項
├── 🟡 中（Medium）：7 項
└── 🟢 低（Low）：15 項

════════════════════════════════════════════════════════════════

🔴 嚴重（Critical）

┌──────┬──────────────────────────┬─────┬─────────────────────┬──────────────────┐
│ 編號 │ URL                       │ OWASP│ 問題                │ 修補建議         │
├──────┼──────────────────────────┼─────┼─────────────────────┼──────────────────┤
│ C-01 │ /api/users?id=1           │ A05 │ SQL Injection       │ 改用 Parameterized│
└──────┴──────────────────────────┴─────┴─────────────────────┴──────────────────┘

【C-01 詳情】
URL：http://localhost:8080/api/users?id=1
方法：GET
參數：id
OWASP：A05:2025 – Injection
CWE：CWE-89
工具：ZAP Active Scanner (rule: 40018)

攻擊 payload：
  id=1' AND '1'='1

回應差異：
  正常回應：200 OK, body length 1024
  注入回應：200 OK, body length 1024（同樣回傳，確認為布林注入）

證據：
  GET /api/users?id=1%27%20AND%20%271%27%3D%271
  HTTP/1.1 200 OK
  {"user": "admin", ...}

修補方案：
  1. 後端改用 PreparedStatement / JPA Named Parameter
  2. 前端驗證 id 必須為數字
  3. WAF 規則補強（暫時緩解）

════════════════════════════════════════════════════════════════

🟠 高（High）
[同上格式]

════════════════════════════════════════════════════════════════

🟡 中（Medium）
[同上格式]

════════════════════════════════════════════════════════════════

🟢 低（Low）— 通常是 Security Headers 缺失
[同上格式]

════════════════════════════════════════════════════════════════

📁 報告檔案位置
├── reports/baseline.html
├── reports/full.html
├── reports/nikto-report.json
└── reports/網頁弱點掃描-summary.md
```

---

## 5. 嚴重度分級

| 等級 | ZAP 對應 | 處理原則 |
|------|---------|---------|
| 🔴 **Critical** | High Risk + High Confidence | 立即修補，阻擋部署 |
| 🟠 **High** | Medium Risk + High Confidence | 本週內修補 |
| 🟡 **Medium** | Low Risk 或 Medium Confidence | 排入 Backlog |
| 🟢 **Low** | Info 或 Low Confidence | 加固建議 |

---

## 6. 掃描結果處理

### 6.1 掃描通過（無 🔴、無 🟠）

```
【網頁弱點掃描結果】

✅ 掃描通過！

📊 掃描摘要
├── 🔴 Critical：0
├── 🟠 High：0
├── 🟡 Medium：{n}
└── 🟢 Low：{n}

動態掃描未發現嚴重資安問題。您可以選擇：
1. 輸入「OKOKYES」→ 啟動「滲透測試」Skill 進行深度驗證
2. 輸入「完成」→ 結束資安檢測流程
3. 輸入其他內容 → 繼續對話
```

### 6.2 掃描失敗（有 🔴 或 🟠）

```
【網頁弱點掃描結果】

❌ 掃描未通過

📊 掃描摘要
├── 🔴 Critical：{n} 項
├── 🟠 High：{n} 項
├── 🟡 Medium：{n}
└── 🟢 Low：{n}

發現必須修補的執行階段漏洞，請先完成修補。

請輸入「OKOKYES」啟動對應的「開發-*」Skill 進行修補。
```

---

## 7. Skill 串接機制

### 7.1 上游：原始碼掃描

`原始碼掃描` 完成後可串接：
```
✅ 原始碼掃描通過 → 是否啟動「網頁弱點掃描」？
```

### 7.2 下游：通過 → 滲透測試

```
【Skill 串接通知】

✅ 網頁弱點掃描通過
✅ 已找到對應的 Skill：「滲透測試」(/滲透測試)

此 Skill 將協助您進行：
- 針對本機服務進行深度滲透測試
- 結合 SAST/DAST 結果挑出可利用點
- 驗證實際攻擊路徑

請輸入「OKOKYES」啟動，或輸入「完成」結束流程。
```

### 7.3 下游：失敗 → 開發修補

參考「原始碼掃描」Skill 的相同機制。

---

## 8. 禁止行為

- ❌ 禁止掃描非白名單（公網）目標
- ❌ 禁止未告知使用者即啟動 Full Scan
- ❌ 禁止掃描危險路徑黑名單
- ❌ 禁止使用 Cloud-based SaaS 掃描服務（資料外洩風險）
- ❌ 禁止對正式環境資料庫執行 Full Scan
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動掃描

---

## 9. 常見問題排除

### 9.1 ZAP Docker 無法連到本機服務

**問題**：`docker run` 內 `localhost` 指向容器本身，非主機。

**解決**：
- macOS：使用 `host.docker.internal:8080`
- Linux：使用 `--network host`
- Windows：同 macOS

### 9.2 掃描速度過慢

- 縮小 `spider.maxDuration`
- 排除靜態資源路徑（`*.js`、`*.css`、`*.png`）
- 使用 `-context` 指定特定 URL pattern

### 9.3 誤報

- 在 ZAP 的 `~/.ZAP/config.xml` 加入忽略規則
- 報告中標記 `false-positive` 並註明原因

---

## 10. 相關文件

- [原始碼掃描/SKILL.md](../原始碼掃描/SKILL.md) — 上一階段 SAST
- [滲透測試/SKILL.md](../滲透測試/SKILL.md) — 下一階段滲透測試
- [_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md) — 共享資安規範
- OWASP Top 10 2025（RC）：https://owasp.org/Top10/2025/
- OWASP ZAP 文件：https://www.zaproxy.org/docs/
- ZAP Docker：https://www.zaproxy.org/docs/docker/
