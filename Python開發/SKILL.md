---
name: Python開發
description: Python 資深開發者協助。後端採用 FastAPI + SQLAlchemy 架構，前端採用 React 18 + Vite + Tailwind CSS 架構（JSX）。每行程式碼加繁體中文註解。嚴格遵守 XSS、SQL Injection 等資安規範。適用於 Python + React 全端開發，特別是 FastAPI 微服務、CLI 工具、模型管理系統等 Python 專案。所有開發必須先完成系統分析，使用者輸入 OKOKYES 後方可啟動開發。
---

# Skill: Full-Stack Developer (Python Backend + React Frontend)

## Usage Trigger
- 當涉及 Python 檔案 (.py) 或 FastAPI 專案架構時自動啟用。
- 當進行後端業務邏輯開發、API 端點新增或修改時。
- 當涉及 React 元件 (.jsx/.js)、CSS 或前端開發時。
- 當涉及前端頁面新增、修改或遷移時。
- 當進入測試，則請使用 測試 skill

---

## 1. 開發技術規範

### 1.1 後端 Python 規範

#### 核心框架
- **Web 框架**：FastAPI（ASGI）
- **ORM**：SQLAlchemy 2.x（async 優先）
- **資料驗證**：Pydantic v2（FastAPI 原生）
- **認證**：PyJWT + bcrypt
- **HTTP 客戶端**：httpx（async）
- **設定管理**：YAML 配置檔（非 .env，與現有專案一致）

#### Python 版本與語法
- **最低版本**：Python 3.10+
- **類型標註**：所有函式必須有完整的 type hints
- **允許語法**：f-string、walrus operator (`:=`)、match-case、dataclass、async/await
- **禁止語法**：
  - ❌ 禁止使用 `print()` 作為日誌輸出，統一使用 `logging` 模組
  - ❌ 禁止使用 bare `except:`，必須指定例外類型
  - ❌ 禁止使用 mutable default arguments（`def foo(items=[])`）
  - ❌ 禁止使用 `import *`（wildcard import）

