---
name: 滲透測試
description: "本機服務滲透測試專家，針對使用者本機或私有網段服務進行五階段滲透（資訊蒐集 → 弱點識別 → 攻擊驗證 → 權限提升 → 後滲透）。使用 nmap、sqlmap、ffuf、httpx、jwt_tool、curl 等工具。強制目標白名單（localhost / RFC1918），禁止 DoS、禁止真實資料破壞、禁止外網目標。產出含攻擊路徑、PoC、CVSS、修補建議的完整滲透報告。當用戶說「/滲透測試」、「/pentest」、「滲透」、「模擬攻擊」時觸發。確認後使用者必須輸入 OKOKYES 才會執行滲透。"
---

# Skill: 滲透測試（Penetration Testing / OWASP Top 10 2025）

> 📌 **依據版本**：OWASP Top 10 **2025 Release Candidate**（2025-11-06 發布）
> 官方連結：https://owasp.org/Top10/2025/

> ⚠️ **本 SKILL 屬於三層資安檢測的第三層（最深入）**
> - 第一層：[原始碼掃描](../原始碼掃描/SKILL.md)（SAST）
> - 第二層：[網頁弱點掃描](../網頁弱點掃描/SKILL.md)（DAST）
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)

---

## ⚠️ 最高紅線（絕對禁止違反）

1. **禁止外網目標**：目標必須為 `localhost`、`127.0.0.1`、`::1`、`192.168.x.x`、`10.x.x.x`、`172.16-31.x.x`
2. **禁止 DoS**：不得使用大流量、壓力測試、連線耗盡等攻擊
3. **禁止真實資料破壞**：不得執行 `DROP TABLE`、`rm -rf`、`DELETE FROM` 等破壞性指令
4. **禁止社交工程**：不得產生釣魚郵件、假冒網頁
5. **禁止後門植入**：不得上傳 webshell、建立反連、竄改設定檔
6. **禁止轉發流量**：測試流量不得流向外網，所有封包留在本機

違反任一紅線，**立即終止測試**並告知使用者。

---

## Usage Trigger

- 使用者輸入 `/滲透測試`、`/pentest`
- 使用者說「滲透」、「模擬攻擊」、「攻擊我的服務看看」
- 原始碼掃描 + 網頁弱點掃描完成後，串接深度驗證
- 資安事件後的攻擊路徑還原

---

## 1. 前置確認流程（強制執行）

### 1.1 目標白名單檢查

與「網頁弱點掃描」相同的白名單邏輯：

| 允許 ✅ | 拒絕 ❌ |
|--------|--------|
| `localhost`、`127.0.0.1`、`::1` | 任何公網域名 / IP |
| `192.168.0.0/16` | 第三方服務 |
| `10.0.0.0/8` | 雲端主機（即使你擁有） |
| `172.16.0.0/12` | 同事的電腦 |
| `*.local`、`*.test` | |

**若目標不符合白名單**：
```
❌ 拒絕執行滲透測試

目標：{target}
理由：非本機 / 私有網段目標

依本 Skill 的安全紅線，僅允許對本機服務進行滲透測試。
若您需要測試雲端或公網服務，須改用其他流程（正式的授權滲透測試）。
```

> 雲端主機（AWS / GCP）即使是你擁有，仍屬於雲端供應商的共享基礎設施，**需有 AUP 授權**才能進行滲透，本 Skill 不支援。

### 1.2 測試類型詢問

```
【滲透測試類型確認】

請選擇測試類型：

1. **黑箱測試**（Black-box）
   - 不提供任何內部資訊
   - 模擬外部攻擊者視角
   - 測試時間長、覆蓋率較低

2. **灰箱測試**（Gray-box）
   - 提供一般使用者帳號
   - 模擬內部員工/已註冊使用者視角
   - 平衡覆蓋率與真實感

3. **白箱測試**（White-box）
   - 提供管理員帳號 + 原始碼 + 架構圖
   - 最深入、最全面
   - 適合正式發布前

請輸入數字（1 / 2 / 3）。
```

