---
name: 開發
description: Java 資深開發者協助。後端禁用 Lombok、Lambda 與 Stream API，必須手工撰寫所有 Getter/Setter，每行程式碼加繁體中文註解。前端採用 React 18 + TypeScript + Vite + Tailwind CSS 架構，使用函式元件與 Hooks 開發。嚴格遵守 XSS、CSRF、SQL Injection 等資安規範。所有開發必須先完成系統分析，使用者輸入 OKOKYES 後方可啟動開發。
---

# Skill: Full-Stack Developer (Java Backend + React Frontend)

## Usage Trigger
- 當涉及 Java 檔案 (.java) 或 Spring Boot 專案架構時自動啟用。
- 當進行後端業務邏輯開發、POJO 建立或 API 修改時。
- 當涉及 React 元件 (.tsx/.ts)、CSS 或前端開發時。
- 當涉及前端頁面新增、修改或遷移時。
- 當進入測試，則請使用 測試 skill

---

## 1. 開發技術限制

### 1.1 後端 Java 規範
- **設定檔**：統一使用 `application.yml`，禁止使用 `application.properties`。
- **程式語法禁令 (Strict)**：
  - **禁用 Lombok**：嚴禁使用 `@Data`, `@Getter`, `@Setter` 等註解，POJO 必須手動生成 Getter/Setter 與 Constructor。
  - **禁用 Lambda**：禁止使用 Lambda 表達式或 Stream API，統一使用傳統 `for-loop`、`if-else` 與匿名內部類別。

### 1.2 前端技術規範
- **前端框架**：強制使用 React 18 + TypeScript + Vite + Tailwind CSS 架構。
- **元件格式**：使用函式元件（Function Component），禁止使用 Class Component。
- **狀態管理**：全域狀態使用 Zustand，伺服器快取可使用 React Query 或 SWR。
- **TypeScript**：嚴格模式，所有 Props 必須定義型別介面。
- **JavaScript 語法**：前端允許使用箭頭函式 `=>`、`const/let`、解構賦值、Template Literal，禁止使用 `var`。

### 1.3 資料庫異動規範
- **強制使用 Flyway**：所有資料庫結構異動（DDL）必須透過 Flyway migration 管理。
- **禁止直接操作資料庫**：
  - ❌ 禁止直接對資料庫執行 CREATE、ALTER、DROP 等 DDL 語句
  - ❌ 禁止使用 DBeaver、pgAdmin 等工具直接修改資料庫結構
  - ❌ 禁止在程式碼中使用 `spring.jpa.hibernate.ddl-auto=update/create`
- **Migration 規範**：
  - 使用 SQL 格式：`V{版本號}__{描述}.sql`
  - 已執行的 migration 禁止修改

---

## 2. 註解規範
- **逐行註解**：每一行具備邏輯意義的程式碼都必須附上繁體中文註解。
- 註解內容應精確說明該行之邏輯目的。
- JSX 模板中的關鍵邏輯區塊亦須加上 `{/* 註解 */}` 說明。

---

## 3. 代碼品質要求
- 必須包含完整的 Null Check 與異常處理機制。
- 確保程式碼符合資深 SD 的高可讀性與穩定性要求。
- 所有對外 API 必須有輸入驗證（Input Validation）。

### 3.1 Exception 處理與日誌規範（強制）

> ⚠️ **最高原則**：所有 Exception 必須記錄完整的堆疊追蹤（Stack Trace），禁止吞掉例外或僅記錄訊息。

**正確範例**：
```java
try {
    userService.createUser(userDto);
} catch (Exception e) {
    // ✅ 正確：包含錯誤描述與完整堆疊追蹤
    log.error("建立使用者失敗，userId: {}", userDto.getUserId(), e);
    throw new BusinessException("使用者建立失敗", e);
}
```

**Logger 宣告規範**：
```java
// ✅ 正確：使用 SLF4J 宣告 Logger
private static final Logger log = LoggerFactory.getLogger(UserService.class);
```

---

## 4. React 前端開發規範

### 4.1 元件結構規範