#### 程式碼風格
- **縮排**：4 空格（禁止 Tab）
- **行長限制**：120 字元
- **引號**：字串統一使用雙引號 `"`（與現有程式碼一致）
- **Import 排序**：標準庫 → 第三方 → 本地模組，各組之間空一行
- **Module Docstring**：每個 `.py` 檔案開頭必須有模組說明

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
模組說明：用途描述
提供 XXX 功能。
"""
```

#### Logger 宣告規範
```python
# ✅ 正確：模組層級宣告 logger
import logging

logger = logging.getLogger(__name__)

# ✅ 正確：在函式中使用
logger.error("建立使用者失敗，user_id: %s", user_id, exc_info=True)

# ❌ 錯誤：使用 print
print("error:", e)

# ❌ 錯誤：吞掉例外
except Exception:
    pass
```

### 1.2 FastAPI 專案結構規範

```
project/
├── shared/                    # 共用模組
│   ├── __init__.py
│   ├── auth.py               # 認證與授權
│   ├── audit.py              # 稽核日誌
│   ├── database.py           # 資料庫連線與 Session
│   ├── models.py             # SQLAlchemy ORM 模型
│   ├── schemas.py            # Pydantic 請求/回應 Schema
│   └── dependencies.py       # FastAPI Depends 共用依賴
├── routers/                   # API 路由（按功能拆分）
│   ├── __init__.py
│   ├── auth_router.py        # 認證相關 API
│   ├── model_router.py       # 模型管理 API
│   └── admin_router.py       # 管理員 API
├── mlops-ui/                  # React 前端
│   ├── src/
│   ├── vite.config.js
│   └── package.json
└── main.py                    # FastAPI 應用入口
```

### 1.3 FastAPI Router 架構規範

> ⚠️ **最高原則**：每一個前端頁面功能，對應到後端必須有專用的 Router 模組，禁止跨 Router 混用端點。

#### Router 設計原則

| 規則 | 說明 |
|------|------|
| **專用 Router** | 每個功能模組對應一個 Router 檔案（如 `auth_router.py`、`model_router.py`） |
| **Router Prefix** | 每個 Router 使用 `APIRouter(prefix="/api/xxx")` 統一前綴 |
| **禁止跨 Router** | ❌ 禁止在 Router A 中直接呼叫 Router B 的內部函式 |
| **共用邏輯** | 跨 Router 共用的邏輯放在 `shared/` 模組中 |

#### 範例

```python
# routers/auth_router.py
from fastapi import APIRouter, Depends
from shared.auth import require_auth, require_role
from shared.schemas import LoginRequest, LoginResponse

# 建立認證路由器，統一前綴 /api/auth
router = APIRouter(prefix="/api/auth", tags=["認證"])

@router.post("/login", response_model=LoginResponse)
async def login(request: LoginRequest):
    """使用者登入端點"""
    ...

@router.get("/me")
async def get_current_user(user=Depends(require_auth)):
    """取得目前登入使用者資訊"""
    ...
```

```python
# main.py - 註冊所有 Router
from routers.auth_router import router as auth_router
from routers.model_router import router as model_router

app.include_router(auth_router)
app.include_router(model_router)
```

### 1.4 前端技術規範
- **前端框架**：強制使用 React 18 + Vite + Tailwind CSS 架構（JSX，非 TypeScript）。
- **元件格式**：使用函式元件（Function Component），禁止使用 Class Component。
- **狀態管理**：全域狀態使用 Zustand，伺服器快取可使用 React Query 或 SWR。
- **檔案格式**：前端使用 `.jsx` / `.js` 檔案，不使用 TypeScript（`.tsx` / `.ts`）。
- **JavaScript 語法**：前端允許使用箭頭函式 `=>`、`const/let`、解構賦值、Template Literal，禁止使用 `var`。

### 1.5 資料庫異動規範
- **ORM 優先**：使用 SQLAlchemy 模型定義資料結構。
- **Migration 工具**：使用 Alembic 管理資料庫遷移。
- **禁止直接操作資料庫**：
  - ❌ 禁止直接對資料庫執行手動 DDL 語句
  - ❌ 禁止使用 DBeaver、pgAdmin 等工具直接修改資料庫結構
- **Alembic 規範**：
  - 每次結構變更建立新的 migration 版本
  - 使用 `alembic revision --autogenerate -m "描述"`
  - 已執行的 migration 禁止修改

---

## 2. 註解規範

### 2.1 Python 註解
- **逐行註解**：每一行具備邏輯意義的程式碼都必須附上繁體中文註解。
- **函式 Docstring**：每個函式必須有繁體中文 Docstring，說明用途、參數、回傳值。
- **模組分隔線**：使用 `# ═══` 分隔線區隔不同功能區塊（與現有程式碼風格一致）。

```python
# ═══════════════════════════════════════════════
#  使用者認證
# ═══════════════════════════════════════════════

async def authenticate_user(username: str, password: str) -> dict | None:
    """
    驗證使用者帳號密碼。
    Args:
        username: 使用者帳號
        password: 明文密碼
    Returns:
        使用者資訊 dict，驗證失敗回傳 None
    """
    # 從資料庫查詢使用者
    user = await get_user_by_username(username)
    # 若使用者不存在，回傳 None
    if user is None:
        return None
    # 驗證密碼雜湊是否匹配
    if not bcrypt.checkpw(password.encode(), user.password_hash.encode()):
        return None
    # 驗證成功，回傳使用者資訊
    return user
```

### 2.2 React 註解
- JSX 模板中的關鍵邏輯區塊須加上 `{/* 註解 */}` 說明。
- 元件區塊使用 `// ==================== N. 區塊名 ====================` 分隔。

---

## 3. 代碼品質要求

### 3.1 Python 品質標準
- 所有函式必須有完整的 type hints（參數 + 回傳值）
- 必須包含完整的例外處理機制
- 所有對外 API 必須有 Pydantic Schema 驗證
- 非同步函式優先（`async def`），除非有明確理由使用同步

### 3.2 Exception 處理與日誌規範（強制）

> ⚠️ **最高原則**：所有 Exception 必須記錄完整的堆疊追蹤（Stack Trace），禁止吞掉例外或僅記錄訊息。

**正確範例**：
```python
try:
    # 嘗試建立使用者
    user = await create_user(user_data)
except IntegrityError as e:
    # ✅ 正確：包含錯誤描述與完整堆疊追蹤
    logger.error("建立使用者失敗，username: %s", user_data.username, exc_info=True)
    raise HTTPException(status_code=409, detail="使用者名稱已存在") from e
except Exception as e:
    # ✅ 正確：未預期的錯誤，記錄完整追蹤並回傳 500
    logger.error("建立使用者發生未預期錯誤", exc_info=True)
    raise HTTPException(status_code=500, detail="內部伺服器錯誤") from e
```

**禁止範例**：
```python
# ❌ 錯誤：bare except
try:
    user = await create_user(user_data)
except:
    pass

# ❌ 錯誤：僅記錄訊息，無堆疊追蹤
except Exception as e:
    logger.error(f"失敗: {e}")

# ❌ 錯誤：使用 print
except Exception as e:
    print(e)
```

### 3.3 FastAPI 依賴注入規範

```python
# shared/dependencies.py

from fastapi import Depends, HTTPException, Request
from shared.auth import verify_jwt_token

# 認證依賴：從 Cookie 或 Header 取得 JWT
async def require_auth(request: Request) -> dict:
    """要求使用者已登入，回傳使用者資訊"""
    token = request.cookies.get("session") or _extract_bearer(request)
    if not token:
        raise HTTPException(status_code=401, detail="未登入")
    user = verify_jwt_token(token)
    if not user:
        raise HTTPException(status_code=401, detail="Token 無效或已過期")
    return user

# 角色依賴：要求特定角色
def require_role(role: str):
    """建立角色檢查依賴"""
    async def _check(user: dict = Depends(require_auth)) -> dict:
        if user.get("role") not in _get_allowed_roles(role):
            raise HTTPException(status_code=403, detail="權限不足")
        return user
    return _check
```

---

## 4. React 前端開發規範

### 4.1 元件結構規範

```jsx
// ==================== 1. 匯入區 ====================
// React 核心
import { useState, useEffect, useCallback } from 'react'
// 路由
import { useNavigate, useParams } from 'react-router-dom'
// API 模組
import { modelApi } from '@/api/models'
// Hooks
import { useAuth } from '@/hooks/useAuth'
// 元件
import { DataTable } from '@/components/common/DataTable'

// ==================== 2. 元件定義 ====================
export function ModelListPage() {
  // ==================== 3. Hooks 呼叫 ====================
  const navigate = useNavigate()
  const { hasPermission } = useAuth()

  // ==================== 4. State ====================
  const [loading, setLoading] = useState(false)
  const [models, setModels] = useState([])

  // ==================== 5. 方法 ====================
  /** 載入模型列表 */
  const loadModels = useCallback(async () => {
    setLoading(true)
    try {
      const response = await modelApi.getModels()
      setModels(response.data)
    } catch (error) {
      console.error('[模型管理] 載入失敗', error)
    } finally {
      setLoading(false)
    }
  }, [])

  // ==================== 6. Effects ====================
  useEffect(() => {
    loadModels()
  }, [loadModels])

  // ==================== 7. Render ====================
  return (
    <div className="p-6">
      {/* 頁面標題列 */}
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">模型管理</h1>
      </div>
    </div>
  )
}
```

### 4.2 命名規範

| 項目 | 規範 | 範例 |
|------|------|------|
| **元件檔名** | PascalCase | `ModelList.jsx`、`Modal.jsx` |
| **頁面元件** | PascalCase + `Page` 後綴 | `DashboardPage.jsx`、`LoginPage.jsx` |
| **Hook** | camelCase + `use` 前綴 | `useAuth.js`、`useSSE.js` |
| **API 模組** | camelCase | `auth.js`、`models.js` |
| **CSS class** | Tailwind class 優先 | - |
| **路由 path** | kebab-case | `/models`、`/audit-log` |

### 4.3 API 客戶端規範

```javascript
// api/client.js - Axios 實例 + JWT 攔截器
import axios from 'axios'

// 建立 Axios 實例，設定基礎 URL 與逾時
const api = axios.create({
  baseURL: '/api',
  timeout: 30000,
})

// 請求攔截器：自動帶 JWT
api.interceptors.request.use((config) => {
  // 從 localStorage 取得 access token
  const token = localStorage.getItem('accessToken')
  // 若有 token，加入 Authorization header
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 回應攔截器：401 自動導向登入頁
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    // 401 未授權：導向登入頁面
    if (error.response?.status === 401) {
      localStorage.removeItem('accessToken')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default api
```

### 4.4 SSE 串流消費規範

```javascript
// hooks/useSSE.js - SSE 串流 Hook（與現有 Python SSE 格式對齊）

/**
 * 消費 FastAPI SSE 端點
 * @param {string} url - SSE 端點 URL
 * @param {function} onEvent - 事件回呼 ({step, message, progress, status})
 */
export async function fetchSSE(url, { method = 'POST', onEvent, onError }) {
  // 發送請求，附加 stream=true 參數
  const response = await fetch(`${url}?stream=true`, { method })
  // 取得 ReadableStream reader
  const reader = response.body.getReader()
  // 建立文字解碼器
  const decoder = new TextDecoder()
  let buffer = ''

  while (true) {
    // 讀取串流資料
    const { done, value } = await reader.read()
    if (done) break
    // 解碼並累加到緩衝區
    buffer += decoder.decode(value, { stream: true })
    // 以換行分割事件
    const lines = buffer.split('\n')
    // 保留最後一行（可能不完整）
    buffer = lines.pop()

    for (const line of lines) {
      // 解析 SSE data 行
      if (line.startsWith('data: ')) {
        try {
          const event = JSON.parse(line.slice(6))
          onEvent(event)
        } catch (e) {
          // 忽略非 JSON 行
        }
      }
    }
  }
}
```

---

## 5. 資安規範

### 5.1 XSS 防護
| 情境 | 措施 |
|------|------|
| 輸出使用者輸入 | React JSX 自動 escape，禁止 `dangerouslySetInnerHTML` |
| Markdown 渲染 | 使用 `react-markdown`（自動 sanitize） |
| URL 參數 | 使用 `encodeURIComponent()` |

### 5.2 CSRF 防護
- JWT Token 透過 HttpOnly Cookie + Authorization header
- FastAPI 不需額外 CSRF middleware（Cookie SameSite=Strict）

### 5.3 SQL Injection 防護
- 絕對禁止字串拼接 SQL
- 使用 SQLAlchemy ORM 查詢或 `text()` 參數綁定

```python
# ✅ 正確：使用 ORM
result = session.query(User).filter(User.username == username).first()

# ✅ 正確：使用參數綁定
result = session.execute(text("SELECT * FROM users WHERE id = :id"), {"id": user_id})

# ❌ 錯誤：字串拼接
result = session.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

### 5.4 密碼與機密管理
- 密碼必須使用 bcrypt 雜湊儲存
- API Key、Secret 透過環境變數讀取，禁止硬編碼
- JWT Secret 必須從環境變數取得

```python
# ✅ 正確：環境變數
JWT_SECRET = os.environ.get("JWT_SECRET", "")
if not JWT_SECRET:
    raise RuntimeError("JWT_SECRET 環境變數未設定")

# ❌ 錯誤：硬編碼
JWT_SECRET = "my-super-secret-key"
```

### 5.5 認證安全
- JWT Token 有效期限不超過 24 小時
- Refresh Token 有效期限不超過 7 天
- 密碼錯誤不洩露「帳號是否存在」（統一回傳「帳號或密碼錯誤」）
- 登入失敗達到上限須鎖定帳號或啟用延遲

---

## 6. 與現有 Python 程式碼整合規範

> ⚠️ **關鍵原則**：本專案已有成熟的 Python 模組架構（shared/），新增功能必須遵循現有風格。

### 6.1 程式碼風格對齊

| 項目 | 現有慣例 | 必須遵循 |
|------|---------|---------|
| 區塊分隔 | `# ═══════════════════════════════════════════════` | ✅ |
| 區塊標題 | `#  標題文字` （前後各一個空格） | ✅ |
| 模組開頭 | `#!/usr/bin/env python3` + `# -*- coding: utf-8 -*-` + docstring | ✅ |
| 函式命名 | snake_case | ✅ |
| 常數命名 | UPPER_SNAKE_CASE | ✅ |
| 字串引號 | 雙引號 `"` | ✅ |
| 配置取值 | `config.get("key", default)` 防禦式 | ✅ |

### 6.2 新增模組位置規範

| 新功能 | 放置位置 | 原因 |
|--------|---------|------|
| 認證邏輯 | `shared/auth.py` | MLX 與 vLLM 共用 |
| 稽核日誌 | `shared/audit.py` | MLX 與 vLLM 共用 |
| DB 連線 | `shared/database.py` | MLX 與 vLLM 共用 |
| ORM 模型 | `shared/db_models.py` | 避免與 `models.yaml` 混淆 |
| Pydantic Schema | `shared/schemas.py` | 請求/回應格式定義 |
| 認證 API | 直接加在 `mlx_manager.py` / `vllm_manager.py` | 引擎特定的路由註冊 |

### 6.3 與現有 FastAPI 整合方式

```python
# 在 mlx_manager.py 中整合新功能（非建立新 app）

# 1. 匯入新模組
from shared.auth import auth_middleware, require_auth, require_role
from shared.audit import audit_log

# 2. 註冊中介軟體（在 lifespan 中或 app 初始化時）
app.middleware("http")(auth_middleware)

# 3. 現有 API 加上認證（最小改動）
@app.post("/api/models/{model_id}/start")
async def start_model(model_id: str, request: Request,
                      user: dict = Depends(require_role("operator"))):
    # 記錄稽核日誌
    audit_log(user, "start_model", model_id, request)
    # ↓ 以下為現有邏輯，完全不動 ↓
    ...
```

---

## 7. 開發流程強制規範

### 7.1 分析階段
1. 需求評估、受影響檔案清單、邏輯變更點、資安風險評估
2. 等待使用者輸入 **`OKOKYES`** 後方可開始開發

### 7.2 啟動開發的唯一條件
- ✅ `OKOKYES` → 啟動開發
- ❌ 「確認」「同意」「OK」「好」→ 不啟動

### 7.3 完成檢查清單

**後端 Python**：
- [ ] 所有程式碼皆有繁體中文註解
- [ ] 所有函式皆有 type hints + docstring
- [ ] 通過例外處理檢查（無 bare except、無 print、有 exc_info）
- [ ] 無字串拼接 SQL
- [ ] 無硬編碼密碼或 Secret
- [ ] 區塊分隔線風格與現有程式碼一致
- [ ] 新模組放在正確位置（shared/ 或引擎目錄）
- [ ] 程式可正常啟動（`python -m py_compile`）

**前端 React**：
- [ ] 所有程式碼皆有繁體中文註解
- [ ] 無 `dangerouslySetInnerHTML`
- [ ] 使用函式元件，無 Class Component
- [ ] 無 `var` 宣告
- [ ] SSE 消費格式與現有 Python SSE 事件對齊
- [ ] 前端可正常建置（`npm run build`）

---

## 8. Skill 串接機制（強制執行）

開發完成且程式可正常啟動後，必須輸出：

```
【Skill 串接通知】

✅ 開發階段已完成
✅ 程式碼可正常執行
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

此 Skill 將協助您進行：
- Python 程式碼品質審查
- React 前端規範檢查
- 資安規範驗證

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview`
- ❌ 其他輸入 → 不啟動
