# 測試規劃模板

標準化測試規劃文件模板。

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
- 瀏覽器：Chrome（實體視窗）
- 執行方式：pytest

## 元素定位（從掃描取得）

| 元素 | 定位方式 | 定位值 |
|-----|---------|--------|
| [元素名] | id | [value] |

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
| 測試 URL | http://localhost:3000 |
| 瀏覽器 | Chrome（實體視窗，禁止 headless） |
| 解析度 | 1920 x 1080 |
| 作業系統 | [OS] |

---

## 2. 元素定位清單（掃描結果）

### 2.1 登入頁面 `/login`

| 元素名稱 | 定位方式 | 定位值 | 來源檔案 |
|---------|---------|--------|---------|
| 帳號輸入框 | id | username | Login.tsx:15 |
| 密碼輸入框 | id | password | Login.tsx:20 |
| 登入按鈕 | id | login-btn | Login.tsx:25 |
| 錯誤訊息 | id | error-message | Login.tsx:30 |

### 2.2 [頁面名稱] `/path`

| 元素名稱 | 定位方式 | 定位值 | 來源檔案 |
|---------|---------|--------|---------|
| [元素] | [方式] | [值] | [檔案] |

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
1. 開啟登入頁面 `/login`
2. 在帳號欄位（id=username）輸入 `testuser`
3. 在密碼欄位（id=password）輸入 `Test@123`
4. 點擊登入按鈕（id=login-btn）

**預期結果：**
- 頁面跳轉至 `/dashboard`
- 顯示歡迎訊息
- 頁面右上角顯示用戶名稱

---

#### TC-LOGIN-002: 錯誤密碼登入失敗

| 項目 | 內容 |
|-----|-----|
| 優先級 | P0 |
| 前置條件 | 系統正常運作 |
| 測試資料 | 帳號: testuser, 密碼: wrongpwd |

**測試步驟：**
1. 開啟登入頁面 `/login`
2. 在帳號欄位輸入 `testuser`
3. 在密碼欄位輸入 `wrongpwd`
4. 點擊登入按鈕

**預期結果：**
- 停留在登入頁面
- 顯示錯誤訊息（id=error-message）：「帳號或密碼錯誤」
- 密碼欄位清空

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
| 元素 ID 變更 | 測試失敗 | 重新掃描程式碼更新定位 |
| 網路延遲 | 元素載入超時 | 增加等待時間、使用顯式等待 |
| 測試資料被修改 | 測試結果不一致 | 每次測試前重置資料 |

---

## 7. 執行計畫

| 階段 | 內容 | 預估時間 |
|-----|-----|---------|
| 環境準備 | 安裝套件、設定環境 | 30 分鐘 |
| 冒煙測試 | 執行 P0 案例 | 1 小時 |
| 完整測試 | 執行所有案例 | 4 小時 |
| 結果分析 | 分析失敗案例、撰寫報告 | 1 小時 |
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
| 預估時間 | X 分鐘 |

### 前置條件
1. [條件 1]
2. [條件 2]

### 測試資料

| 欄位 | 值 | 說明 |
|-----|---|-----|
| [欄位] | [值] | [說明] |

### 測試步驟

| 步驟 | 操作 | 元素定位 | 輸入值 |
|-----|-----|---------|-------|
| 1 | 開啟頁面 | - | /login |
| 2 | 輸入帳號 | id=username | testuser |
| 3 | 輸入密碼 | id=password | Test@123 |
| 4 | 點擊登入 | id=login-btn | - |

### 預期結果

| 驗證點 | 預期值 |
|-------|-------|
| URL | 包含 /dashboard |
| 元素 (id=welcome) | 顯示「歡迎」|
| [驗證點] | [預期值] |

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
    "title_min": "A",  # 最小長度
    "title_max": "A" * 100,  # 最大長度
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
def test_admin_login(admin_account):
    # 使用 fixture 提供的測試資料
    driver.find_element(By.ID, "username").send_keys(admin_account["username"])
    driver.find_element(By.ID, "password").send_keys(admin_account["password"])
```
```