### 1.3 範圍與排除項目

```
【測試範圍確認】

目標服務：http://localhost:8080
測試類型：{選擇的類型}

請回答以下問題：

1. 測試時段：是否現在開始？或指定時間？
2. 端點範圍：
   - 全部端點
   - 指定 URL 前綴（例如 /api/*）
3. 排除項目（必填，預設至少包含以下，可追加）：
   - /logout
   - /admin/reset
   - 任何含 delete、drop、truncate 的端點
4. 資料庫狀態：
   - 是否為測試資料？（強烈建議 ✅）
   - 是否已備份？
5. 已知資訊（灰/白箱需提供）：
   - 帳號密碼
   - API 文件
   - 資料庫帳號（白箱）

確認後請輸入「OKOKYES」啟動測試。
```

### 1.4 雙重確認

```
⚠️ 最終確認

即將對以下目標執行滲透測試：

目標：http://localhost:8080
類型：白箱測試
範圍：所有 /api/* 端點
排除：/logout、/admin/reset、/api/*/delete
資料庫：已備份（2026-04-11 13:00:00）

測試行為可能包含：
- 埠掃描與服務指紋
- 目錄爆破（1000+ 常見路徑）
- SQL Injection PoC（使用 sqlmap）
- XSS PoC
- 認證繞過嘗試
- IDOR / 越權測試
- JWT 算法混淆測試
- 檔案上傳繞過

禁止行為：
❌ 任何對資料的實際破壞
❌ DoS / 大流量壓測
❌ 外網連線

請輸入「OKOKYES」啟動滲透測試，或輸入其他內容取消。
```

---

## 2. 五階段滲透流程

### Stage 1：資訊蒐集（Reconnaissance）

#### 1.1 埠掃描

```bash
# 快速掃描常見埠
nmap -sT -T4 --top-ports 1000 -oA recon/nmap-top1000 127.0.0.1

# 服務版本與指紋
nmap -sV -sC -p {發現的埠} -oA recon/nmap-version 127.0.0.1
```

#### 1.2 Web 服務指紋

```bash
# httpx 取得 title、status、tech stack
httpx -u http://localhost:8080 -tech-detect -title -status-code -server \
      -o recon/httpx.txt

# WhatWeb 備援
whatweb http://localhost:8080 --log-json=recon/whatweb.json
```

#### 1.3 目錄與檔案爆破

```bash
# ffuf 使用常見目錄字典
ffuf -u http://localhost:8080/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -mc 200,204,301,302,307,401,403 \
     -o recon/ffuf.json -of json \
     -rate 50  # 限速 50 req/s，避免打爆本機
```

**字典建議**：
- `common.txt`（4,727 條）
- `raft-medium-directories.txt`
- `api/objects.txt`（API 端點）

#### 1.4 JS 檔案分析

從前端 JS 檔案提取隱藏端點、API Key：
```bash
# 下載所有 JS
curl -s http://localhost:8080/ | grep -oP '(?<=src=")[^"]+\.js' | \
  xargs -I {} curl -s http://localhost:8080/{} > recon/all-js.txt

# 搜尋敏感字串
grep -E "(api[_-]?key|secret|token|password|/api/)" recon/all-js.txt
```

### Stage 2：弱點識別（Vulnerability Identification）

#### 2.1 整合 SAST / DAST 結果

若先前已執行「原始碼掃描」與「網頁弱點掃描」，**讀取其報告**作為輸入：
- 挑出 🔴 Critical / 🟠 High 的項目
- 優先驗證這些項目為**可利用**還是**誤報**

#### 2.2 常見漏洞點位清單

