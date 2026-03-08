---
name: 測試
description: "Playwright 自動化測試。必須：(1) 先掃描程式碼取得元素定位 (2) 使用實體瀏覽器（禁用 headless）(3) 設定視窗可見（viewport 1920x1080）(4) 啟用 slow_mo 方便觀察。"
---

# Playwright 自動化測試

> **最高原則**：在使用者回覆 `OKOKYES` 確認測試計畫前，**絕對禁止**撰寫或執行任何測試程式碼！

---

## 強制使用實體瀏覽器（極重要）

```
+===============================================================================+
|                                                                               |
|   絕對禁止使用 HEADLESS 模式！必須開啟真實瀏覽器視窗！                         |
|                                                                               |
|   正確：開啟實體瀏覽器視窗，使用者可以看到測試過程                             |
|   錯誤：使用 headless=True，瀏覽器在背景執行看不到                            |
|                                                                               |
+===============================================================================+
```

### 為什麼必須使用實體瀏覽器？

1. **可視化驗證**：使用者可以親眼看到測試執行過程
2. **除錯方便**：出錯時可以立即看到畫面狀態
3. **真實環境**：模擬真實使用者操作，避免 headless 特有的問題
4. **開發模式**：開發環境測試需要即時觀察

### Python 強制設定

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    # ===============================================================
    # 絕對禁止以下設定：
    # ===============================================================
    # browser = p.chromium.launch(headless=True)    # 禁止！
    # browser = p.chromium.launch()                 # 禁止！（預設為 headless=True）
    # ===============================================================

    # 必須使用的設定（確保視窗可見）：
    browser = p.chromium.launch(
        headless=False,       # 強制開啟實體瀏覽器
        slow_mo=500           # 每步間隔 500ms，方便肉眼觀察
    )

    # 建立瀏覽器上下文，設定視窗大小
    context = browser.new_context(
        viewport={"width": 1920, "height": 1080}   # 固定視窗大小
    )

    page = context.new_page()
```

### Java 強制設定

```java
import com.microsoft.playwright.*;

public class TestSetup {
    public static void main(String[] args) {
        try (Playwright playwright = Playwright.create()) {
            // ===============================================================
            // 絕對禁止以下設定：
            // ===============================================================
            // Browser browser = playwright.chromium().launch();  // 禁止！（預設 headless）
            // new LaunchOptions().setHeadless(true)              // 禁止！
            // ===============================================================

            // 必須使用的設定（確保視窗可見）：
            Browser browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions()
                    .setHeadless(false)     // 強制開啟實體瀏覽器
                    .setSlowMo(500)         // 每步間隔 500ms
            );

            BrowserContext context = browser.newContext(
                new Browser.NewContextOptions()
                    .setViewportSize(1920, 1080)   // 固定視窗大小
            );

            Page page = context.newPage();
        }
    }
}
```

---

## 觸發情境

此 Skill 通常由 CodeReview Skill 審查通過後串接觸發，此時 Claude 已知道：
- 剛剛開發了什麼功能
- 異動了哪些檔案
- 專案的技術架構

---

## 強制執行流程

```
Step 1: 讀取 CLAUDE.md + 回顧開發內容 → Step 2: 掃描程式碼 → Step 3: 產出測試計畫 → 等待 OKOKYES → Step 4: 執行測試
         |
    取得測試 URL、帳號、驗證碼設定
```

---

## Step 1: 回顧開發內容與專案資訊（自動執行）

> Claude 必須自動收集所有必要資訊，減少詢問使用者。

### 1.1 讀取專案 CLAUDE.md（最優先）

> **必須先讀取**：專案的 CLAUDE.md 通常包含測試所需的關鍵資訊。

**讀取路徑**（依序嘗試）：
```bash
# 專案根目錄
cat ./CLAUDE.md
cat ./.claude/CLAUDE.md

