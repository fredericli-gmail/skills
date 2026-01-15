# Selenium 操作參考

完整的 Selenium 操作方法參考。

## 目錄

1. [Driver 設定](#driver-設定)
2. [元素定位](#元素定位)
3. [元素操作](#元素操作)
4. [等待策略](#等待策略)
5. [瀏覽器操作](#瀏覽器操作)
6. [進階操作](#進階操作)
7. [斷言方法](#斷言方法)

---

## Driver 設定

### Chrome 實體瀏覽器（標準設定）

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service

def create_chrome_driver():
    """建立 Chrome 實體瀏覽器"""
    options = Options()
    
    # ✅ 推薦設定
    options.add_argument('--start-maximized')
    options.add_argument('--disable-gpu')
    options.add_argument('--no-sandbox')
    options.add_argument('--disable-dev-shm-usage')
    options.add_argument('--disable-extensions')
    options.add_argument('--disable-popup-blocking')
    
    # 設定視窗大小（替代 maximized）
    # options.add_argument('--window-size=1920,1080')
    
    # ❌ 禁止設定
    # options.add_argument('--headless')
    
    driver = webdriver.Chrome(options=options)
    driver.implicitly_wait(10)
    return driver
```

### 使用 WebDriver Manager

```python
from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.service import Service

def create_driver_with_manager():
    """自動管理 ChromeDriver 版本"""
    options = webdriver.ChromeOptions()
    options.add_argument('--start-maximized')
    
    service = Service(ChromeDriverManager().install())
    driver = webdriver.Chrome(service=service, options=options)
    return driver
```

### Firefox 設定

```python
from selenium import webdriver
from selenium.webdriver.firefox.options import Options

def create_firefox_driver():
    options = Options()
    # options.add_argument('--headless')  # 禁止
    
    driver = webdriver.Firefox(options=options)
    driver.maximize_window()
    return driver
```

---

## 元素定位

### 定位方法優先順序

```python
from selenium.webdriver.common.by import By

# 1. ID（最穩定）
element = driver.find_element(By.ID, "element-id")

# 2. data-testid（測試專用）
element = driver.find_element(By.CSS_SELECTOR, "[data-testid='element']")

# 3. Name
element = driver.find_element(By.NAME, "field-name")

# 4. CSS Selector
element = driver.find_element(By.CSS_SELECTOR, ".class-name")
element = driver.find_element(By.CSS_SELECTOR, "#id-name")
element = driver.find_element(By.CSS_SELECTOR, "div.class > input")

# 5. XPath（最後手段）
element = driver.find_element(By.XPATH, "//button[@type='submit']")
element = driver.find_element(By.XPATH, "//div[contains(text(), '文字')]")

# 6. 其他
element = driver.find_element(By.CLASS_NAME, "class-name")
element = driver.find_element(By.TAG_NAME, "button")
element = driver.find_element(By.LINK_TEXT, "連結文字")
element = driver.find_element(By.PARTIAL_LINK_TEXT, "部分文字")
```

### 查找多個元素

```python
# 查找所有符合條件的元素
elements = driver.find_elements(By.CSS_SELECTOR, ".list-item")

for element in elements:
    print(element.text)

# 取得數量
count = len(driver.find_elements(By.CSS_SELECTOR, ".item"))
```

### 常用 CSS Selector

```python
# 基本選擇器
"#id-name"              # ID
".class-name"           # Class
"div"                   # Tag
"[name='field']"        # 屬性

# 組合選擇器
"div.container"         # div 且有 class container
"div#main"             # div 且 ID 是 main
"div.a.b"              # div 且有 class a 和 b

# 層級選擇器
"div > span"           # 直接子元素
"div span"             # 所有後代
"div + span"           # 相鄰兄弟
"div ~ span"           # 所有兄弟

# 屬性選擇器
"[type='text']"        # 完全匹配
"[class*='partial']"   # 包含
"[class^='start']"     # 開頭
"[class$='end']"       # 結尾

# 偽類選擇器
"li:first-child"       # 第一個
"li:last-child"        # 最後一個
"li:nth-child(2)"      # 第 N 個
```

### 常用 XPath

```python
# 基本
"//div[@id='main']"                    # ID
"//div[@class='container']"            # Class
"//input[@type='text']"                # 屬性

# 文字匹配
"//button[text()='登入']"              # 完全匹配
"//button[contains(text(), '登')]"     # 包含
"//button[starts-with(text(), '登')]"  # 開頭

# 層級
"//div/span"                           # 直接子元素
"//div//span"                          # 所有後代
"//div/.."                             # 父元素
"//div/following-sibling::span"        # 後面兄弟

# 條件組合
"//input[@type='text' and @name='user']"
"//input[@type='text' or @type='password']"
"//div[not(@class='hidden')]"
```

---

## 元素操作

### 基本操作

```python
# 點擊
element.click()

# 輸入文字
element.send_keys("輸入內容")

# 清空後輸入
element.clear()
element.send_keys("新內容")

# 取得文字
text = element.text

# 取得屬性
value = element.get_attribute("value")
href = element.get_attribute("href")
class_name = element.get_attribute("class")

# 取得 CSS 屬性
color = element.value_of_css_property("color")
```

### 表單操作

```python
from selenium.webdriver.support.ui import Select

# 文字輸入框
input_field = driver.find_element(By.ID, "username")
input_field.clear()
input_field.send_keys("test@example.com")

# 密碼框
password_field = driver.find_element(By.ID, "password")
password_field.send_keys("password123")

# 下拉選單
select = Select(driver.find_element(By.ID, "dropdown"))
select.select_by_value("option1")          # 依 value
select.select_by_visible_text("選項一")    # 依文字
select.select_by_index(0)                  # 依索引

# 取得選中的選項
selected = select.first_selected_option
print(selected.text)

# Checkbox
checkbox = driver.find_element(By.ID, "agree")
if not checkbox.is_selected():
    checkbox.click()

# Radio Button
radio = driver.find_element(By.CSS_SELECTOR, "input[value='option1']")
radio.click()

# 檔案上傳
file_input = driver.find_element(By.ID, "file-upload")
file_input.send_keys("/absolute/path/to/file.pdf")
```

### 鍵盤操作

```python
from selenium.webdriver.common.keys import Keys

element = driver.find_element(By.ID, "search")

# 特殊按鍵
element.send_keys(Keys.ENTER)
element.send_keys(Keys.TAB)
element.send_keys(Keys.ESCAPE)
element.send_keys(Keys.BACKSPACE)

# 組合鍵
element.send_keys(Keys.CONTROL + "a")  # 全選
element.send_keys(Keys.CONTROL + "c")  # 複製
element.send_keys(Keys.CONTROL + "v")  # 貼上
```

### 元素狀態

```python
element = driver.find_element(By.ID, "button")

# 檢查狀態
element.is_displayed()   # 是否可見
element.is_enabled()     # 是否可用
element.is_selected()    # 是否選中（checkbox/radio）

# 取得位置和大小
location = element.location  # {'x': 100, 'y': 200}
size = element.size          # {'width': 100, 'height': 50}
rect = element.rect          # 完整資訊
```

---

## 等待策略

### 隱式等待

```python
# 全域設定，影響所有 find_element
driver.implicitly_wait(10)  # 秒
```

### 顯式等待（推薦）

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)

# 等待元素出現
element = wait.until(
    EC.presence_of_element_located((By.ID, "element-id"))
)

# 等待元素可見
element = wait.until(
    EC.visibility_of_element_located((By.ID, "element-id"))
)

# 等待元素可點擊
element = wait.until(
    EC.element_to_be_clickable((By.ID, "button"))
)

# 等待元素消失
wait.until(
    EC.invisibility_of_element_located((By.ID, "loading"))
)

# 等待文字出現
wait.until(
    EC.text_to_be_present_in_element((By.ID, "message"), "成功")
)

# 等待 URL 變化
wait.until(EC.url_contains("/dashboard"))
wait.until(EC.url_to_be("http://example.com/page"))

# 等待 title 變化
wait.until(EC.title_contains("Dashboard"))

# 等待 alert 出現
alert = wait.until(EC.alert_is_present())
```

### 自訂等待條件

```python
def wait_for_element_count(locator, count):
    """等待元素數量達到指定值"""
    def condition(driver):
        elements = driver.find_elements(*locator)
        return len(elements) >= count
    return condition

# 使用
wait.until(wait_for_element_count((By.CSS_SELECTOR, ".item"), 5))
```

### 封裝等待函數

```python
class WaitHelper:
    def __init__(self, driver, timeout=10):
        self.driver = driver
        self.wait = WebDriverWait(driver, timeout)
    
    def find(self, by, value):
        """等待並返回元素"""
        return self.wait.until(
            EC.presence_of_element_located((by, value))
        )
    
    def click(self, by, value):
        """等待可點擊並點擊"""
        element = self.wait.until(
            EC.element_to_be_clickable((by, value))
        )
        element.click()
        return element
    
    def input(self, by, value, text):
        """等待並輸入文字"""
        element = self.find(by, value)
        element.clear()
        element.send_keys(text)
        return element
    
    def wait_invisible(self, by, value):
        """等待元素消失"""
        self.wait.until(
            EC.invisibility_of_element_located((by, value))
        )
```

---

## 瀏覽器操作

### 導航

```python
# 開啟 URL
driver.get("http://example.com")

# 重新整理
driver.refresh()

# 上一頁 / 下一頁
driver.back()
driver.forward()

# 取得當前 URL
current_url = driver.current_url

# 取得 title
title = driver.title
```

### 視窗操作

```python
# 最大化
driver.maximize_window()

# 設定大小
driver.set_window_size(1920, 1080)

# 取得大小
size = driver.get_window_size()

# 全螢幕
driver.fullscreen_window()

# 最小化
driver.minimize_window()
```

### 多視窗/分頁

```python
# 取得當前視窗 handle
main_window = driver.current_window_handle

# 取得所有視窗
all_windows = driver.window_handles

# 切換視窗
driver.switch_to.window(all_windows[1])

# 開新分頁
driver.execute_script("window.open('http://example.com', '_blank');")

# 關閉當前視窗
driver.close()

# 切回主視窗
driver.switch_to.window(main_window)
```

### iframe 操作

```python
# 切換到 iframe（by ID）
driver.switch_to.frame("iframe-id")

# 切換到 iframe（by 元素）
iframe = driver.find_element(By.CSS_SELECTOR, "iframe.main")
driver.switch_to.frame(iframe)

# 切換到 iframe（by 索引）
driver.switch_to.frame(0)

# 切回主文件
driver.switch_to.default_content()

# 切回父 frame
driver.switch_to.parent_frame()
```

### Alert 處理

```python
from selenium.webdriver.common.alert import Alert

# 切換到 alert
alert = driver.switch_to.alert

# 取得文字
text = alert.text

# 接受（OK）
alert.accept()

# 取消（Cancel）
alert.dismiss()

# 輸入文字（prompt）
alert.send_keys("輸入內容")
alert.accept()
```

---

## 進階操作

### Action Chains

```python
from selenium.webdriver.common.action_chains import ActionChains

actions = ActionChains(driver)

# 滑鼠懸停
element = driver.find_element(By.ID, "menu")
actions.move_to_element(element).perform()

# 右鍵點擊
actions.context_click(element).perform()

# 雙擊
actions.double_click(element).perform()

# 拖放
source = driver.find_element(By.ID, "source")
target = driver.find_element(By.ID, "target")
actions.drag_and_drop(source, target).perform()

# 按住並拖動
actions.click_and_hold(source).move_to_element(target).release().perform()

# 鍵盤操作
actions.key_down(Keys.CONTROL).click(element).key_up(Keys.CONTROL).perform()
```

### JavaScript 執行

```python
# 執行 JavaScript
driver.execute_script("alert('Hello');")

# 返回值
result = driver.execute_script("return document.title;")

# 操作元素
element = driver.find_element(By.ID, "button")
driver.execute_script("arguments[0].click();", element)

# 滾動到元素
driver.execute_script("arguments[0].scrollIntoView(true);", element)

# 滾動頁面
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")  # 底部
driver.execute_script("window.scrollTo(0, 0);")  # 頂部

# 修改元素屬性
driver.execute_script("arguments[0].setAttribute('value', 'new value');", element)

# 移除元素
driver.execute_script("arguments[0].remove();", element)
```

### 截圖

```python
# 整頁截圖
driver.save_screenshot("screenshot.png")

# 元素截圖
element = driver.find_element(By.ID, "specific-element")
element.screenshot("element.png")

# 取得 base64
base64_image = driver.get_screenshot_as_base64()

# 取得 bytes
png_bytes = driver.get_screenshot_as_png()
```

### Cookie 操作

```python
# 取得所有 cookies
cookies = driver.get_cookies()

# 取得特定 cookie
cookie = driver.get_cookie("session_id")

# 新增 cookie
driver.add_cookie({
    "name": "my_cookie",
    "value": "cookie_value",
    "domain": "example.com"
})

# 刪除 cookie
driver.delete_cookie("my_cookie")

# 刪除所有 cookies
driver.delete_all_cookies()
```

---

## 斷言方法

### pytest 斷言

```python
import pytest

def test_example():
    # 基本斷言
    assert element.text == "預期文字"
    assert "關鍵字" in element.text
    assert element.is_displayed()
    
    # URL 斷言
    assert "/dashboard" in driver.current_url
    assert driver.current_url == "http://example.com/dashboard"
    
    # 元素存在
    elements = driver.find_elements(By.CSS_SELECTOR, ".item")
    assert len(elements) > 0
    assert len(elements) == 5
    
    # 元素不存在
    error_elements = driver.find_elements(By.ID, "error")
    assert len(error_elements) == 0
    
    # 屬性斷言
    assert element.get_attribute("class") == "active"
    assert "disabled" not in element.get_attribute("class")
```

### 軟斷言（收集所有失敗）

```python
import pytest
from pytest_check import check

def test_with_soft_assertions():
    with check:
        assert element1.text == "Text 1"
    with check:
        assert element2.text == "Text 2"
    with check:
        assert element3.is_displayed()
    # 所有斷言都會執行，最後彙總失敗
```

### 自訂斷言

```python
def assert_element_text(element, expected, message=""):
    """斷言元素文字"""
    actual = element.text.strip()
    assert actual == expected, f"{message}\n預期: {expected}\n實際: {actual}"

def assert_url_contains(driver, expected_path):
    """斷言 URL 包含特定路徑"""
    assert expected_path in driver.current_url, \
        f"URL 應包含 {expected_path}，實際為 {driver.current_url}"

def assert_element_visible(driver, by, value, timeout=10):
    """斷言元素可見"""
    try:
        WebDriverWait(driver, timeout).until(
            EC.visibility_of_element_located((by, value))
        )
    except:
        pytest.fail(f"元素 {by}={value} 在 {timeout} 秒內未出現")
```
