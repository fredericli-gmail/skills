---
name: 開發-Python後端
description: "Python 資深後端開發者協助。FastAPI + SQLAlchemy 架構，每行程式碼加繁體中文註解。嚴格遵守 SQL Injection、敏感資料保護等資安規範。所有開發必須先完成系統分析，使用者輸入 OKOKYES 後方可啟動開發。當涉及 .py 檔案、FastAPI 專案、requirements.txt / pyproject.toml 時觸發。前端請另外使用 /開發-React前端。"
---

# Skill: Python 後端開發（FastAPI）

> ⚠️ **本 SKILL 引用共享規範**：
> - 編碼規範：[_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md)
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)

## Usage Trigger

- 當涉及 Python 檔案 (.py) 或 FastAPI 專案架構時自動啟用
- 當進行後端業務邏輯開發、API 端點新增或修改時
- 不負責前端：前端請使用 `/開發-React前端`
- 當進入測試，請使用 `/測試` Skill

---

## 1. Python 後端規範

### 1.1 核心框架

| 項目 | 選用 |
|------|------|
| Web 框架 | FastAPI（ASGI）|
| ORM | SQLAlchemy 2.x（async 優先）|
| 資料驗證 | Pydantic v2 |
| 認證 | PyJWT + bcrypt |
| HTTP 客戶端 | httpx（async）|
| Migration | Alembic |
| 設定管理 | YAML 配置檔或環境變數 |

### 1.2 Python 版本與語法

- **最低版本**：Python 3.10+
- **類型標註**：所有函式必須有完整的 type hints
- **允許語法**：f-string、walrus operator (`:=`)、match-case、dataclass、async/await

### 1.3 禁止語法

| 禁止 | 替代 |
|------|------|
| ❌ `print()` 作為日誌輸出 | ✅ `logging` 模組 |
| ❌ bare `except:` | ✅ 指定例外類型 `except XxxError:` |
| ❌ mutable default arguments：`def foo(items=[])` | ✅ `def foo(items=None)`，內部 `items = items or []` |
| ❌ `from xxx import *`（wildcard import） | ✅ 明確 import |

### 1.4 程式碼風格

- **縮排**：4 空格（禁止 Tab）
- **行長限制**：120 字元
- **引號**：字串統一使用雙引號 `"`
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

### 1.5 Logger 標準寫法

```python
import logging

# ✅ 正確：模組層級宣告 logger
logger = logging.getLogger(__name__)

# ✅ 正確：函式中使用，含完整堆疊
logger.error("建立使用者失敗，user_id: %s", user_id, exc_info=True)
```