# 或從 git 根目錄
cat $(git rev-parse --show-toplevel)/CLAUDE.md
cat $(git rev-parse --show-toplevel)/.claude/CLAUDE.md
```

**從 CLAUDE.md 提取的資訊**：

| 資訊類型 | 說明 | 範例 |
|---------|------|------|
| 測試 URL | 開發環境網址 | `http://localhost:8080` |
| 登入頁路徑 | 登入頁面的 URL | `/login`, `/auth/login` |
| 測試帳號 | 開發環境的測試帳號 | `admin / admin123` |
| 資料庫連線 | 開發環境 DB 資訊 | `localhost:5432/devdb` |
| 驗證碼規則 | 圖形驗證碼/OTP 取得方式 | `固定值 1234`、`關閉驗證` |
| 特殊設定 | 開發模式特有設定 | `spring.profiles.active=dev` |

**輸出格式**：
```
【專案 CLAUDE.md 資訊】

檔案位置：[路徑]
測試 URL：[從 CLAUDE.md 取得]
登入頁：[從 CLAUDE.md 取得]
測試帳號：[從 CLAUDE.md 取得，若有]
資料庫：[從 CLAUDE.md 取得，若有]
驗證碼：[從 CLAUDE.md 取得，若有]
```

> 若 CLAUDE.md 已包含測試帳號，則在 Step 3 不需再詢問使用者。

### 1.2 彙整開發資訊

從前面的對話上下文中提取：

```
【開發內容回顧】

開發功能：[從對話中取得，例：使用者登入功能]
異動檔案：
   [檔案1路徑]
   [檔案2路徑]
   [檔案3路徑]

測試目標：
   - [功能1：例如登入成功]
   - [功能2：例如登入失敗顯示錯誤]
   - [功能3：例如記住我功能]
```

### 1.3 取得專案設定（補充）

若 CLAUDE.md 未提供完整資訊，則掃描設定檔：

```bash
# 掃描 application 設定
grep -rn 'server.port\|server.servlet.context-path' --include="*.properties" --include="*.yml" src/main/resources/

# 掃描 profile 設定
grep -rn 'spring.profiles.active' --include="*.properties" --include="*.yml" src/main/resources/
```

**輸出格式**：
```
測試環境（從設定檔補充）：
   - Port: [從設定檔取得，例：8080]
   - Context Path: [從設定檔取得，例：/]
   - Profile: [從設定檔取得，例：dev]
   - 測試 URL: http://localhost:[port][context-path]
```

### 1.4 資訊來源優先順序

| 優先順序 | 來源 | 說明 |
|---------|------|------|
| 1 | 專案 CLAUDE.md | 最優先，通常有完整開發環境資訊 |
| 2 | application-dev.yml | 開發環境設定檔 |
| 3 | application.yml | 預設設定檔 |
| 4 | 詢問使用者 | 最後手段，上述都找不到時才詢問 |

> 若所有來源都無法取得測試 URL，才詢問使用者確認。

---

## Step 2: 掃描程式碼（必須完成）

> **禁止跳過此步驟**：必須掃描程式碼取得真實的元素定位，禁止猜測！

### 2.1 掃描前端元素

針對**異動的檔案及其相關頁面**進行掃描：

```bash
# 掃描 HTML/Thymeleaf 頁面的 ID、name、data-testid 屬性
grep -rn 'id="\|name="\|th:id="\|th:name=\|data-testid=' --include="*.html" src/main/resources/templates/

# 掃描表單元素
grep -rn '<form\|<input\|<button\|<select\|<textarea' --include="*.html" src/main/resources/templates/

# 掃描 JavaScript 的元素操作
grep -rn 'getElementById\|querySelector\|querySelectorAll' --include="*.js" src/main/resources/static/js/

# 掃描 aria-label（Playwright 可用 get_by_role 定位）
grep -rn 'aria-label=\|role=' --include="*.html" src/main/resources/templates/
```

### 2.2 掃描後端 API

```bash
# 掃描相關 Controller 端點
grep -rn '@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping\|@RequestMapping' --include="*.java" src/main/java/

# 掃描登入相關（若測試涉及登入）
grep -rn 'login\|Login\|logout\|Logout\|session\|Session' --include="*.java" --include="*.html" src/
```

### 2.3 產出元素定位清單

掃描完成後，**必須**整理成以下格式：