| 類別 | 測試點 |
|------|--------|
| **注入** | 所有含 query string、body 參數的端點 |
| **認證** | `/login`、`/register`、`/forgot-password`、JWT 端點 |
| **授權** | `/api/users/{id}`、`/api/orders/{id}`（IDOR） |
| **檔案上傳** | `/upload`、`/api/files` |
| **SSRF** | 任何含 URL 參數的功能（`?url=`、`?redirect=`） |
| **反序列化** | Cookie、隱藏欄位含 base64 / pickle 資料 |
| **XXE** | XML 上傳、SOAP API |

### Stage 3：攻擊驗證（Exploitation）

#### 3.1 SQL Injection 驗證

**僅對測試資料庫執行**，禁用破壞性 payload：

```bash
# sqlmap 使用安全參數
sqlmap -u "http://localhost:8080/api/users?id=1" \
       --batch \
       --level=3 --risk=1 \
       --technique=BEUSTQ \
       --exclude-sysdbs \
       --no-cast \
       --output-dir=pentest/sqlmap \
       --flush-session

# 禁止使用：
# --os-shell, --os-cmd, --file-write, --sql-shell
```

**列出資料庫、表格以驗證（僅做證據），禁止 dump 全部資料**：
```bash
sqlmap ... --dbs
sqlmap ... --tables -D {db}
sqlmap ... --columns -T users -D {db}
# ❌ 禁止：sqlmap ... --dump -T users
```

#### 3.2 XSS 驗證

```bash
# 使用 curl 送出 PoC
curl -G "http://localhost:8080/search" \
     --data-urlencode 'q=<script>alert(1)</script>' \
     -o pentest/xss-poc.html

# 檢查回應是否包含未 escape 的 script tag
grep -c '<script>alert(1)</script>' pentest/xss-poc.html
```

#### 3.3 IDOR / 越權

```bash
# 使用 user A 的 token 存取 user B 的資料
curl -H "Authorization: Bearer ${USER_A_TOKEN}" \
     http://localhost:8080/api/users/2/profile
# 預期：403 / 404
# 若回傳 200 + user B 資料 → 確認 IDOR
```

#### 3.4 JWT 測試

```bash
# jwt_tool 測試算法混淆、弱密鑰
jwt_tool ${JWT} -M at  # algorithm confusion
jwt_tool ${JWT} -C -d /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt
# ❌ 禁止使用外部 wordlist 連線
```

#### 3.5 檔案上傳繞過

- 嘗試上傳 `.jsp`、`.jspx`、`.php`、`.phtml`
- 雙副檔名：`shell.jpg.jsp`
- MIME type 偽造：`Content-Type: image/jpeg` + 實際 jsp 內容
- Magic bytes 偽造：`GIF89a;<?php ...?>`

**注意**：僅驗證可上傳 + 可執行的**證據**，禁止上傳真正的 webshell。

### Stage 4：權限提升（Privilege Escalation）

**只在測試環境執行**：

- 水平越權：A 使用者存取 B 使用者資料
- 垂直越權：一般使用者存取 `/admin/*` 端點
- 角色竄改：修改 JWT payload 中的 `role` 欄位
- Mass Assignment：註冊時傳入 `isAdmin=true`

### Stage 5：後滲透（Post-Exploitation）

**僅做證據蒐集，禁止實際操作**：

- **記錄能做什麼**，而非**真的去做**
- 例如：確認可以讀取 `/etc/passwd` → 截圖證據 → 停止
- 禁止：建立新帳號、修改管理員密碼、植入後門、刪除日誌

---

## 3. 工具清單

| 工具 | 用途 | 安裝 |
|------|------|------|
| **nmap** | 埠掃描、服務指紋 | `brew install nmap` |
| **sqlmap** | SQL Injection 自動化 | `brew install sqlmap` |
| **ffuf** | 目錄 / 參數爆破 | `brew install ffuf` |
| **httpx** | Web 指紋 | `brew install httpx` |
| **jwt_tool** | JWT 攻擊工具 | `pip install jwt-tool` |
| **curl** | 手動 PoC | 內建 |
| **gobuster** | 目錄爆破備援 | `brew install gobuster` |
| **seclists** | 字典集合 | `brew install seclists` |