詳見 [共享規範 §3](../_shared/CODING-STANDARDS.md#3-exception-處理與日誌規範)。

---

## 2. FastAPI 專案結構規範

```
project/
├── shared/                    # 共用模組
│   ├── __init__.py
│   ├── auth.py               # 認證與授權
│   ├── audit.py              # 稽核日誌
│   ├── database.py           # 資料庫連線與 Session
│   ├── db_models.py          # SQLAlchemy ORM 模型
│   ├── schemas.py            # Pydantic 請求/回應 Schema
│   └── dependencies.py       # FastAPI Depends 共用依賴
├── routers/                   # API 路由（按功能拆分）
│   ├── __init__.py
│   ├── auth_router.py        # 認證相關 API
│   └── user_router.py        # 使用者管理 API
├── alembic/                   # 資料庫 migration
└── main.py                    # FastAPI 應用入口
```

---

## 3. FastAPI Router 架構規範

> ⚠️ **強制遵守**：詳見 [共享規範 §1](../_shared/CODING-STANDARDS.md#1-controller--service-架構規範)。
> Python 中的「Router」對應 Java 中的「Controller」。

### 3.1 設計原則

| 規則 | 說明 |
|------|------|
| **專用 Router** | 每個功能模組對應一個 Router 檔案（如 `auth_router.py`、`user_router.py`） |
| **Router Prefix** | 每個 Router 使用 `APIRouter(prefix="/api/xxx")` 統一前綴 |
| **禁止跨 Router** | ❌ 禁止在 Router A 中直接呼叫 Router B 的內部函式 |
| **共用邏輯** | 跨 Router 共用的邏輯放在 `shared/` 模組中 |

### 3.2 範例

```python
# routers/auth_router.py
from fastapi import APIRouter, Depends
from shared.auth import require_auth
from shared.schemas import LoginRequest, LoginResponse

# 建立認證路由器，統一前綴 /api/auth
router = APIRouter(prefix="/api/auth", tags=["認證"])

@router.post("/login", response_model=LoginResponse)
async def login(request: LoginRequest):
    """
    使用者登入端點

    Args:
        request: 登入請求（含帳號密碼）
    Returns:
        登入成功後的 Token 與使用者資訊
    """
    # 業務邏輯...
    ...

@router.get("/me")
async def get_current_user(user=Depends(require_auth)):
    """取得目前登入使用者資訊"""
    return user
```

```python
# main.py - 註冊所有 Router
from fastapi import FastAPI
from routers.auth_router import router as auth_router
from routers.user_router import router as user_router

app = FastAPI()
app.include_router(auth_router)
app.include_router(user_router)
```

---

## 4. 註解規範

詳見 [共享規範 §2](../_shared/CODING-STANDARDS.md#2-註解規範)。

- **逐行註解**：每一行邏輯程式碼必須附上繁體中文 `# 註解`
- **函式 Docstring**：每個函式必須有繁體中文 Docstring，說明用途、參數、回傳值
- **模組分隔線**：使用 `# ═══` 分隔線區隔不同功能區塊

```python
# ═══════════════════════════════════════════════
#  使用者認證
# ═══════════════════════════════════════════════

async def authenticate_user(username: str, password: str) -> dict | None:
    """
    驗證使用者帳號密碼

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

---

## 5. Exception 處理與依賴注入

詳見 [共享規範 §3](../_shared/CODING-STANDARDS.md#3-exception-處理與日誌規範)。

### 5.1 Exception 處理範例

```python
# ✅ 正確：bare except 禁止；必須含 exc_info=True
try:
    user = await create_user(user_data)
except IntegrityError as e:
    logger.error("建立使用者失敗，username: %s", user_data.username, exc_info=True)
    raise HTTPException(status_code=409, detail="使用者名稱已存在") from e
except Exception as e:
    logger.error("建立使用者發生未預期錯誤", exc_info=True)
    raise HTTPException(status_code=500, detail="內部伺服器錯誤") from e
```

### 5.2 FastAPI 依賴注入

```python
# shared/dependencies.py
from fastapi import Depends, HTTPException, Request
from shared.auth import verify_jwt_token

async def require_auth(request: Request) -> dict:
    """要求使用者已登入，回傳使用者資訊"""
    # 從 Cookie 或 Header 取得 JWT
    token = request.cookies.get("session") or _extract_bearer(request)
    # 若無 token，回傳 401
    if not token:
        raise HTTPException(status_code=401, detail="未登入")
    # 驗證 token 並取得使用者
    user = verify_jwt_token(token)
    if not user:
        raise HTTPException(status_code=401, detail="Token 無效或已過期")
    return user

def require_role(role: str):
    """建立角色檢查依賴"""
    async def _check(user: dict = Depends(require_auth)) -> dict:
        # 檢查使用者角色是否符合
        if user.get("role") not in _get_allowed_roles(role):
            raise HTTPException(status_code=403, detail="權限不足")
        return user
    return _check
```

---

## 6. 資安規範

> 詳見 [共享規範 SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)。

### 6.1 SQL Injection 防護

```python
# ✅ 正確：使用 ORM
result = session.query(User).filter(User.username == username).first()

# ✅ 正確：使用參數綁定
result = session.execute(text("SELECT * FROM users WHERE id = :id"), {"id": user_id})

# ❌ 錯誤：字串拼接
result = session.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

### 6.2 密碼與機密管理

```python
import os

# ✅ 正確：環境變數，啟動時檢查
JWT_SECRET = os.environ.get("JWT_SECRET", "")
if not JWT_SECRET:
    raise RuntimeError("JWT_SECRET 環境變數未設定")

# ❌ 錯誤：硬編碼
JWT_SECRET = "my-super-secret-key"
```

### 6.3 認證安全

- 密碼必須使用 bcrypt 雜湊儲存
- JWT Access Token 有效期不超過 24 小時
- Refresh Token 有效期不超過 7 天
- 密碼錯誤統一回傳「帳號或密碼錯誤」（不洩露帳號是否存在）
- 登入失敗達上限須鎖定帳號或啟用延遲

---

## 7. 資料庫異動

詳見 [共享規範 §4](../_shared/CODING-STANDARDS.md#4-資料庫異動規範)。

- **強制使用 Alembic** 管理 migration
- 使用 `alembic revision --autogenerate -m "描述"` 產生
- 已執行的 migration 禁止修改

---

## 8. 開發流程強制規範

### 8.1 啟動開發的唯一條件

```
┌─────────────────────────────────────────────────────────┐
│  僅當使用者輸入「OKOKYES」關鍵字時，方可啟動開發程序    │
│  ❌ 禁止接受其他確認詞彙                                │
└─────────────────────────────────────────────────────────┘
```

### 8.2 完成檢查清單

- [ ] 所有程式碼皆有繁體中文註解
- [ ] 所有函式皆有 type hints + docstring
- [ ] 通過例外處理檢查（無 bare except、無 print、有 exc_info=True）
- [ ] 無字串拼接 SQL
- [ ] 無硬編碼密碼或 Secret
- [ ] 區塊分隔線風格與現有程式碼一致
- [ ] 新模組放在正確位置（shared/ 或 routers/）
- [ ] 程式可正常啟動（`python -m py_compile <檔案>`）

---

## 9. Skill 串接機制（強制執行）

開發完成且程式可正常啟動後，必須輸出：

```
【Skill 串接通知】

✅ 開發階段已完成
✅ 程式碼可正常執行
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

此 Skill 將協助您進行：
- Python 程式碼品質審查
- 資安規範驗證

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview`
- ❌ 其他輸入 → 不啟動