```
【掃描結果 - 元素定位清單】

================================================================================
                              頁面元素掃描
================================================================================

## 頁面：[頁面名稱] (src/main/resources/templates/xxx.html)

| 元素名稱 | Playwright 定位方式 | 定位值 | 行號 | 用途說明 |
|---------|-------------------|--------|------|---------|
| 帳號輸入框 | page.locator("#username") | id=username | :25 | 登入帳號輸入 |
| 密碼輸入框 | page.locator("#password") | id=password | :30 | 登入密碼輸入 |
| 登入按鈕 | page.get_by_role("button", name="登入") | role=button | :35 | 提交登入表單 |
| 錯誤訊息 | page.locator("#errorMsg") | id=errorMsg | :40 | 顯示登入錯誤 |

================================================================================
                              API 端點掃描
================================================================================

| 功能 | Method | Endpoint | Controller | 行號 |
|-----|--------|----------|------------|------|
| 登入頁面 | GET | /login | AuthController | :20 |
| 登入處理 | POST | /login | AuthController | :35 |
| 登出 | POST | /logout | AuthController | :50 |

================================================================================
```

### 2.4 Playwright 元素定位優先順序

| 優先順序 | 定位方式 | 適用場景 | 範例 |
|---------|---------|---------|------|
| 1 | `get_by_role()` | 有 role/aria-label 的元素 | `page.get_by_role("button", name="登入")` |
| 2 | `get_by_test_id()` | 有 data-testid 的元素 | `page.get_by_test_id("submit-btn")` |
| 3 | `get_by_text()` | 有明確文字的元素 | `page.get_by_text("歡迎回來")` |
| 4 | `get_by_label()` | 有 label 關聯的表單欄位 | `page.get_by_label("帳號")` |
| 5 | `get_by_placeholder()` | 有 placeholder 的輸入框 | `page.get_by_placeholder("請輸入帳號")` |
| 6 | `locator()` | CSS/XPath 定位 | `page.locator("#username")` |

---

## 測試原則（強制）

> **所有測試案例必須遵守以下原則，違反即為無效測試。**

### 1. 禁止純 UI 驗證
不允許只檢查「元素是否可見」就算通過，每個測試案例必須有真實的資料輸入與結果驗證。

### 2. 端到端資料流（實體瀏覽器優先）
- **有瀏覽器可用時**：必須透過實體瀏覽器進行完整的端到端操作（輸入 → 點擊 → 頁面跳轉 → 結果驗證），禁止只用 API 寫入資料來取代使用者操作
- **無瀏覽器時**：才允許退而使用 API 呼叫模擬操作流程
- 每個測試必須走完「使用者操作 → 畫面回饋 → 資料寫入 → 後續流程 → 最終狀態」的完整鏈路

### 3. 資料驗證（實體瀏覽器優先）
- **有瀏覽器可用時**：優先透過實體瀏覽器操作來驗證資料（例如：新增後到列表頁確認資料出現、編輯後重新開啟確認值已更新）
- **無瀏覽器時**：才允許透過 API 查詢驗證資料確實寫入
- **最後手段**：若 API 也無法覆蓋，再透過 DB 直連驗證（需注意連線安全）

### 4. 後續作業驗證
若操作觸發非同步流程（例如建立任務 → Agent 開始執行 → 產出輸出），必須驗證到最後一步。非同步等待應使用 polling + 合理 timeout，避免 flaky test。

### 5. 測試資料管理
- **Setup 階段**：清理可能的殘留資料，確保初始狀態乾淨
- **Teardown 階段**：清理本次測試建立的資料
- 即使測試失敗，清理邏輯也必須執行（使用 fixture/finally）

### 範例對照

**登入功能**：
- ❌ 打開登入頁 → 看到帳號框 → pass
- ✅ 在瀏覽器輸入帳密 → 點登入 → 驗證拿到 JWT Token → 用 Token 呼叫受保護 API 確認有效 → 驗證使用者資訊正確

**快速啟動功能**：
- ❌ 看到啟動按鈕 → 填 URL → 按啟動 → 看到終端機頁面 → pass
- ✅ 在瀏覽器填入 Git URL → 按啟動 → 驗證任務已建立（API 查詢狀態）→ 等待 Agent 開始執行 → 驗證 CLI 有產出輸出 → 發送使用者訊息 → 驗證訊息被回顯 → 驗證 Agent 回應 → 終止任務 → 驗證狀態變更為已終止

---

## Step 3: 產出測試計畫（必須完成）

> 根據 Step 1 的開發內容 + Step 2 的掃描結果，**自動**產出完整測試計畫。

