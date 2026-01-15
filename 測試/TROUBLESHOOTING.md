# 疑難排解指南

常見測試問題與解決方案。

## 目錄

1. [元素定位問題](#元素定位問題)
2. [元素互動問題](#元素互動問題)
3. [等待與超時問題](#等待與超時問題)
4. [瀏覽器問題](#瀏覽器問題)
5. [測試穩定性問題](#測試穩定性問題)

---

## 元素定位問題

### 問題：NoSuchElementException

**症狀**：
```
selenium.common.exceptions.NoSuchElementException: Message: no such element: Unable to locate element
```

**原因與解決方案**：

| 原因 | 解決方案 |
|-----|---------|
| 元素 ID 錯誤 | 重新掃描程式碼，確認正確的 ID |
| 元素尚未載入 | 使用顯式等待 `WebDriverWait` |
| 元素在 iframe 中 | 先切換到 iframe |
| 元素動態生成 | 等待元素出現後再定位 |
| 頁面尚未載入完成 | 等待頁面載入完成 |

**解決程式碼**：

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# 1. 使用顯式等待
element = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.ID, "element-id"))
)

# 2. 檢查是否在 iframe 中
iframes = driver.find_elements(By.TAG_NAME, "iframe")
for iframe in iframes:
    driver.switch_to.frame(iframe)
    try:
        element = driver.find_element(By.ID, "element-id")
        break
    except:
        driver.switch_to.default_content()

# 3. 等待頁面載入
WebDriverWait(driver, 10).until(
    lambda d: d.execute_script("return document.readyState") == "complete"
)
```

### 問題：StaleElementReferenceException

**症狀**：
```
selenium.common.exceptions.StaleElementReferenceException: stale element reference
```

**原因**：頁面已更新，之前定位的元素已失效

**解決方案**：

```python
# 1. 重新定位元素
def click_with_retry(driver, by, value, max_attempts=3):
    for attempt in range(max_attempts):
        try:
            element = driver.find_element(by, value)
            element.click()
            return
        except StaleElementReferenceException:
            time.sleep(0.5)
    raise Exception(f"無法點擊元素 {by}={value}")

# 2. 使用等待確保元素穩定
element = WebDriverWait(driver, 10).until(
    EC.element_to_be_clickable((By.ID, "button"))
)
element.click()
```

### 問題：動態 ID

**症狀**：ID 每次都不同，如 `element-12345`

**解決方案**：

```python
# 1. 使用部分匹配
element = driver.find_element(By.CSS_SELECTOR, "[id^='element-']")  # 開頭
element = driver.find_element(By.CSS_SELECTOR, "[id*='element']")   # 包含

# 2. 使用其他穩定屬性
element = driver.find_element(By.CSS_SELECTOR, "[data-testid='submit-btn']")
element = driver.find_element(By.NAME, "username")

# 3. 使用 XPath 文字匹配
element = driver.find_element(By.XPATH, "//button[contains(text(), '提交')]")
```

---

## 元素互動問題

### 問題：ElementNotInteractableException

**症狀**：
```
selenium.common.exceptions.ElementNotInteractableException: element not interactable
```

**原因與解決方案**：

| 原因 | 解決方案 |
|-----|---------|
| 元素被遮擋 | 滾動到元素或關閉遮擋物 |
| 元素不可見 | 等待元素可見 |
| 元素禁用 | 等待元素可用 |
| 需要先懸停 | 使用 ActionChains 懸停 |

**解決程式碼**：

```python
from selenium.webdriver.common.action_chains import ActionChains

# 1. 滾動到元素
element = driver.find_element(By.ID, "button")
driver.execute_script("arguments[0].scrollIntoView(true);", element)
time.sleep(0.5)
element.click()

# 2. 等待元素可點擊
element = WebDriverWait(driver, 10).until(
    EC.element_to_be_clickable((By.ID, "button"))
)
element.click()

# 3. 使用 JavaScript 點擊
driver.execute_script("arguments[0].click();", element)

# 4. 使用 ActionChains
actions = ActionChains(driver)
actions.move_to_element(element).click().perform()

# 5. 先懸停再點擊
menu = driver.find_element(By.ID, "menu")
actions = ActionChains(driver)
actions.move_to_element(menu).perform()
time.sleep(0.5)
submenu = driver.find_element(By.ID, "submenu-item")
submenu.click()
```

### 問題：無法輸入文字

**解決方案**：

```python
element = driver.find_element(By.ID, "input-field")

# 1. 先清空再輸入
element.clear()
element.send_keys("文字")

# 2. 使用 JavaScript
driver.execute_script("arguments[0].value = '文字';", element)

# 3. 觸發 input 事件
driver.execute_script("""
    arguments[0].value = '文字';
    arguments[0].dispatchEvent(new Event('input', { bubbles: true }));
""", element)

# 4. 先點擊再輸入
element.click()
element.clear()
element.send_keys("文字")
```

### 問題：下拉選單無法選擇

**解決方案**：

```python
from selenium.webdriver.support.ui import Select

# 1. 標準 select 元素
select = Select(driver.find_element(By.ID, "dropdown"))
select.select_by_value("option1")

# 2. 自訂下拉選單（如 React Select）
# 先點擊觸發下拉
dropdown = driver.find_element(By.CSS_SELECTOR, ".react-select__control")
dropdown.click()

# 等待選項出現
WebDriverWait(driver, 5).until(
    EC.presence_of_element_located((By.CSS_SELECTOR, ".react-select__option"))
)

# 點擊選項
option = driver.find_element(By.XPATH, "//div[contains(@class, 'option') and text()='選項']")
option.click()

# 3. 使用鍵盤
dropdown.send_keys("選項文字")
dropdown.send_keys(Keys.ENTER)
```

---

## 等待與超時問題

### 問題：TimeoutException

**症狀**：
```
selenium.common.exceptions.TimeoutException: Message: timeout
```

**解決方案**：

```python
# 1. 增加等待時間
element = WebDriverWait(driver, 30).until(  # 增加到 30 秒
    EC.presence_of_element_located((By.ID, "slow-loading"))
)

# 2. 檢查網路請求
# 在開發者工具查看是否有 API 請求卡住

# 3. 分段等待
def wait_for_ajax(driver, timeout=30):
    """等待 AJAX 請求完成"""
    WebDriverWait(driver, timeout).until(
        lambda d: d.execute_script("return jQuery.active == 0")
    )

# 4. 等待 Loading 消失
WebDriverWait(driver, 30).until(
    EC.invisibility_of_element_located((By.CSS_SELECTOR, ".loading-spinner"))
)
```

### 問題：頁面載入過慢

**解決方案**：

```python
# 1. 設定頁面載入超時
driver.set_page_load_timeout(60)

# 2. 等待特定元素而非整頁
driver.get("http://example.com")
WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.ID, "main-content"))
)

