---
name: 測試
description: "Selenium 自動化測試。必須：(1) 先掃描程式碼取得元素定位 (2) 使用實體瀏覽器（禁用 headless）(3) 設定視窗可見（--start-maximized + --window-position=0,0）。"
---

# Selenium 自動化測試

## 核心原則（必須遵守）

1. **先掃描，後測試** - 絕不在未掃描程式碼前撰寫測試
2. **實體瀏覽器** - 禁止 headless 模式
3. **視窗可見** - 必須設定 `--start-maximized` 和 `--window-position=0,0` 確保視窗可見
4. **精確定位** - 使用從程式碼掃描取得的實際元素 ID/selector
5. **登入流程** - 若功能需登入，必須一併掃描登入相關程式碼

## 執行流程

### Step 1: 掃描程式碼（必須執行）

掃描前端取得元素定位：
```bash
# 掃描 HTML/Thymeleaf 的 ID
grep -rn 'id="' --include="*.html" src/main/resources/templates/

# 掃描 JavaScript 的元素操作
grep -rn 'getElementById\|querySelector\|By.ID\|By.CSS' --include="*.js" src/main/resources/static/js/
```

掃描後端取得 API 端點：
```bash
grep -rn '@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping' --include="*.java" src/main/java/
```

### Step 2: 產出元素清單

掃描完成後輸出：
```
【元素定位清單】
頁面：[頁面路徑]
| 元素 | 定位方式 | 定位值 | 來源 |
|------|---------|--------|------|
| 登入按鈕 | id | loginBtn | login.html:25 |
```

### Step 3: 撰寫測試

使用 Java + Selenium（本專案標準）：
- 測試類別放在 `src/test/java/.../selenium/`
- 使用 `WebDriverWait` 等待元素

**Chrome 選項必須設定**（確保視窗可見）：
```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--no-sandbox");
options.addArguments("--disable-dev-shm-usage");
options.addArguments("--start-maximized");        // 視窗最大化
options.addArguments("--window-position=0,0");    // 視窗從左上角開始
// 禁止使用 --headless 參數！
```

### Step 4: 執行測試

```bash
source /home/ubuntu/selenium-env/bin/activate && ./mvnw test -Dtest=測試類別名稱
```

## 禁止事項

- 使用 `--headless` 模式
- 只設定 `--window-size` 而不設定 `--window-position`（會導致視窗開在螢幕外）
- 未掃描就撰寫測試
- 憑空猜測元素 ID
- 使用 `Thread.sleep()` 取代 `WebDriverWait`

## 測試帳號取得流程

> ⚠️ **禁止在程式碼或文件中寫死帳號密碼**

### 流程規範

1. **情境盤點**：先列出所有測試情境與所需角色
2. **逐一詢問**：針對每個需登入的角色，詢問使用者提供帳號密碼
3. **確認清單**：彙整所有帳號資訊，等待使用者確認後才開始撰寫測試

### 詢問格式

```
【測試登入資訊確認】

本次測試需要以下角色登入：

| # | 角色/情境 | 用途說明 |
|---|----------|---------|
| 1 | [角色名稱] | [測試什麼功能] |
| 2 | [角色名稱] | [測試什麼功能] |

請提供以下資訊：

【帳號密碼】
1. [角色名稱] 帳號/密碼：
2. [角色名稱] 帳號/密碼：

【驗證機制】
- 圖形驗證碼取得方式：（例：API 端點、固定值、或其他方式）
- OTP 驗證碼取得方式：（例：後門日期格式、固定值、或其他方式）
```

### 確認後才可進行

- ✅ 收到所有帳號密碼後，彙整確認
- ✅ 確認圖形驗證碼與 OTP 的取得方式
- ✅ 使用者確認無誤後，方可開始撰寫測試程式碼
- ❌ 禁止猜測或使用預設帳號密碼
- ❌ 禁止假設驗證碼取得方式

## 本專案特定資訊

- 基礎 URL：`http://localhost:8080`

---

## Skill 流程完成通知（強制執行）

> ⚠️ **當所有測試執行完成後，必須輸出完成通知：**

### 測試完成後輸出格式

```
【Skill 流程完成通知】

✅ 分析階段 → 已完成
✅ 開發階段 → 已完成
✅ 測試階段 → 已完成

📊 測試結果摘要：
- 執行測試數：[數量]
- 通過：[數量]
- 失敗：[數量]

🎉 完整的「分析 → 開發 → 測試」流程已全部完成！
```

### 測試失敗處理

若有測試失敗：
1. 列出失敗的測試案例與錯誤原因
2. 詢問使用者是否要修正問題
3. 修正後重新執行測試，直到全部通過