### 3.1 測試計畫格式

```
【測試計畫書】

================================================================================
                              測試基本資訊
================================================================================

測試目標：[從 Step 1 取得的功能描述]
測試 URL：[從 Step 1 取得的 URL]
瀏覽器：Chromium（實體視窗，禁用 headless，slow_mo=500）
測試框架：pytest + playwright
測試類別：tests/e2e/test_[功能名].py

================================================================================
                              測試帳號需求
================================================================================

【情況 A：CLAUDE.md 已提供帳號】

從專案 CLAUDE.md 取得以下測試帳號：

| # | 角色 | 帳號 | 密碼 | 用於測試案例 |
|---|-----|------|------|-------------|
| 1 | [角色名] | [帳號] | [密碼] | TC-001, TC-002 |

驗證碼設定：[從 CLAUDE.md 取得，例：開發環境已關閉驗證碼]

---

【情況 B：需要使用者提供帳號】

根據測試案例，需要以下帳號：

| # | 角色 | 用於測試案例 |
|---|-----|-------------|
| 1 | [角色名] | [TC-001, TC-002] |
| 2 | [角色名] | [TC-003] |

請提供：
1. 帳號 1 ([角色名]) 的帳號/密碼：
2. 帳號 2 ([角色名]) 的帳號/密碼：

【驗證機制】（若有）
- 圖形驗證碼取得方式：
- OTP 驗證碼取得方式：

================================================================================
                              測試案例
================================================================================

## TC-001: [測試案例名稱，例：正確帳密登入成功]

**測試目的**：驗證 [具體目的]

**前置條件**：
1. 測試帳號已建立
2. 系統正常運作

**測試步驟**：

| 步驟 | 操作 | Playwright 指令 | 預期結果 |
|-----|------|----------------|---------|
| 1 | 開啟登入頁面 | page.goto("[URL]/login") | 顯示登入表單 |
| 2 | 輸入帳號 | page.locator("#username").fill("[帳號]") | 欄位顯示輸入值 |
| 3 | 輸入密碼 | page.locator("#password").fill("[密碼]") | 欄位顯示遮罩 |
| 4 | 點擊登入按鈕 | page.get_by_role("button", name="登入").click() | - |
| 5 | 驗證登入成功 | expect(page).to_have_url(re.compile(r".*/dashboard")) | URL 包含 /dashboard |

**預期結果**：成功跳轉至首頁，顯示歡迎訊息

---

## TC-002: [測試案例名稱，例：錯誤密碼登入失敗]

**測試目的**：驗證錯誤密碼時顯示適當錯誤訊息

**測試步驟**：

| 步驟 | 操作 | Playwright 指令 | 預期結果 |
|-----|------|----------------|---------|
| 1 | 開啟登入頁面 | page.goto("[URL]/login") | 顯示登入表單 |
| 2 | 輸入帳號 | page.locator("#username").fill("[帳號]") | 欄位顯示輸入值 |
| 3 | 輸入錯誤密碼 | page.locator("#password").fill("wrongpassword") | 欄位顯示遮罩 |
| 4 | 點擊登入按鈕 | page.get_by_role("button", name="登入").click() | - |
| 5 | 驗證錯誤訊息 | expect(page.locator("#errorMsg")).to_have_text("帳號或密碼錯誤") | 顯示錯誤訊息 |

**預期結果**：停留在登入頁面，顯示錯誤訊息

---

## TC-003: [更多測試案例...]

================================================================================
                              確認事項
================================================================================

請確認以下項目：

測試 URL：[URL]
測試案例數：[N] 個
元素定位來源：從程式碼掃描取得（非猜測）

待您提供：
1. 測試帳號密碼（共 [N] 組）
2. 驗證碼取得方式（如有）

================================================================================

請提供上述帳號資訊，並回覆「OKOKYES」開始執行測試
如需調整測試案例，請告知要修改的項目

================================================================================
```

### 3.2 測試案例設計原則

根據開發的功能，自動規劃以下類型的測試：