```tsx
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
// 型別
import type { User } from '@/types/user'

// ==================== 2. Props 型別定義 ====================
interface UserListPageProps {
  /** 是否為編輯模式 */
  isEdit?: boolean
}

// ==================== 3. 元件定義 ====================
export function UserListPage({ isEdit = false }: UserListPageProps) {
  // ==================== 4. Hooks 呼叫 ====================
  const navigate = useNavigate()
  const { hasPermission } = useAuth()

  // ==================== 5. State ====================
  const [loading, setLoading] = useState(false)
  const [users, setUsers] = useState<User[]>([])

  // ==================== 6. 方法 ====================
  /** 載入使用者列表 */
  const loadUsers = useCallback(async () => {
    setLoading(true)
    try {
      const response = await userApi.getUsers()
      setUsers(response.data)
    } catch (error) {
      console.error('[使用者管理] 載入失敗', error)
    } finally {
      setLoading(false)
    }
  }, [])

  // ==================== 7. Effects ====================
  useEffect(() => {
    loadUsers()
  }, [loadUsers])

  // ==================== 8. Render ====================
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

### 4.2 命名規範

| 項目 | 規範 | 範例 |
|------|------|------|
| **元件檔名** | PascalCase | `UserList.tsx`、`Modal.tsx` |
| **頁面元件** | PascalCase + `Page` 後綴 | `DashboardPage.tsx`、`LoginPage.tsx` |
| **Hook** | camelCase + `use` 前綴 | `useAuth.ts`、`useWebSocket.ts` |
| **API 模組** | camelCase | `auth.ts`、`projects.ts` |
| **Props 介面** | PascalCase + `Props` 後綴 | `UserListProps`、`ModalProps` |
| **型別定義** | PascalCase | `User`、`Project`、`TaskStatus` |
| **CSS class** | Tailwind class 優先 | - |
| **路由 path** | kebab-case | `/projects`、`/quick-start` |

### 4.3 API 客戶端規範

```typescript
// api/client.ts - Axios 實例 + JWT 攔截器
import axios from 'axios'

const api = axios.create({
  baseURL: '/api',
  timeout: 30000,
})

// 請求攔截器：自動帶 JWT
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 回應攔截器：401 自動 refresh token
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    // 401 且非重試：嘗試刷新 Token
    // refresh 失敗：導向 /login
  }
)

export default api
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
- JWT Token 透過 Authorization header，天然免疫 CSRF
- Spring Security 關閉 CSRF

### 5.3 SQL Injection 防護
- 絕對禁止字串拼接 SQL
- 使用 JPA Named Parameter 或 `@Query` 參數綁定

---

## 6. Controller / Service 架構規範

> 每個前端功能模組對應一個專用 RestController，禁止跨 Controller 使用。

| Service 類型 | 判斷標準 | 範例 |
|-------------|---------|------|
| **共用 Service** | 被 2+ 個 Controller 使用 | `UserService`、`NotificationService` |
| **專用 Service** | 只被 1 個 Controller 使用 | `ProjectService`、`WorkflowService` |

---

## 7. 開發流程強制規範

### 7.1 分析階段
1. 需求評估、受影響檔案清單、邏輯變更點、資安風險評估
2. 等待使用者輸入 **`OKOKYES`** 後方可開始開發

### 7.2 啟動開發的唯一條件
- ✅ `OKOKYES` → 啟動開發
- ❌ 「確認」「同意」「OK」「好」→ 不啟動

### 7.3 完成檢查清單

**後端 Java**：
- [ ] 所有程式碼皆有繁體中文註解
- [ ] 無 Lombok、Lambda、Stream API
- [ ] 通過 Null Check 與異常處理檢查
- [ ] 無字串拼接 SQL
- [ ] 專案可正常編譯（`mvn compile`）

**前端 React**：
- [ ] 所有程式碼皆有繁體中文註解
- [ ] TypeScript 嚴格模式通過
- [ ] 無 `dangerouslySetInnerHTML`
- [ ] Props 皆有 TypeScript 型別定義
- [ ] 前端可正常建置（`npm run build`）

---

## 8. Skill 串接機制（強制執行）

開發完成且專案編譯成功後，必須輸出：

```
【Skill 串接通知】

✅ 開發階段已完成
✅ 專案編譯成功
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview`
- ❌ 其他輸入 → 不啟動
