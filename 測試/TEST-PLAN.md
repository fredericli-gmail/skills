# 測試規劃模板

標準化測試規劃文件模板（Playwright 版本）。

## 目錄

1. [快速測試規劃](#快速測試規劃)
2. [完整測試規劃](#完整測試規劃)
3. [測試案例模板](#測試案例模板)
4. [測試資料模板](#測試資料模板)

---

## 快速測試規劃

適用於單一功能的快速測試。

```markdown
# [功能名稱] 測試規劃

## 基本資訊
- 測試目標：[功能描述]
- 測試 URL：[http://...]
- 瀏覽器：Chromium（實體視窗，headless=False）
- 執行方式：pytest + playwright

## 元素定位（從掃描取得）

| 元素 | Playwright 定位 | 來源 |
|-----|----------------|------|
| [元素名] | page.locator("#id") | [檔案:行號] |
| [元素名] | page.get_by_role("button", name="xxx") | [檔案:行號] |

## 測試案例

| ID | 名稱 | 步驟 | 預期結果 |
|----|-----|-----|---------|
| TC01 | [名稱] | [步驟] | [結果] |

## 測試資料

| 欄位 | 有效值 | 無效值 |
|-----|-------|-------|
| [欄位] | [值] | [值] |
```

---

## 完整測試規劃

適用於模組或系統級測試。

```markdown
# [模組/系統名稱] 測試規劃書

版本：v1.0
建立日期：[日期]
測試負責人：[名稱]

---

## 1. 測試概述

### 1.1 測試目標
[描述本次測試的主要目標]

### 1.2 測試範圍

**包含：**
- [功能 1]
- [功能 2]

**不包含：**
- [排除項 1]

### 1.3 測試環境

| 項目 | 設定 |
|-----|-----|
| 測試 URL | http://localhost:8080 |
| 瀏覽器 | Chromium（實體視窗，headless=False，slow_mo=500） |
| 視窗大小 | 1920 x 1080 |
| 測試框架 | pytest + playwright |
| 作業系統 | [OS] |

---

## 2. 元素定位清單（掃描結果）

### 2.1 登入頁面 `/login`

| 元素名稱 | Playwright 定位 | 定位值 | 來源檔案 |
|---------|----------------|--------|---------|
| 帳號輸入框 | page.locator("#username") | id=username | login.html:15 |
| 密碼輸入框 | page.locator("#password") | id=password | login.html:20 |
| 登入按鈕 | page.get_by_role("button", name="登入") | role=button | login.html:25 |
| 錯誤訊息 | page.locator("#error-message") | id=error-message | login.html:30 |

### 2.2 [頁面名稱] `/path`

| 元素名稱 | Playwright 定位 | 定位值 | 來源檔案 |
|---------|----------------|--------|---------|
| [元素] | [定位方式] | [值] | [檔案] |

---

## 3. API 端點清單（掃描結果）

| 功能 | Method | Endpoint | 請求參數 | 回應 |
|-----|--------|----------|---------|-----|
| 登入 | POST | /api/auth/login | username, password | token |
| [功能] | [Method] | [Endpoint] | [參數] | [回應] |

---

## 4. 測試案例

### 4.1 登入功能

#### TC-LOGIN-001: 正確帳密登入成功

| 項目 | 內容 |
|-----|-----|
| 優先級 | P0 |
| 前置條件 | 1. 測試帳號已建立 2. 系統正常運作 |
| 測試資料 | 帳號: testuser, 密碼: Test@123 |

**測試步驟：**
1. `page.goto("http://localhost:8080/login")`
2. `page.locator("#username").fill("testuser")`
3. `page.locator("#password").fill("Test@123")`
4. `page.get_by_role("button", name="登入").click()`

**預期結果：**
- `expect(page).to_have_url(re.compile(r".*/dashboard"))`
- `expect(page.get_by_text("歡迎")).to_be_visible()`

---

#### TC-LOGIN-002: 錯誤密碼登入失敗

| 項目 | 內容 |
|-----|-----|
| 優先級 | P0 |
| 前置條件 | 系統正常運作 |
| 測試資料 | 帳號: testuser, 密碼: wrongpwd |

**測試步驟：**
1. `page.goto("http://localhost:8080/login")`
2. `page.locator("#username").fill("testuser")`
3. `page.locator("#password").fill("wrongpwd")`
4. `page.get_by_role("button", name="登入").click()`

**預期結果：**
- `expect(page).to_have_url(re.compile(r".*/login"))`
- `expect(page.locator("#error-message")).to_have_text("帳號或密碼錯誤")`

---

### 4.2 [功能名稱]

#### TC-[功能]-001: [案例名稱]

[同上格式]

---

## 5. 測試資料

### 5.1 測試帳號

| 角色 | 帳號 | 密碼 | 權限說明 |
|-----|-----|-----|---------|
| 系統管理員 | admin | Admin@123 | 所有權限 |
| 一般用戶 | user1 | User@123 | 基本權限 |
| 訪客 | guest | Guest@123 | 僅瀏覽 |

### 5.2 測試資料集

| 資料類型 | 有效值 | 邊界值 | 無效值 |
|---------|-------|-------|-------|
| 帳號 | user123 | a (1字元) | 空值、特殊字元 |
| 密碼 | Test@123 | 12345678 (8字元) | 123 (過短) |
| Email | test@example.com | - | test@, @example.com |

---

## 6. 風險與對策

| 風險 | 影響 | 對策 |
|-----|-----|-----|
| 元素定位變更 | 測試失敗 | 重新掃描程式碼更新定位 |
| 網路延遲 | 操作超時 | Playwright auto-wait 自動處理 |
| 測試資料被修改 | 測試結果不一致 | 每次測試使用獨立 context |
| 動畫干擾 | 元素點擊失敗 | 使用 reduced_motion 停用動畫 |

---

## 7. 除錯工具

| 工具 | 用途 | 使用方式 |
|-----|-----|---------|
| Trace Viewer | 回放測試過程 | `playwright show-trace trace.zip` |
| 錄影 | 觀看測試影片 | `record_video_dir="videos/"` |
| slow_mo | 放慢測試速度 | `slow_mo=500` |
| 截圖 | 失敗時截圖 | `page.screenshot(path="error.png")` |
```

---

## 測試案例模板

單一測試案例的詳細模板。

```markdown
## TC-[模組]-[編號]: [案例名稱]

### 基本資訊

| 項目 | 內容 |
|-----|-----|
| 測試類型 | 功能測試 / 整合測試 / E2E |
| 優先級 | P0 / P1 / P2 |
| 自動化 | 是 / 否 |

### 前置條件
1. [條件 1]
2. [條件 2]

### 測試資料

| 欄位 | 值 | 說明 |
|-----|---|-----|
| [欄位] | [值] | [說明] |

### 測試步驟（Playwright 指令）

| 步驟 | 操作 | Playwright 指令 |
|-----|-----|----------------|
| 1 | 開啟頁面 | page.goto("/login") |
| 2 | 輸入帳號 | page.locator("#username").fill("testuser") |
| 3 | 輸入密碼 | page.locator("#password").fill("Test@123") |
| 4 | 點擊登入 | page.get_by_role("button", name="登入").click() |

### 預期結果（Playwright 斷言）

| 驗證點 | Playwright 斷言 |
|-------|----------------|
| URL | expect(page).to_have_url(re.compile(r".*/dashboard")) |
| 歡迎訊息 | expect(page.get_by_text("歡迎")).to_be_visible() |
| [驗證點] | [斷言] |

### 實際結果

| 執行日期 | 結果 | 備註 |
|---------|-----|-----|
| [日期] | PASS / FAIL | [備註] |
```

---

## 測試資料模板

測試資料管理模板。

```markdown
# 測試資料清單

## 1. 帳號資料

### 1.1 系統帳號

```python
TEST_ACCOUNTS = {
    "admin": {
        "username": "admin",
        "password": "Admin@123",
        "role": "administrator",
        "permissions": ["read", "write", "delete", "admin"]
    },
    "user": {
        "username": "testuser",
        "password": "User@123",
        "role": "user",
        "permissions": ["read", "write"]
    },
    "guest": {
        "username": "guest",
        "password": "Guest@123",
        "role": "guest",
        "permissions": ["read"]
    }
}
```

### 1.2 無效帳號（用於負面測試）

```python
INVALID_ACCOUNTS = {
    "wrong_password": {
        "username": "testuser",
        "password": "wrongpassword",
        "expected_error": "帳號或密碼錯誤"
    },
    "nonexistent": {
        "username": "notexist",
        "password": "Test@123",
        "expected_error": "帳號不存在"
    },
    "empty_username": {
        "username": "",
        "password": "Test@123",
        "expected_error": "請輸入帳號"
    }
}
```

## 2. 表單資料

### 2.1 有效資料

```python
VALID_FORM_DATA = {
    "case_create": {
        "title": "測試案件標題",
        "description": "這是測試案件的描述內容",
        "priority": "high",
        "category": "bug",
        "assignee": "user1"
    }
}
```

### 2.2 邊界值資料

```python
BOUNDARY_DATA = {
    "title_min": "A",
    "title_max": "A" * 100,
    "description_empty": "",
    "description_max": "測試" * 500
}
```

### 2.3 無效資料

```python
INVALID_DATA = {
    "title_too_long": "A" * 101,
    "special_chars": "<script>alert('xss')</script>",
    "sql_injection": "'; DROP TABLE users; --"
}
```

## 3. 資料使用方式

```python
# conftest.py
import pytest

@pytest.fixture
def admin_account():
    return TEST_ACCOUNTS["admin"]

@pytest.fixture
def valid_form_data():
    return VALID_FORM_DATA["case_create"]

# test_login.py
def test_admin_login(page, admin_account):
    page.goto("http://localhost:8080/login")
    page.locator("#username").fill(admin_account["username"])
    page.locator("#password").fill(admin_account["password"])
    page.get_by_role("button", name="登入").click()
    expect(page).to_have_url(re.compile(r".*/dashboard"))
```
```