# 3. 停止載入並繼續
try:
    driver.get("http://example.com")
except TimeoutException:
    driver.execute_script("window.stop();")
    # 繼續測試關鍵元素
```

---

## 瀏覽器問題

### 問題：ChromeDriver 版本不符

**症狀**：
```
SessionNotCreatedException: session not created: This version of ChromeDriver only supports Chrome version XX
```

**解決方案**：

```python
# 使用 webdriver-manager 自動管理
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.service import Service

service = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=options)
```

```bash
# 或手動下載對應版本
# 1. 檢查 Chrome 版本：chrome://version
# 2. 下載對應 ChromeDriver：https://chromedriver.chromium.org/downloads
```

### 問題：瀏覽器崩潰

**解決方案**：

```python
options = webdriver.ChromeOptions()
options.add_argument('--disable-gpu')
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')
options.add_argument('--disable-extensions')
options.add_argument('--disable-software-rasterizer')

# 增加記憶體
options.add_argument('--js-flags="--max-old-space-size=4096"')
```

### 問題：無法開啟瀏覽器

**檢查清單**：

1. ChromeDriver 是否在 PATH 中
2. Chrome 瀏覽器是否已安裝
3. 權限是否足夠
4. 是否有其他 ChromeDriver 進程佔用

```bash
# 清除殘留進程
pkill -f chromedriver
pkill -f chrome
```

---

## 測試穩定性問題

### 問題：測試時過時失敗（Flaky Tests）

**解決方案**：

```python
# 1. 使用重試機制
import pytest

@pytest.mark.flaky(reruns=3, reruns_delay=2)
def test_unstable_feature():
    # 測試程式碼
    pass

# 2. 確保每次測試狀態一致
@pytest.fixture(autouse=True)
def reset_state(driver):
    driver.delete_all_cookies()
    yield
    # 清理測試資料

# 3. 使用顯式等待而非 time.sleep
# 不好
time.sleep(3)
element = driver.find_element(By.ID, "button")

# 好
element = WebDriverWait(driver, 10).until(
    EC.element_to_be_clickable((By.ID, "button"))
)

# 4. 加入適當的同步點
def wait_for_page_stable(driver, timeout=10):
    """等待頁面穩定（無 DOM 變化）"""
    old_page = driver.find_element(By.TAG_NAME, "html").get_attribute("innerHTML")
    time.sleep(0.5)
    
    for _ in range(timeout * 2):
        new_page = driver.find_element(By.TAG_NAME, "html").get_attribute("innerHTML")
        if new_page == old_page:
            return
        old_page = new_page
        time.sleep(0.5)
```

### 問題：測試間互相影響

**解決方案**：

```python
import pytest

class TestFeature:
    @pytest.fixture(autouse=True)
    def setup_teardown(self, driver):
        # Setup: 測試前準備
        driver.get("http://example.com")
        driver.delete_all_cookies()
        
        yield
        
        # Teardown: 測試後清理
        driver.delete_all_cookies()
        
    def test_case_1(self, driver):
        # 獨立的測試
        pass
    
    def test_case_2(self, driver):
        # 獨立的測試
        pass
```

### 問題：截圖與日誌

**在失敗時自動截圖**：

```python
import pytest
from datetime import datetime

@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    report = outcome.get_result()
    
    if report.when == "call" and report.failed:
        driver = item.funcargs.get("driver")
        if driver:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            filename = f"screenshots/{item.name}_{timestamp}.png"
            driver.save_screenshot(filename)
            print(f"\n截圖已儲存: {filename}")
```

---

## 快速診斷清單

遇到問題時，依序檢查：

1. ✅ 元素 ID/selector 是否正確？（重新掃描程式碼）
2. ✅ 元素是否已載入？（加入顯式等待）
3. ✅ 元素是否在 iframe 中？（切換 frame）
4. ✅ 元素是否被遮擋？（滾動或關閉遮擋物）
5. ✅ 頁面是否完全載入？（等待載入完成）
6. ✅ 是否有 AJAX 請求進行中？（等待請求完成）
7. ✅ ChromeDriver 版本是否相符？（使用 webdriver-manager）
8. ✅ 瀏覽器是否正常啟動？（檢查錯誤日誌）