**本 Skill 禁止使用**：
- Metasploit（過於強力，風險高）
- Cobalt Strike、Sliver（C2 框架）
- 任何勒索軟體、後門產生器
- hashcat 離線破解（與本機服務滲透無關）

---

## 4. 滲透測試報告格式

```
【滲透測試報告】

════════════════════════════════════════════════════════════════

📋 執行摘要
├── 目標：http://localhost:8080
├── 類型：白箱測試
├── 測試時間：2026-04-11 14:00 ~ 16:30
├── 測試人員：Claude Code + 使用者
├── 範圍：/api/* 所有端點
├── 排除：/logout、/admin/reset、刪除類端點
│
├── 🔴 可利用嚴重漏洞：2 項
├── 🟠 可利用高風險漏洞：3 項
├── 🟡 需特定條件可利用：5 項
└── 🟢 加固建議：8 項

════════════════════════════════════════════════════════════════

🔴 漏洞 P-01：SQL Injection（Union-based）

基本資訊
├── 位置：GET /api/users?id={id}
├── 參數：id
├── OWASP：A05:2025 – Injection
├── CWE：CWE-89
├── CVSS 3.1：9.8（Critical）
└── 工具：sqlmap

攻擊路徑
  Step 1: 使用者未登入，直接發送請求
  Step 2: payload: id=1 UNION SELECT 1,2,version()
  Step 3: 成功回傳資料庫版本
  Step 4: 列出所有資料表（users、orders、config）
  Step 5: 確認可讀取 users 表結構

PoC 證據
  Request:
    GET /api/users?id=1%20UNION%20SELECT%201,2,version() HTTP/1.1
    Host: localhost:8080

  Response:
    HTTP/1.1 200 OK
    {"id":1,"name":"2","email":"PostgreSQL 14.5"}
       ↑ 注入結果出現在回應中

影響評估
  - 機密性：可讀取所有資料表（使用者、訂單、設定）
  - 完整性：可修改資料（但測試過程未執行）
  - 可用性：可能導致資料庫效能問題
  - 實際損失：若在正式環境將導致完整資料外洩

修補建議
  1. 【立即】將所有 SQL 改用 PreparedStatement / JPA Named Parameter
     範例：
       @Query("SELECT u FROM User u WHERE u.id = :id")
       User findById(@Param("id") Long id);
  2. 【短期】後端驗證 id 為數字類型
  3. 【短期】最小權限資料庫帳號
  4. 【長期】導入 WAF + 定期 SAST 掃描

驗證指令
  修補後以下請求應回傳 400 或 404：
    curl "http://localhost:8080/api/users?id=1%20UNION%20SELECT%201"

════════════════════════════════════════════════════════════════

🔴 漏洞 P-02：...
🟠 漏洞 P-03：...
...

════════════════════════════════════════════════════════════════

📊 OWASP Top 10 2025 對應統計

┌──────┬──────────────────────────────────────┬─────────┬─────────┐
│ 編號 │ 類別（2025）                          │ 發現數量 │ 已利用   │
├──────┼──────────────────────────────────────┼─────────┼─────────┤
│ A01  │ Broken Access Control（含 SSRF）     │ 3        │ 2        │
│ A02  │ Security Misconfiguration            │ 2        │ 1        │
│ A03  │ Software Supply Chain Failures       │ 1        │ 0        │
│ A04  │ Cryptographic Failures               │ 1        │ 0        │
│ A05  │ Injection（含 XSS）                  │ 2        │ 2        │
│ A06  │ Insecure Design                      │ 0        │ 0        │
│ A07  │ Authentication Failures              │ 1        │ 1        │
│ A08  │ Software or Data Integrity Failures  │ 0        │ 0        │
│ A09  │ Logging and Alerting Failures        │ 2        │ 0        │
│ A10  │ Mishandling of Exceptional Conditions│ 1        │ 1        │
└──────┴──────────────────────────────────────┴─────────┴─────────┘

════════════════════════════════════════════════════════════════

📁 證據檔案位置
├── pentest/recon/nmap-*.xml
├── pentest/recon/ffuf.json
├── pentest/sqlmap/ (工具輸出)
├── pentest/evidence/  (截圖 / HTTP log)
└── pentest/滲透測試-summary.md

════════════════════════════════════════════════════════════════

🔧 建議修補順序

1. 【本週】修補 🔴 P-01 SQL Injection
2. 【本週】修補 🔴 P-02 JWT 驗證繞過
3. 【2 週內】修補 🟠 高風險項目
4. 【1 個月內】修補 🟡 中風險項目
5. 【持續】加固建議項目

修補後請重新執行完整資安檢測流程：
SAST → DAST → 滲透測試
```