| 測試類型 | 說明 | 範例 |
|---------|------|------|
| **正向測試** | 正確輸入，預期成功 | 正確帳密登入成功 |
| **負向測試** | 錯誤輸入，預期失敗並顯示錯誤 | 錯誤密碼登入失敗 |
| **邊界測試** | 邊界值輸入 | 密碼長度剛好 8 碼 |
| **空值測試** | 必填欄位為空 | 帳號為空點擊登入 |
| **權限測試** | 驗證權限控制 | 未登入存取需授權頁面 |

### 3.3 等待使用者確認

> **強制等待**：必須等待使用者提供帳號密碼並回覆 `OKOKYES` 才能執行測試。

- 收到帳號密碼 + `OKOKYES` → 進入 Step 4
- 收到其他回覆 → 詢問需要調整什麼
- **絕對禁止**在未收到 `OKOKYES` 前撰寫任何測試程式碼

---

## Step 4: 執行測試

> 只有在收到 `OKOKYES` 確認後，才能進入此步驟。

### 4.1 環境準備

```bash
# 安裝 Playwright（Python）
pip install playwright pytest-playwright
playwright install chromium

# 安裝 Playwright（Java Maven）
# 在 pom.xml 加入 com.microsoft.playwright 依賴
```

### 4.2 瀏覽器設定（強制）

> **再次提醒**：必須使用實體瀏覽器！詳細設定請參考本文件最上方「強制使用實體瀏覽器」章節。

```
+-------------------------------------------------------------+
|  禁止 headless    必須開啟真實瀏覽器視窗                      |
|  禁止背景執行      使用者要能看到測試過程                      |
+-------------------------------------------------------------+
```

**必要參數**：
- `headless=False`（強制開啟實體瀏覽器）
- `slow_mo=500`（放慢速度，方便觀察）
- `viewport={"width": 1920, "height": 1080}`（固定視窗大小）

**建議啟用（除錯用）**：
- `context.tracing.start(screenshots=True, snapshots=True)`（啟用 Trace 記錄）
- `record_video_dir="videos/"`（錄製測試影片）

### 4.3 撰寫測試程式碼

- 測試檔案放在 `tests/e2e/` 目錄
- **Playwright 自帶 auto-wait，不需手動等待元素**
- 使用 `expect()` 斷言（內建 auto-retry）
- **所有元素定位必須來自 Step 2 的掃描結果**
- 每行程式碼加繁體中文註解
- **禁止使用 `time.sleep()`**，Playwright 的 auto-wait 已自動處理等待

### 4.4 測試程式碼範本

```python
import re
import pytest
from playwright.sync_api import Page, expect

class TestLogin:
    """登入功能測試"""

    def test_login_success(self, page: Page):
        """TC-001: 正確帳密登入成功"""
        # 開啟登入頁面
        page.goto("http://localhost:8080/login")

        # 輸入帳號（元素來自 Step 2 掃描）
        page.locator("#username").fill("admin")

        # 輸入密碼
        page.locator("#password").fill("admin123")

        # 點擊登入按鈕
        page.get_by_role("button", name="登入").click()

        # 驗證登入成功：URL 包含 /dashboard
        expect(page).to_have_url(re.compile(r".*/dashboard"))

    def test_login_wrong_password(self, page: Page):
        """TC-002: 錯誤密碼登入失敗"""
        # 開啟登入頁面
        page.goto("http://localhost:8080/login")

        # 輸入帳號
        page.locator("#username").fill("admin")

        # 輸入錯誤密碼
        page.locator("#password").fill("wrongpassword")

        # 點擊登入按鈕
        page.get_by_role("button", name="登入").click()

        # 驗證錯誤訊息顯示
        expect(page.locator("#errorMsg")).to_have_text("帳號或密碼錯誤")
```

### 4.5 conftest.py 設定範本

```python
import pytest
from playwright.sync_api import sync_playwright

@pytest.fixture(scope="session")
def browser():
    """建立瀏覽器實例（整個測試 session 共用）"""
    with sync_playwright() as p:
        # 強制使用實體瀏覽器
        browser = p.chromium.launch(
            headless=False,     # 禁止 headless
            slow_mo=500         # 放慢速度方便觀察
        )
        yield browser
        browser.close()

@pytest.fixture(scope="function")
def context(browser):
    """每個測試建立獨立的瀏覽器上下文"""
    context = browser.new_context(
        viewport={"width": 1920, "height": 1080},    # 固定視窗大小
        record_video_dir="videos/"                     # 錄製影片
    )
    # 啟用 Trace（除錯用）
    context.tracing.start(screenshots=True, snapshots=True)
    yield context
    # 測試結束後儲存 Trace
    context.tracing.stop(path="traces/trace.zip")
    context.close()

@pytest.fixture(scope="function")
def page(context):
    """每個測試建立獨立的頁面"""
    page = context.new_page()
    yield page
    page.close()
```

