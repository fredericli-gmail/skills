# 疑難排解指南

常見 Playwright 測試問題與解決方案。

## 目錄

1. [元素定位問題](#元素定位問題)
2. [元素互動問題](#元素互動問題)
3. [等待與超時問題](#等待與超時問題)
4. [瀏覽器問題](#瀏覽器問題)
5. [測試穩定性問題](#測試穩定性問題)
6. [Trace 除錯](#trace-除錯)

---

## 元素定位問題

### 問題：嚴格模式匹配到多個元素

**症狀**：
```
Error: strict mode violation: locator resolved to 2 elements
```

**原因**：Playwright 預設嚴格模式，locator 匹配到多個元素時拋錯

**解決方案**：

```python
# 1. 使用更精確的定位
page.locator("#unique-id").click()                           # 用 ID
page.locator("button.submit-btn").first.click()              # 取第一個
page.locator("button").filter(has_text="確認").click()       # 篩選

# 2. 使用 nth() 指定索引
page.locator(".item").nth(0).click()     # 第一個
page.locator(".item").nth(2).click()     # 第三個

# 3. 使用 first / last
page.locator(".item").first.click()
page.locator(".item").last.click()

# 4. 組合定位縮小範圍
page.locator("div.login-form").locator("#username").fill("admin")
```

### 問題：元素不存在

**症狀**：
```
TimeoutError: Timeout 30000ms exceeded.
waiting for locator("#nonexistent")
```

**解決方案**：

```python
# 1. 確認元素 ID 是否正確（重新掃描程式碼）
# 2. 確認元素是否在 iframe 中
frame = page.frame_locator("#my-iframe")
frame.locator("#element").click()

# 3. 確認頁面是否載入完成
page.wait_for_load_state("networkidle")

# 4. 檢查元素是否存在
if page.locator("#element").count() > 0:
    page.locator("#element").click()
```

### 問題：動態 ID

**症狀**：ID 每次不同，如 `element-abc123`

**解決方案**：

```python
# 1. 使用部分匹配
page.locator("[id^='element-']").click()       # 開頭匹配
page.locator("[id*='element']").click()        # 包含匹配

# 2. 使用 data-testid（建議前端加上）
page.get_by_test_id("submit-btn").click()

# 3. 使用文字或角色定位
page.get_by_role("button", name="提交").click()
page.get_by_text("提交").click()

# 4. 使用相對定位
page.locator("form").locator("button").click()
```

---

## 元素互動問題

### 問題：元素被遮擋無法點擊

**症狀**：
```
Error: element is not visible or is covered by another element
```

**解決方案**：

```python
# 1. 等待遮擋物消失
page.locator(".loading-overlay").wait_for(state="hidden")
page.locator("#button").click()

# 2. 滾動到元素
page.locator("#button").scroll_into_view_if_needed()
page.locator("#button").click()

# 3. 使用 force 強制點擊（跳過可操作性檢查）
page.locator("#button").click(force=True)

# 4. 使用 JavaScript 點擊
page.locator("#button").dispatch_event("click")

# 5. 關閉遮擋的彈窗
page.locator(".modal-close").click()
page.locator("#button").click()
```

### 問題：無法輸入文字

**解決方案**：

```python
element = page.locator("#input-field")

# 1. 使用 fill（清空後填入）
element.fill("新文字")

# 2. 先點擊再輸入
element.click()
element.fill("新文字")

# 3. 使用 type 逐字輸入
element.type("新文字", delay=50)

# 4. 使用 JavaScript
page.evaluate("document.querySelector('#input-field').value = '新文字'")

# 5. 先清空再輸入
element.clear()
element.type("新文字")

# 6. 觸發 input 事件
page.evaluate("""
    const el = document.querySelector('#input-field');
    el.value = '新文字';
    el.dispatchEvent(new Event('input', { bubbles: true }));
""")
```

### 問題：下拉選單無法選擇

**解決方案**：

```python
# 1. 標準 <select> 元素
page.locator("#dropdown").select_option("value1")
page.locator("#dropdown").select_option(label="選項一")

# 2. 自訂下拉選單（如 React Select、Ant Design）
# 先點擊觸發下拉
page.locator(".custom-select-trigger").click()
# 等待選項出現
page.locator(".custom-select-option").filter(has_text="選項一").click()

# 3. 使用鍵盤
page.locator(".custom-select-trigger").click()
page.keyboard.type("選項文字")
page.keyboard.press("Enter")
```

### 問題：檔案上傳

**解決方案**：

```python
# 1. 標準 file input
page.locator("input[type='file']").set_input_files("path/to/file.pdf")

# 2. 隱藏的 file input（透過按鈕觸發）
with page.expect_file_chooser() as fc_info:
    page.locator("#upload-btn").click()
file_chooser = fc_info.value
file_chooser.set_files("path/to/file.pdf")
```

---

## 等待與超時問題

### 問題：TimeoutError

**症狀**：
```
TimeoutError: Timeout 30000ms exceeded.
```

**解決方案**：

```python
# 1. 增加單一操作超時
page.locator("#slow-element").click(timeout=60000)    # 60 秒

# 2. 增加全域超時
page.set_default_timeout(60000)

# 3. 等待網路閒置
page.wait_for_load_state("networkidle")

# 4. 等待特定 API 完成
with page.expect_response("**/api/data") as response_info:
    page.locator("#load-btn").click()
response = response_info.value

# 5. 等待 Loading 消失
page.locator(".loading-spinner").wait_for(state="hidden", timeout=60000)
```

### 問題：頁面導航超時

**解決方案**：

```python
# 1. 設定導航超時
page.set_default_navigation_timeout(60000)

# 2. 指定等待條件
page.goto("http://example.com", wait_until="domcontentloaded")

# 3. 不等待完全載入
page.goto("http://example.com", wait_until="commit")
# 然後等待關鍵元素
page.locator("#main-content").wait_for(state="visible")
```

### 問題：AJAX 請求未完成

**解決方案**：

```python
# 1. 等待網路閒置
page.wait_for_load_state("networkidle")

# 2. 等待特定 API 回應
with page.expect_response(lambda resp: "/api/data" in resp.url) as resp_info:
    page.locator("#refresh").click()
response = resp_info.value
assert response.status == 200

# 3. 等待元素內容更新
expect(page.locator("#result")).not_to_have_text("載入中...")
expect(page.locator("#result")).to_have_text(re.compile(r".+"))
```

---

## 瀏覽器問題

### 問題：瀏覽器未安裝

**症狀**：
```
Error: Executable doesn't exist at /path/to/browser
```

**解決方案**：

```bash
# 安裝所有瀏覽器
playwright install

# 只安裝 Chromium
playwright install chromium

# 安裝系統依賴（Linux）
playwright install-deps
```

### 問題：瀏覽器崩潰

**解決方案**：

```python
# 1. 降低資源使用
browser = p.chromium.launch(
    headless=False,
    args=[
        "--disable-gpu",
        "--disable-extensions",
        "--disable-software-rasterizer"
    ]
)

# 2. 每個測試使用獨立 context
@pytest.fixture(scope="function")
def context(browser):
    context = browser.new_context()
    yield context
    context.close()
```

### 問題：SSL 憑證錯誤

**解決方案**：

```python
# 忽略 SSL 錯誤
context = browser.new_context(ignore_https_errors=True)
```

---

## 測試穩定性問題

### 問題：測試時有時無（Flaky Tests）

**解決方案**：

```python
# 1. 使用 expect 斷言（自帶 retry）
# 不好：直接取值比較
assert page.locator("#count").text_content() == "10"

# 好：使用 expect（自動重試直到通過或超時）
expect(page.locator("#count")).to_have_text("10")

# 2. 等待頁面穩定
page.wait_for_load_state("networkidle")

# 3. 使用 pytest-rerunfailures 重試
# pip install pytest-rerunfailures
# pytest --reruns 3 --reruns-delay 2

# 4. 測試間隔離
@pytest.fixture(scope="function")
def page(browser):
    context = browser.new_context()
    page = context.new_page()
    yield page
    context.close()    # 每個測試完整隔離
```

### 問題：測試間互相影響

**解決方案**：

```python
# 1. 每個測試使用獨立 context（最佳實踐）
@pytest.fixture(scope="function")
def clean_context(browser):
    context = browser.new_context(
        viewport={"width": 1920, "height": 1080}
    )
    yield context
    context.close()

# 2. 測試前清除狀態
@pytest.fixture(scope="function")
def page(clean_context):
    page = clean_context.new_page()
    yield page
    page.close()
```

### 問題：動畫造成測試不穩定

**解決方案**：

```python
# 1. 停用動畫
context = browser.new_context(
    reduced_motion="reduce"
)

# 2. 注入 CSS 停用動畫
page.add_style_tag(content="""
    *, *::before, *::after {
        animation-duration: 0s !important;
        transition-duration: 0s !important;
    }
""")
```

---

## Trace 除錯

### 測試失敗時自動產出 Trace

```python
# conftest.py
import pytest

@pytest.fixture(scope="function")
def traced_context(browser, request):
    context = browser.new_context(
        viewport={"width": 1920, "height": 1080}
    )
    context.tracing.start(screenshots=True, snapshots=True)
    yield context

    # 測試失敗時儲存 Trace
    if request.node.rep_call and request.node.rep_call.failed:
        trace_path = f"traces/{request.node.name}.zip"
        context.tracing.stop(path=trace_path)
        print(f"\nTrace 已儲存: {trace_path}")
    else:
        context.tracing.stop()

    context.close()

@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    rep = outcome.get_result()
    setattr(item, f"rep_{rep.when}", rep)
```

### 開啟 Trace Viewer

```bash
# 開啟 Trace Viewer（瀏覽器介面）
playwright show-trace traces/test_login.zip

# Trace Viewer 功能：
# - 查看每個操作的截圖
# - 查看 DOM 快照
# - 查看網路請求
# - 查看 Console 日誌
# - 時間軸回放
```

---

## 快速診斷清單

遇到問題時，依序檢查：

1. 元素定位是否正確？（重新掃描程式碼）
2. 是否匹配到多個元素？（使用 filter/nth 縮小範圍）
3. 元素是否在 iframe 中？（使用 frame_locator）
4. 元素是否被遮擋？（等待遮擋物消失或用 force）
5. 頁面是否載入完成？（wait_for_load_state）
6. 是否有 AJAX 進行中？（等待 networkidle）
7. 瀏覽器是否已安裝？（playwright install）
8. 是否產出 Trace？（playwright show-trace 分析）
