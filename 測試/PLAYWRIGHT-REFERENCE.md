# Playwright 操作參考

完整的 Playwright 操作方法參考（Python 同步 API）。

## 目錄

1. [瀏覽器啟動](#瀏覽器啟動)
2. [元素定位](#元素定位)
3. [元素操作](#元素操作)
4. [等待機制](#等待機制)
5. [斷言方法](#斷言方法)
6. [頁面操作](#頁面操作)
7. [進階操作](#進階操作)
8. [Trace 與錄影](#trace-與錄影)

---

## 瀏覽器啟動

### 標準設定（實體瀏覽器）

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    # 啟動實體瀏覽器（禁止 headless）
    browser = p.chromium.launch(
        headless=False,       # 強制開啟實體瀏覽器
        slow_mo=500           # 每步間隔 500ms
    )

    # 建立瀏覽器上下文
    context = browser.new_context(
        viewport={"width": 1920, "height": 1080}
    )

    # 建立頁面
    page = context.new_page()
    page.goto("http://localhost:8080")

    # 測試完成後關閉
    browser.close()
```

### pytest + playwright 設定

```python
# conftest.py
import pytest
from playwright.sync_api import sync_playwright

@pytest.fixture(scope="session")
def browser():
    with sync_playwright() as p:
        browser = p.chromium.launch(
            headless=False,
            slow_mo=500
        )
        yield browser
        browser.close()

@pytest.fixture(scope="function")
def context(browser):
    context = browser.new_context(
        viewport={"width": 1920, "height": 1080}
    )
    yield context
    context.close()

@pytest.fixture(scope="function")
def page(context):
    page = context.new_page()
    yield page
    page.close()
```

### 多瀏覽器支援

```python
# Chromium
browser = p.chromium.launch(headless=False)

# Firefox
browser = p.firefox.launch(headless=False)

# WebKit (Safari)
browser = p.webkit.launch(headless=False)
```

---

## 元素定位

### 推薦定位方法（依優先順序）

```python
# 1. 依角色定位（最推薦，語意化）
page.get_by_role("button", name="登入")
page.get_by_role("textbox", name="帳號")
page.get_by_role("link", name="忘記密碼")
page.get_by_role("heading", name="歡迎")
page.get_by_role("checkbox", name="記住我")
page.get_by_role("combobox", name="城市")

# 2. 依 test-id 定位（測試專用）
page.get_by_test_id("submit-btn")
page.get_by_test_id("username-input")

# 3. 依文字定位
page.get_by_text("登入成功")
page.get_by_text("登入成功", exact=True)    # 完全匹配

# 4. 依 label 定位（表單欄位）
page.get_by_label("帳號")
page.get_by_label("密碼")

# 5. 依 placeholder 定位
page.get_by_placeholder("請輸入帳號")
page.get_by_placeholder("請輸入密碼")

# 6. 依 alt text 定位（圖片）
page.get_by_alt_text("公司 Logo")

# 7. 依 title 定位
page.get_by_title("說明")
```

### CSS / XPath 定位（備用）

```python
# CSS Selector
page.locator("#username")                    # ID
page.locator(".login-form")                  # Class
page.locator("input[name='email']")          # 屬性
page.locator("div.container > input")        # 子元素
page.locator("[data-testid='submit']")       # data 屬性

# XPath
page.locator("//button[@type='submit']")
page.locator("//div[contains(text(), '歡迎')]")

# 組合定位
page.locator("form").locator("#username")    # 先找 form 再找 #username
page.locator(".item").nth(0)                 # 第一個 .item
page.locator(".item").first                  # 第一個
page.locator(".item").last                   # 最後一個
```

### 篩選定位

```python
# 依文字篩選
page.locator("button").filter(has_text="確認")

# 依子元素篩選
page.locator("div.card").filter(has=page.locator("h3", has_text="標題"))

# 依不包含篩選
page.locator("div.card").filter(has_not_text="已刪除")
```

### 查找多個元素

```python
# 取得所有符合的元素
items = page.locator(".list-item")

# 取得數量
count = items.count()

# 遍歷
for i in range(items.count()):
    print(items.nth(i).text_content())

# 取得所有文字
texts = items.all_text_contents()
```

---

## 元素操作

### 基本操作

```python
# 點擊
page.locator("#button").click()
page.locator("#button").dblclick()             # 雙擊
page.locator("#button").click(button="right")  # 右鍵

# 輸入文字（清空後填入）
page.locator("#username").fill("admin")

# 逐字輸入（模擬真實打字）
page.locator("#search").type("關鍵字", delay=100)

# 清空
page.locator("#username").clear()

# 取得文字
text = page.locator("#message").text_content()       # 含子元素文字
text = page.locator("#message").inner_text()          # 可見文字

# 取得屬性
value = page.locator("#input").get_attribute("value")
href = page.locator("a").get_attribute("href")

# 取得 input 的值
value = page.locator("#input").input_value()
```

### 表單操作

```python
# 下拉選單（<select>）
page.locator("#city").select_option("taipei")                  # 依 value
page.locator("#city").select_option(label="台北市")             # 依文字
page.locator("#city").select_option(index=0)                    # 依索引
page.locator("#city").select_option(["taipei", "taichung"])     # 多選

# Checkbox
page.locator("#agree").check()                # 勾選
page.locator("#agree").uncheck()              # 取消勾選
page.locator("#agree").set_checked(True)      # 設定狀態

# Radio Button
page.locator("input[value='option1']").check()

# 檔案上傳
page.locator("#file-upload").set_input_files("path/to/file.pdf")
page.locator("#file-upload").set_input_files(["file1.pdf", "file2.pdf"])  # 多檔
page.locator("#file-upload").set_input_files([])                          # 清除

# 表單提交
page.locator("form").locator("button[type='submit']").click()
```

### 鍵盤操作

```python
# 按鍵
page.keyboard.press("Enter")
page.keyboard.press("Tab")
page.keyboard.press("Escape")
page.keyboard.press("Backspace")

# 組合鍵
page.keyboard.press("Control+a")    # 全選
page.keyboard.press("Control+c")    # 複製
page.keyboard.press("Control+v")    # 貼上
page.keyboard.press("Meta+a")       # macOS 全選

# 在元素上按鍵
page.locator("#search").press("Enter")
```

### 滑鼠操作

```python
# 懸停
page.locator("#menu").hover()

# 拖放
page.locator("#source").drag_to(page.locator("#target"))

# 精確滑鼠操作
page.mouse.move(100, 200)
page.mouse.click(100, 200)
page.mouse.dblclick(100, 200)
```

### 元素狀態

```python
# 檢查狀態
page.locator("#button").is_visible()      # 是否可見
page.locator("#button").is_enabled()      # 是否可用
page.locator("#button").is_disabled()     # 是否禁用
page.locator("#checkbox").is_checked()    # 是否勾選
page.locator("#button").is_hidden()       # 是否隱藏
page.locator("#button").is_editable()     # 是否可編輯
```

---

## 等待機制

### Auto-Wait（自動等待）

Playwright 的所有操作（click、fill、check 等）都自帶 auto-wait，會自動等待元素：
1. 出現在 DOM 中
2. 可見
3. 穩定（無動畫）
4. 可接收事件（未被其他元素遮擋）
5. 可用（非 disabled）

**因此，大多數情況不需要手動等待。**

### 手動等待（特殊場景才使用）

```python
# 等待元素出現
page.locator("#result").wait_for(state="visible")
page.locator("#loading").wait_for(state="hidden")
page.locator("#dynamic").wait_for(state="attached")

# 等待 URL 變化
page.wait_for_url("**/dashboard")
page.wait_for_url(re.compile(r".*/dashboard"))

# 等待載入狀態
page.wait_for_load_state("networkidle")    # 網路閒置
page.wait_for_load_state("domcontentloaded")
page.wait_for_load_state("load")

# 等待網路請求完成
with page.expect_response("**/api/data") as response_info:
    page.locator("#load-btn").click()
response = response_info.value

# 等待特定條件
page.wait_for_function("document.querySelector('#count').textContent === '10'")

# 設定全域超時
page.set_default_timeout(30000)    # 30 秒
```

---

## 斷言方法

### expect 斷言（推薦，自帶 auto-retry）

```python
from playwright.sync_api import expect
import re

# 頁面斷言
expect(page).to_have_url("http://example.com/dashboard")
expect(page).to_have_url(re.compile(r".*/dashboard"))
expect(page).to_have_title("首頁")
expect(page).to_have_title(re.compile(r"首頁.*"))

# 元素文字斷言
expect(page.locator("#message")).to_have_text("操作成功")
expect(page.locator("#message")).to_have_text(re.compile(r"操作.*"))
expect(page.locator("#message")).to_contain_text("成功")

# 元素可見性斷言
expect(page.locator("#modal")).to_be_visible()
expect(page.locator("#loading")).to_be_hidden()
expect(page.locator("#loading")).not_to_be_visible()

# 元素狀態斷言
expect(page.locator("#submit")).to_be_enabled()
expect(page.locator("#submit")).to_be_disabled()
expect(page.locator("#agree")).to_be_checked()
expect(page.locator("#agree")).not_to_be_checked()
expect(page.locator("#input")).to_be_editable()
expect(page.locator("#input")).to_be_empty()
expect(page.locator("#input")).to_be_focused()

# 元素屬性斷言
expect(page.locator("#input")).to_have_attribute("type", "text")
expect(page.locator("#input")).to_have_value("admin")
expect(page.locator("#input")).to_have_id("username")
expect(page.locator("#div")).to_have_class(re.compile(r".*active.*"))
expect(page.locator("#div")).to_have_css("color", "rgb(255, 0, 0)")

# 元素數量斷言
expect(page.locator(".item")).to_have_count(5)

# 多個文字斷言
expect(page.locator(".item")).to_have_text(["項目1", "項目2", "項目3"])
```

### 否定斷言

```python
# 在任何斷言前加 not_to
expect(page.locator("#error")).not_to_be_visible()
expect(page.locator("#input")).not_to_have_value("old")
expect(page).not_to_have_url(re.compile(r".*/login"))
```

### 自訂超時

```python
# 單一斷言設定超時
expect(page.locator("#slow-load")).to_be_visible(timeout=30000)
```

---

## 頁面操作

### 導航

```python
# 開啟 URL
page.goto("http://example.com")
page.goto("http://example.com", wait_until="networkidle")

# 重新整理
page.reload()

# 上一頁 / 下一頁
page.go_back()
page.go_forward()

# 取得當前 URL
url = page.url

# 取得 title
title = page.title()
```

### 多分頁

```python
# 開新分頁
new_page = context.new_page()
new_page.goto("http://example.com")

# 等待新分頁彈出
with context.expect_page() as new_page_info:
    page.locator("a[target='_blank']").click()
new_page = new_page_info.value
new_page.wait_for_load_state()

# 取得所有分頁
pages = context.pages

# 關閉分頁
new_page.close()
```

### iframe 操作

```python
# 依 id 或 name 定位 iframe
frame = page.frame_locator("#iframe-id")
frame.locator("#button").click()

# 依 CSS 定位 iframe
frame = page.frame_locator("iframe.main")
frame.locator("#input").fill("文字")

# 巢狀 iframe
inner_frame = page.frame_locator("#outer").frame_locator("#inner")
inner_frame.locator("#element").click()
```

### Dialog 處理（alert / confirm / prompt）

```python
# 自動接受 alert
page.on("dialog", lambda dialog: dialog.accept())

# 自動取消 confirm
page.on("dialog", lambda dialog: dialog.dismiss())

# 處理 prompt
def handle_dialog(dialog):
    print(dialog.message)
    dialog.accept("輸入值")

page.on("dialog", handle_dialog)

# 一次性處理
with page.expect_event("dialog") as dialog_info:
    page.locator("#trigger-alert").click()
dialog = dialog_info.value
dialog.accept()
```

### 截圖

```python
# 整頁截圖
page.screenshot(path="screenshot.png")

# 全頁截圖（含滾動）
page.screenshot(path="fullpage.png", full_page=True)

# 元素截圖
page.locator("#specific-element").screenshot(path="element.png")

# 取得 bytes
png_bytes = page.screenshot()
```

### JavaScript 執行

```python
# 執行 JavaScript
page.evaluate("alert('Hello')")

# 取得返回值
title = page.evaluate("document.title")

# 傳遞參數
page.evaluate("selector => document.querySelector(selector).remove()", "#element")

# 操作元素
element = page.locator("#button")
page.evaluate("el => el.scrollIntoView()", element.element_handle())
```

### Cookie 操作

```python
# 取得所有 cookies
cookies = context.cookies()

# 取得特定域名 cookies
cookies = context.cookies(["http://example.com"])

# 新增 cookie
context.add_cookies([{
    "name": "session",
    "value": "abc123",
    "domain": "example.com",
    "path": "/"
}])

# 清除 cookies
context.clear_cookies()
```

---

## 進階操作

### 網路攔截

```python
# 攔截 API 請求，回傳模擬資料
def handle_route(route):
    route.fulfill(
        status=200,
        content_type="application/json",
        body='{"status": "ok"}'
    )

page.route("**/api/data", handle_route)

# 攔截並修改請求
def modify_request(route):
    headers = route.request.headers
    headers["Authorization"] = "Bearer token123"
    route.continue_(headers=headers)

page.route("**/api/**", modify_request)

# 中止請求（例：阻擋圖片載入加速測試）
page.route("**/*.{png,jpg,jpeg}", lambda route: route.abort())

# 等待特定 API 回應
with page.expect_response("**/api/login") as response_info:
    page.locator("#login-btn").click()
response = response_info.value
assert response.status == 200
```

### 本地儲存

```python
# 儲存登入狀態（避免重複登入）
context.storage_state(path="auth-state.json")

# 載入登入狀態
context = browser.new_context(storage_state="auth-state.json")
```

### 模擬裝置

```python
# 模擬手機
iphone = p.devices["iPhone 13"]
context = browser.new_context(**iphone)

# 模擬地理位置
context = browser.new_context(
    geolocation={"latitude": 25.0330, "longitude": 121.5654},
    permissions=["geolocation"]
)

# 模擬時區
context = browser.new_context(timezone_id="Asia/Taipei")
```

---

## Trace 與錄影

### 啟用 Trace（除錯利器）

```python
# 開始記錄 Trace
context.tracing.start(
    screenshots=True,     # 每個操作截圖
    snapshots=True,       # DOM 快照
    sources=True          # 原始碼
)

# 執行測試...
page.goto("http://example.com")
page.locator("#button").click()

# 停止並儲存 Trace
context.tracing.stop(path="traces/trace.zip")
```

```bash
# 開啟 Trace Viewer 分析測試過程
playwright show-trace traces/trace.zip
```

### 啟用錄影

```python
# 在 context 建立時啟用錄影
context = browser.new_context(
    record_video_dir="videos/",
    record_video_size={"width": 1280, "height": 720}
)

page = context.new_page()
# 執行測試...

# 必須關閉 context 才會儲存影片
context.close()

# 取得影片路徑
video_path = page.video.path()
```

### pytest 自動記錄失敗

```python
# conftest.py
import pytest

@pytest.fixture(scope="function")
def context_with_trace(browser):
    context = browser.new_context(
        viewport={"width": 1920, "height": 1080}
    )
    context.tracing.start(screenshots=True, snapshots=True)
    yield context
    context.close()

@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    report = outcome.get_result()

    if report.when == "call" and report.failed:
        context = item.funcargs.get("context_with_trace")
        if context:
            context.tracing.stop(path=f"traces/{item.name}.zip")
            print(f"\nTrace 已儲存: traces/{item.name}.zip")
            print(f"開啟方式: playwright show-trace traces/{item.name}.zip")
```