---

## 5. 測試結果處理

### 5.1 無嚴重漏洞

```
【滲透測試結果】

✅ 無可利用的嚴重漏洞

雖然發現 {n} 項建議加固項目，但未發現可直接利用的 Critical / High 漏洞。

建議：
- 繼續保持定期資安檢測
- 加固建議項目可排入 Backlog
```

### 5.2 發現可利用漏洞

```
【滲透測試結果】

❌ 發現可利用漏洞

🔴 Critical：{n}
🟠 High：{n}

必須立即修補，並重新跑一次完整資安檢測。
請輸入「OKOKYES」啟動對應的「開發-*」Skill 進行修補。
```

---

## 6. Skill 串接機制

### 6.1 上游

- `原始碼掃描` → `網頁弱點掃描` → **`滲透測試`**（本 Skill）

### 6.2 下游

- 發現漏洞 → 串接對應「開發-*」Skill 修補
- 無漏洞 → 產出最終報告並結束

---

## 7. 測試過程中的即時防呆

每執行一個工具前，**再次確認目標為白名單**：

```bash
# 所有工具呼叫前的檢查 pseudo code
function check_target() {
  local target=$1
  if [[ ! $target =~ ^(localhost|127\.|::1|192\.168\.|10\.|172\.(1[6-9]|2[0-9]|3[01])\.) ]]; then
    echo "❌ 拒絕執行：目標不在白名單"
    exit 1
  fi
}
```

若在測試過程中，工具自動延伸到白名單外的目標（例如 sqlmap 自動跟隨 redirect），**立即停止並告知使用者**。

---

## 8. 禁止行為（再次強調）

| 類別 | 禁止項目 |
|------|---------|
| **目標** | 公網、雲端、第三方、未授權主機 |
| **資料** | 真實刪除、修改、dump 全表、竄改設定 |
| **流量** | DoS、壓力測試、大流量爆破（rate > 200/s） |
| **工具** | Metasploit、C2 框架、勒索軟體、後門產生器 |
| **行為** | 社交工程、釣魚、真實植入 webshell、修改系統檔 |
| **外洩** | 將發現的漏洞資料上傳第三方 SaaS |
| **確認** | 接受 `OKOKYES` 以外的確認詞彙啟動測試 |

---

## 9. 法律與道德聲明

- 本 Skill 僅用於使用者自有之本機 / 私有環境測試
- 使用者須確保目標屬於自身擁有且有權測試
- 產出的漏洞資訊僅供修補使用，不得外流
- 禁止將本 Skill 用於任何未授權的攻擊行為
- 違反以上原則的後果由使用者自負

---

## 10. 相關文件

- [原始碼掃描/SKILL.md](../原始碼掃描/SKILL.md) — 第一層 SAST
- [網頁弱點掃描/SKILL.md](../網頁弱點掃描/SKILL.md) — 第二層 DAST
- [_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md) — 共享資安規範
- OWASP Top 10 2025（RC）：https://owasp.org/Top10/2025/
- OWASP Testing Guide：https://owasp.org/www-project-web-security-testing-guide/
- PTES（Penetration Testing Execution Standard）：http://www.pentest-standard.org/
- CVSS 3.1 計算器：https://www.first.org/cvss/calculator/3.1