### 4.6 執行測試

```bash
# Python + pytest
pytest tests/e2e/test_xxx.py -v --headed

# 若需要指定瀏覽器
pytest tests/e2e/test_xxx.py -v --browser chromium

# 查看 Trace（測試失敗時除錯用）
playwright show-trace traces/trace.zip
```

---

## 禁止事項（違反即為錯誤）

| 禁止行為 | 正確做法 |
|---------|---------|
| 跳過讀取專案 CLAUDE.md | Step 1.1 最優先讀取 CLAUDE.md |
| 不回顧開發內容就開始測試 | Step 1.2 先彙整開發資訊 |
| 未掃描程式碼就撰寫測試 | Step 2 完整掃描元素 |
| 未產出測試計畫就執行 | Step 3 產出計畫並等待確認 |
| 未收到 OKOKYES 就寫程式碼 | 等待明確的 OKOKYES |
| 猜測元素 ID | 從掃描結果取得真實定位 |
| 猜測測試網址 | 從 CLAUDE.md 或設定檔取得 |
| 使用 headless 模式 | 設定 headless=False |
| 使用 time.sleep() | 依賴 Playwright auto-wait |
| 省略 slow_mo 設定 | 設定 slow_mo=500 方便觀察 |
| CLAUDE.md 有帳號卻還問使用者 | 優先使用 CLAUDE.md 的帳號 |
| CLAUDE.md 沒帳號卻不問使用者 | 必須詢問並取得帳號密碼 |

---

## 測試帳號處理規範

> **禁止在測試程式碼中寫死帳號密碼並 commit**

### 帳號取得流程

```
讀取 CLAUDE.md → 有帳號？ → 是 → 直接使用
                    |
                   否
                    |
              詢問使用者提供
```

### 處理規則

| 情況 | 處理方式 |
|-----|---------|
| CLAUDE.md 有測試帳號 | 直接使用，不需詢問使用者 |
| CLAUDE.md 無測試帳號 | 在 Step 3 詢問使用者提供 |
| CLAUDE.md 有部分帳號 | 使用已有的，缺少的再詢問 |

### 安全規範

1. 測試帳號僅用於測試程式碼執行，不 commit 到版本控制
2. 若 CLAUDE.md 的帳號是開發環境專用，可直接使用
3. 執行完畢後提醒使用者帳密僅用於本次測試

---

## 完成通知

> **測試執行完成後，必須輸出：**

```
【測試執行完成】

================================================================================
                              測試結果摘要
================================================================================

測試 URL：[URL]
測試功能：[功能描述]
測試案例：[N] 個

| 狀態 | 數量 | 案例 |
|-----|------|------|
| 通過 | [N] | TC-001, TC-002 |
| 失敗 | [N] | TC-003 |
| 跳過 | [N] | - |

================================================================================
                              失敗案例明細（如有）
================================================================================

## TC-003: [案例名稱]
- 失敗步驟：步驟 4
- 失敗原因：[原因]
- 截圖：[路徑]
- Trace 檔：traces/trace.zip（可用 playwright show-trace 開啟）
- 錄影：videos/[檔名].webm
- 建議修正：[建議]

================================================================================
                              Skill 流程完成
================================================================================

分析階段 → 已完成
開發階段 → 已完成
CodeReview → 已完成
測試階段 → 已完成

完整的「分析 → 開發 → CodeReview → 測試」流程已全部完成！

================================================================================
```

### 測試失敗處理

若有測試失敗：
1. 列出失敗的測試案例與錯誤原因
2. 提供 Trace 檔路徑，方便使用者用 `playwright show-trace` 除錯
3. 提供錄影檔路徑（若有啟用）
4. 詢問使用者是否要修正問題
5. 若需修正，串接「開發」Skill
6. 修正後重新執行測試，直到全部通過
