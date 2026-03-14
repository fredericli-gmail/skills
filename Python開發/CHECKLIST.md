# Python + React 開發檢查清單

## 後端 Python 審查項目

### 🔴 錯誤（必須修正）

| 類別 | 檢查項目 | 檢測模式 |
|------|---------|---------|
| **架構違規：跨 Router** | Router A 直接匯入 Router B 的內部函式 | 檢查 import 來源是否為其他 router 模組 |
| **架構違規：共用邏輯未放 shared/** | 多個 router 共用的邏輯未放在 shared/ | 檢查重複函式定義 |
| **bare except** | 使用 `except:` 無指定例外類型 | 正則：`except\s*:` |
| **print 日誌** | 使用 `print()` 而非 `logging` | 正則：`\bprint\s*\(` （排除 docstring/註解） |
| **吞掉例外** | `except Exception: pass` 或僅 `logger.error(str(e))` | 檢查 catch 區塊是否有 `exc_info=True` 或 `raise` |
| **SQL Injection** | 使用 f-string 或字串拼接組 SQL | 正則：`execute\s*\(\s*f"` 或 `execute\s*\(.*\+` |
| **硬編碼密碼** | password/secret/key/token 硬編碼字串值 | 正則：`(password\|secret\|key\|token)\s*=\s*"[^"]+"`（忽略大小寫） |
| **import star** | 使用 `from xxx import *` | 正則：`from\s+\S+\s+import\s+\*` |
| **mutable default** | 函式預設值使用可變物件 | 正則：`def\s+\w+\(.*=\s*(\[\]\|\{\}\|set\(\))` |
| **XSS (React)** | 使用 `dangerouslySetInnerHTML` | 正則：`dangerouslySetInnerHTML` |
| **eval 使用** | 使用 `eval()` 或 `exec()` | 正則：`\b(eval\|exec)\s*\(` |

### 🟡 警告（建議修正）

| 類別 | 檢查項目 | 檢測模式 |
|------|---------|---------|
| **缺少 type hints** | 函式缺少參數或回傳值型別標註 | 檢查 `def` 行是否有 `->` 和參數 `:` |
| **缺少 docstring** | 公開函式缺少 docstring | 檢查 `def` 下一行是否為 `"""` |
| **註解缺失** | 程式碼行缺少繁體中文註解 | 檢查非空白、非括號行是否有對應註解 |
| **區塊分隔風格** | 未使用 `═══` 分隔線 | 與現有程式碼風格比對 |
| **模組 docstring** | `.py` 檔案缺少模組開頭 docstring | 檢查檔案開頭是否有 `"""..."""` |
| **log 格式** | `logger.error(f"...")` 而非 `logger.error("...", arg)` | 正則：`logger\.\w+\(f"` |
| **var 禁用 (React)** | 使用 `var` 宣告變數 | 正則：`\bvar\s+` |
| **console.log (React)** | 正式程式碼中殘留 `console.log` | 正則：`console\.log\(` |

### 🟢 建議（可選改善）

| 類別 | 檢查項目 |
|------|---------|
| **函式長度** | 單一函式是否過長（建議 < 50 行） |
| **重複程式碼** | 是否有重複的程式碼區塊可抽取 |
| **命名規範** | Python snake_case、React PascalCase 是否一致 |
| **async 一致性** | 能用 async 的場景是否都使用了 async |
