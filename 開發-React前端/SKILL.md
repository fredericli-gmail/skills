---
name: 開發-React前端
description: "React 18 前端開發者協助。Vite + Tailwind CSS 架構（JSX，非 TypeScript），使用函式元件與 Hooks 開發。每行程式碼加繁體中文註解。嚴格遵守 XSS 防護、敏感資料保護等資安規範。所有開發必須先完成系統分析，使用者輸入 OKOKYES 後方可啟動開發。當涉及 .jsx / .js 檔案、vite.config.js、Tailwind 設定時觸發。"
---

# Skill: React 18 前端開發

> ⚠️ **本 SKILL 引用共享規範**：
> - 編碼規範：[_shared/CODING-STANDARDS.md](../_shared/CODING-STANDARDS.md)
> - 資安規範：[_shared/SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)

## Usage Trigger

- 當涉及 React 元件 (.jsx/.js)、`vite.config.js`、Tailwind 設定時自動啟用
- 當涉及前端頁面新增、修改或遷移時
- 後端開發請使用 `/開發-Java後端` 或 `/開發-Python後端`
- 當進入測試，請使用 `/測試` Skill

---

## 1. 前端技術規範

### 1.1 技術棧

| 項目 | 選用 |
|------|------|
| **前端框架** | React 18 |
| **建構工具** | Vite |
| **CSS 框架** | Tailwind CSS |
| **檔案格式** | `.jsx` / `.js`（**禁用 TypeScript** `.tsx` / `.ts`） |
| **元件格式** | 函式元件 + Hooks（**禁止 Class Component**） |
| **狀態管理** | 全域狀態：Zustand；伺服器狀態：React Query 或 SWR |
| **路由** | React Router |
| **HTTP 客戶端** | Axios |

### 1.2 JavaScript 語法

| 允許 | 禁止 |
|------|------|
| `const` / `let` | ❌ `var` |
| 箭頭函式 `=>` | - |
| 解構賦值 | - |
| Template Literal `` ` ` `` | - |
| 可選鏈 `?.`、空值合併 `??` | - |

---

## 2. 元件結構規範

### 2.1 標準元件範本

```jsx
// ==================== 1. 匯入區 ====================
// React 核心
import { useState, useEffect, useCallback } from 'react'
// 路由
import { useNavigate, useParams } from 'react-router-dom'
// API 模組
import { userApi } from '@/api/users'
// Hooks
import { useAuth } from '@/hooks/useAuth'
// 元件
import { DataTable } from '@/components/common/DataTable'

// ==================== 2. 元件定義 ====================
export function UserListPage({ isEdit = false }) {
  // ==================== 3. Hooks 呼叫 ====================
  const navigate = useNavigate()
  const { hasPermission } = useAuth()

  // ==================== 4. State ====================
  // 載入狀態
  const [loading, setLoading] = useState(false)
  // 使用者列表
  const [users, setUsers] = useState([])

  // ==================== 5. 方法 ====================
  /** 載入使用者列表 */
  const loadUsers = useCallback(async () => {
    // 開始載入
    setLoading(true)
    try {
      // 呼叫 API 取得使用者
      const response = await userApi.getUsers()
      // 更新列表狀態
      setUsers(response.data)
    } catch (error) {
      // 記錄錯誤
      console.error('[使用者管理] 載入失敗', error)
    } finally {
      // 結束載入
      setLoading(false)
    }
  }, [])

  // ==================== 6. Effects ====================
  useEffect(() => {
    // 元件掛載時載入資料
    loadUsers()
  }, [loadUsers])

  // ==================== 7. Render ====================
  return (
    <div className="p-6">
      {/* 頁面標題列 */}
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">使用者管理</h1>
      </div>
    </div>
  )
}
```

---

## 3. 命名規範

| 項目 | 規範 | 範例 |
|------|------|------|
| **元件檔名** | PascalCase | `UserList.jsx`、`Modal.jsx` |
| **頁面元件** | PascalCase + `Page` 後綴 | `DashboardPage.jsx`、`LoginPage.jsx` |
| **Hook** | camelCase + `use` 前綴 | `useAuth.js`、`useWebSocket.js` |
| **API 模組** | camelCase | `auth.js`、`users.js` |
| **CSS class** | Tailwind class 優先；自訂類別用 kebab-case | `bg-blue-500`、`user-card` |
| **路由 path** | kebab-case | `/users`、`/quick-start` |

---

## 4. 註解規範

詳見 [共享規範 §2](../_shared/CODING-STANDARDS.md#2-註解規範)。

- **逐行註解**：每一行邏輯程式碼必須附上繁體中文 `// 註解`
- **JSX 註解**：JSX 模板中的關鍵邏輯區塊使用 `{/* 註解 */}`
- **JSDoc**：非預期/重要的工具函式須附 JSDoc

---

## 5. API 客戶端規範

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

// 回應攔截器：401 自動處理
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    // 401 未授權：清除 token 並導向登入頁
    if (error.response?.status === 401) {
      localStorage.removeItem('accessToken')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default api
```

---

## 6. 資安規範

> 詳見 [共享規範 SECURITY-CHECKLIST.md](../_shared/SECURITY-CHECKLIST.md)。

### 6.1 React 重點

| 類別 | 規範 |
|------|------|
| **XSS 防護** | React JSX 自動 escape，**禁止 `dangerouslySetInnerHTML`** |
| **Markdown 渲染** | 使用 `react-markdown`（自動 sanitize） |
| **URL 參數** | 使用 `encodeURIComponent()` |
| **CSRF 防護** | JWT Token 透過 Authorization header 傳遞，天然免疫 CSRF |
| **敏感資料** | 禁止在 localStorage 儲存敏感資料；JWT 可考慮 HttpOnly Cookie |
| **環境變數** | 前端環境變數使用 Vite `import.meta.env.VITE_*`；禁止把 Secret 放入前端 |

---

## 7. 代碼品質要求

詳見 [共享規範 §6](../_shared/CODING-STANDARDS.md#6-程式碼品質要求)。

- 元件不超過 300 行（過大應拆分子元件）
- 函式不超過 50 行
- 必須處理 loading / error / empty 三種狀態
- 表單必須有前端驗證 + 後端驗證

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
- [ ] 無 `dangerouslySetInnerHTML`
- [ ] 使用函式元件，無 Class Component
- [ ] 無 `var` 宣告
- [ ] 無 TypeScript（`.tsx` / `.ts`）檔案
- [ ] 處理 loading / error / empty 狀態
- [ ] 前端可正常建置（`npm run build`）

---

## 9. Skill 串接機制（強制執行）

開發完成且建置成功後，必須輸出：

```
【Skill 串接通知】

✅ 開發階段已完成
✅ 前端建置成功（npm run build）
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

此 Skill 將協助您進行：
- React 程式碼品質審查
- XSS 防護檢查
- 繁體中文註解完整度檢查

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview`
- ❌ 其他輸入 → 不啟動
