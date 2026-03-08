---
name: 重構
description: "程式碼重構專家（支援 Java 與 React + TypeScript）。在不改變業務邏輯的前提下，改善程式碼結構與可維護性。專注解決：(1) 方法/函式/元件過長 (2) 重複程式碼 (3) 高耦合低內聚 (4) 命名不清。採用小步快跑策略，每個重構步驟獨立 commit，確保可回滾。重構前必須確認測試可執行，重構後測試必須通過。"
---

# Skill: 重構（Refactoring Expert）

## 支援語言

| 語言 | 支援狀態 | 說明 |
|------|---------|------|
| **Java** | ✅ 完整支援 | Spring Boot、POJO、Service 層等 |
| **React + TypeScript** | ✅ 完整支援 | React 18 函式元件、Hooks、TypeScript |

## Usage Trigger

- 當使用者說「重構」、「這個方法太長」、「程式碼很亂」、「重複程式碼太多」時觸發
- 當 CodeReview 發現方法過長、重複程式碼等建議項目時，可建議使用此 Skill
- 當需要在新增功能前，先整理既有程式碼時
- 當前端 React 元件過長、需要拆分或提取自訂 Hook 時

---

## 1. 核心原則（必須遵守）

### 1.1 重構黃金守則

| 守則 | 說明 |
|------|------|
| **行為不變** | 重構只改變程式碼結構，絕不改變業務邏輯 |
| **小步快跑** | 每次只做一個小重構，確認沒問題再繼續 |
| **頻繁提交** | 每完成一個重構步驟立即 commit |
| **測試先行** | 重構前確認測試可執行，重構後測試必須通過 |
| **可回滾** | 每個 commit 都是可獨立回滾的單位 |

### 1.2 禁止事項

- ❌ 禁止在重構過程中新增功能
- ❌ 禁止在重構過程中修復 bug（除非是重構導致的）
- ❌ 禁止一次重構太多東西
- ❌ 禁止在沒有測試覆蓋的情況下進行大規模重構
- ❌ 禁止重構後不執行測試就宣告完成

---

## 2. 重構流程

### Phase 1: 掃描與識別（必須執行）

#### 1.1 掃描目標程式碼

**Java 檔案掃描**：
```bash
# 掃描 Java 檔案結構
find src/main/java -name "*.java" -type f | head -50

# 計算方法行數（識別過長方法）
grep -n "public\|private\|protected" --include="*.java" [目標檔案]

# 搜尋重複程式碼模式
grep -rn "[疑似重複的程式碼片段]" --include="*.java" src/
```

**React + TypeScript 檔案掃描**：
```bash
# 掃描前端元件檔案結構
find src -name "*.tsx" -o -name "*.ts" | head -50

# 計算元件/函式行數（識別過長元件）
grep -n "export function\|export const\|const.*=.*(" --include="*.tsx" --include="*.ts" [目標檔案]

# 搜尋重複程式碼模式
grep -rn "[疑似重複的程式碼片段]" --include="*.tsx" --include="*.ts" src/

# 統計檔案行數
wc -l src/**/*.tsx src/**/*.ts | sort -rn | head -10

# 搜尋可提取的自訂 Hook
grep -rn "useState\|useEffect\|useCallback" --include="*.tsx" src/ | cut -d: -f1 | sort | uniq -c | sort -rn
```

#### 1.2 識別程式碼壞味道

> 完整清單請參考 [PATTERNS.md](PATTERNS.md)

**Java 優先處理的壞味道**：

| 優先級 | 壞味道 | 識別方式 |
|-------|-------|---------|
| 🔴 P0 | 方法過長（> 50 行） | 計算方法行數 |
| 🔴 P0 | 重複程式碼 | 搜尋相似程式碼區塊 |
| 🟡 P1 | 過深巢狀（> 3 層） | 檢查 if/for 巢狀層數 |
| 🟡 P1 | 過長參數列（> 5 個） | 檢查方法簽章 |
| 🟢 P2 | 命名不清 | 人工審查 |
| 🟢 P2 | 註解過時或錯誤 | 人工審查 |

**React + TypeScript 優先處理的壞味道**：

| 優先級 | 壞味道 | 識別方式 |
|-------|-------|---------|
| 🔴 P0 | 元件過長（> 200 行） | 計算元件行數 |
| 🔴 P0 | 重複程式碼 | 搜尋相似程式碼區塊 |
| 🔴 P0 | 使用 `any` 型別 | 搜尋 `: any` |
| 🟡 P1 | 重複的狀態邏輯 | 多個元件有相同的 useState + useEffect 組合 |
| 🟡 P1 | 過深巢狀條件渲染 | 檢查 JSX 中的三元運算子/&& 巢狀層數 |
| 🟡 P1 | Props 傳遞過深（Prop Drilling） | 檢查 Props 層層傳遞的層數 |
| 🟡 P1 | 魔術數字/字串 | 搜尋未命名的常數 |
| 🟢 P2 | 命名不清 | 人工審查 |
| 🟢 P2 | 過多 console.log | 搜尋 `console.log` 數量 |

#### 1.3 產出掃描報告

```
【程式碼掃描報告】

📁 掃描範圍：[檔案/目錄]

🔴 高優先（必須重構）：
┌──────────────────────────┬──────────┬─────────────────────────┐
│ 檔案:位置                 │ 壞味道   │ 說明                    │
├──────────────────────────┼──────────┼─────────────────────────┤
│ UserService.java:45-150  │ 方法過長 │ processUser() 共 105 行 │
│ OrderService.java:23-45  │ 重複程式碼│ 與 UserService 相似     │
└──────────────────────────┴──────────┴─────────────────────────┘

🟡 中優先（建議重構）：
[同上格式]

🟢 低優先（可選重構）：
[同上格式]

📊 統計：
├── 🔴 高優先：{n} 項
├── 🟡 中優先：{n} 項
└── 🟢 低優先：{n} 項
```

---

### Phase 2: 重構規劃

#### 2.1 決定重構範圍與順序

根據掃描報告，規劃重構順序：

1. **優先處理高風險項目**：方法過長、重複程式碼
2. **由內而外**：先重構被依賴的底層方法
3. **相關項目一起處理**：同一類別的問題盡量一起重構

#### 2.2 產出重構計畫

```
【重構計畫】

📋 重構範圍：[檔案清單]

🎯 重構目標：
1. [目標 1：例如將 processUser() 從 105 行縮減到 30 行以內]
2. [目標 2：例如消除 UserService 與 OrderService 的重複程式碼]

📝 重構步驟：

Step 1: [重構項目名稱]
├── 檔案：[檔案路徑]
├── 手法：[使用的重構手法，如 Extract Method]
├── 說明：[具體做什麼]
└── 預期：[重構後的效果]

Step 2: [重構項目名稱]
├── 檔案：[檔案路徑]
├── 手法：[重構手法]
├── 說明：[具體做什麼]
└── 預期：[重構後的效果]

[更多步驟...]

⚠️ 風險提示：
- [可能的風險與應對方式]

請確認以上計畫，輸入「OKOKYES」開始執行重構。
```

⏸️ **停止點：等待使用者輸入 OKOKYES 後才開始執行重構**

---

### Phase 3: 執行重構

#### 3.1 重構前準備

**Java 專案**：
```bash
# 確認目前工作區乾淨
git status

# 確認測試可執行（若有測試）
./mvnw test -Dtest=[相關測試類別]
```

**React + TypeScript 前端**：
```bash
# 確認目前工作區乾淨
git status

# 確認 TypeScript 編譯通過
npx tsc --noEmit

# 若有前端測試框架
npm test -- [相關測試檔案]
```

#### 3.2 執行重構（小步快跑）

**每個重構步驟的執行流程**：

```
1. 執行單一重構手法
   ↓
2. 確認編譯通過
   ↓
3. 執行相關測試（若有）
   ↓
4. 立即 commit
   ↓
5. 進行下一個重構步驟
```

**Commit 訊息格式**：

```bash
git commit -m "refactor: [重構說明]

- [具體變更 1]
- [具體變更 2]

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

#### 3.3 重構手法對照表

> 完整手法請參考 [PATTERNS.md](PATTERNS.md)

**Java**：

| 壞味道 | 重構手法 | 說明 |
|-------|---------|------|
| 方法過長 | Extract Method | 將程式碼區塊提取為獨立方法 |
| 重複程式碼 | Extract Method → Move Method | 提取後移到共用位置 |
| 過深巢狀 | Replace Nested Conditional with Guard Clauses | 用衛述句取代巢狀 |
| 過長參數列 | Introduce Parameter Object | 將參數封裝為物件 |
| 資料泥團 | Extract Class | 將相關欄位提取為新類別 |

**React + TypeScript**：

| 壞味道 | 重構手法 | 說明 |
|-------|---------|------|
| 元件過長 | Extract Component | 將 JSX 區塊提取為獨立子元件 |
| 重複狀態邏輯 | Extract Custom Hook | 將 useState + useEffect 提取為自訂 Hook |
| 過深條件渲染 | Simplify Conditional Rendering | 用早期返回或 renderContent 函式取代巢狀三元 |
| Props 過多 | Introduce Props Interface | 將相關 Props 分組或使用組合元件 |
| Prop Drilling | Extract Context / Zustand Store | 使用全域狀態取代層層傳遞 |
| 魔術數字 | Extract Constants | 提取為具名常數 |

---

### Phase 4: 驗證與完成

#### 4.1 重構後檢查

> 完整檢查清單請參考 [CHECKLIST.md](CHECKLIST.md)

**Java 專案**：
```bash
# 確認編譯通過
./mvnw compile

# 執行所有相關測試
./mvnw test

# 檢查是否有遺漏的 commit
git status
```

**React + TypeScript 前端**：
```bash
# 確認 TypeScript 編譯通過
npx tsc --noEmit

# 確認前端可正常建置
npm run build

# 檢查是否有遺漏的 commit
git status
```

#### 4.2 產出重構報告

```
【重構完成報告】

✅ 重構完成

📊 重構統計：
├── 重構檔案數：{n}
├── 重構步驟數：{n}
├── Commit 數量：{n}
└── 測試結果：通過/失敗

📝 重構摘要：

| 項目 | 重構前 | 重構後 | 改善 |
|-----|-------|-------|------|
| processUser() 行數 | 105 行 | 28 行 | -73% |
| 重複程式碼區塊 | 5 處 | 0 處 | -100% |

🔧 主要變更：
1. [變更說明 1]
2. [變更說明 2]

📋 Commit 清單：
1. refactor: [commit 訊息 1]
2. refactor: [commit 訊息 2]
```

---

## 3. 常用重構手法詳解

### 3.1 Extract Method（提取方法）

**適用情境**：方法過長、程式碼區塊可獨立命名

```java
// ❌ 重構前
public void processOrder(Order order) {
    // 驗證訂單 - 開始
    if (order == null) {
        throw new IllegalArgumentException("訂單不可為空");
    }
    if (order.getItems() == null || order.getItems().isEmpty()) {
        throw new IllegalArgumentException("訂單項目不可為空");
    }
    if (order.getCustomerId() == null) {
        throw new IllegalArgumentException("客戶編號不可為空");
    }
    // 驗證訂單 - 結束

    // 後續處理...
}

// ✅ 重構後
public void processOrder(Order order) {
    // 驗證訂單資料
    validateOrder(order);

    // 後續處理...
}

/**
 * 驗證訂單資料完整性
 * @param order 待驗證的訂單
 * @throws IllegalArgumentException 當訂單資料不完整時
 */
private void validateOrder(Order order) {
    // 檢查訂單是否為空
    if (order == null) {
        throw new IllegalArgumentException("訂單不可為空");
    }
    // 檢查訂單項目是否為空
    if (order.getItems() == null || order.getItems().isEmpty()) {
        throw new IllegalArgumentException("訂單項目不可為空");
    }
    // 檢查客戶編號是否為空
    if (order.getCustomerId() == null) {
        throw new IllegalArgumentException("客戶編號不可為空");
    }
}
```

### 3.2 Extract Utility Class（提取工具類別）

**適用情境**：多個類別有相同的程式碼

```java
// ❌ 重構前：重複的驗證邏輯散落各處
public class UserService {
    public void saveUser(User user) {
        if (user == null) {
            throw new IllegalArgumentException("user 不可為 null");
        }
        // ...
    }
}

public class OrderService {
    public void saveOrder(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("order 不可為 null");
        }
        // ...
    }
}

// ✅ 重構後：提取共用工具類別
public class ValidationUtils {

    /**
     * 檢查物件不可為 null
     * @param obj 待檢查的物件
     * @param fieldName 欄位名稱（用於錯誤訊息）
     * @throws IllegalArgumentException 當物件為 null 時
     */
    public static void requireNonNull(Object obj, String fieldName) {
        // 檢查物件是否為 null
        if (obj == null) {
            // 拋出例外並附上欄位名稱
            throw new IllegalArgumentException(fieldName + " 不可為 null");
        }
    }
}

public class UserService {
    public void saveUser(User user) {
        // 驗證使用者物件不可為空
        ValidationUtils.requireNonNull(user, "user");
        // ...
    }
}
```

### 3.3 Replace Nested Conditional with Guard Clauses（衛述句）

**適用情境**：過深的 if-else 巢狀

```java
// ❌ 重構前：過深巢狀
public String getPaymentStatus(Order order) {
    String result = "";
    if (order != null) {
        if (order.getPayment() != null) {
            if (order.getPayment().isCompleted()) {
                result = "已付款";
            } else {
                result = "待付款";
            }
        } else {
            result = "無付款資訊";
        }
    } else {
        result = "訂單不存在";
    }
    return result;
}

// ✅ 重構後：使用衛述句
public String getPaymentStatus(Order order) {
    // 檢查訂單是否存在
    if (order == null) {
        return "訂單不存在";
    }
    // 檢查是否有付款資訊
    if (order.getPayment() == null) {
        return "無付款資訊";
    }
    // 檢查付款是否完成
    if (order.getPayment().isCompleted()) {
        return "已付款";
    }
    // 預設為待付款狀態
    return "待付款";
}
```

---

## 4. React + TypeScript 重構手法詳解

### 4.1 Extract Component（提取元件）

**適用情境**：元件過長、JSX 區塊可獨立命名

```tsx
// ❌ 重構前：元件過長，所有內容擠在一個元件中
export function UserListPage() {
  const [users, setUsers] = useState<User[]>([])
  const [loading, setLoading] = useState(false)

  return (
    <div className="p-6">
      {/* 搜尋列 - 很長的 JSX */}
      <div className="flex gap-4 mb-4">
        <input className="..." placeholder="搜尋使用者" />
        <select className="...">
          <option value="all">全部</option>
          <option value="active">啟用</option>
        </select>
        <button className="...">搜尋</button>
      </div>
      {/* 資料表格 - 更長的 JSX */}
      <table className="...">
        {/* 50+ 行的表格渲染 */}
      </table>
    </div>
  )
}

// ✅ 重構後：提取為獨立子元件
// 搜尋列元件的 Props 型別
interface UserSearchBarProps {
  /** 搜尋關鍵字變更回呼 */
  onSearch: (keyword: string, status: string) => void
}

// 搜尋列元件
function UserSearchBar({ onSearch }: UserSearchBarProps) {
  return (
    <div className="flex gap-4 mb-4">
      <input className="..." placeholder="搜尋使用者" />
      <select className="...">
        <option value="all">全部</option>
        <option value="active">啟用</option>
      </select>
      <button className="...">搜尋</button>
    </div>
  )
}

// 主頁面元件（精簡後）
export function UserListPage() {
  const [users, setUsers] = useState<User[]>([])

  return (
    <div className="p-6">
      {/* 搜尋列 */}
      <UserSearchBar onSearch={handleSearch} />
      {/* 資料表格 */}
      <UserTable users={users} />
    </div>
  )
}
```

### 4.2 Extract Custom Hook（提取自訂 Hook）

**適用情境**：多個元件有相同的狀態邏輯（如載入資料、分頁、表單處理）

```tsx
// ❌ 重構前：多個元件重複相同的載入邏輯
export function UserListPage() {
  const [users, setUsers] = useState<User[]>([])
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const loadUsers = useCallback(async () => {
    setLoading(true)
    setError(null)
    try {
      const response = await userApi.getUsers()
      setUsers(response.data)
    } catch (err) {
      setError('載入失敗')
    } finally {
      setLoading(false)
    }
  }, [])

  useEffect(() => { loadUsers() }, [loadUsers])
  // ...
}

// ProjectListPage 也有幾乎一樣的邏輯...

// ✅ 重構後：提取為通用自訂 Hook
/** 通用非同步資料載入 Hook */
function useAsyncData<T>(fetchFn: () => Promise<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const load = useCallback(async () => {
    setLoading(true)
    setError(null)
    try {
      const result = await fetchFn()
      setData(result)
    } catch (err) {
      setError('載入失敗')
    } finally {
      setLoading(false)
    }
  }, [fetchFn])

  useEffect(() => { load() }, [load])

  return { data, loading, error, reload: load }
}

// 使用自訂 Hook（精簡後）
export function UserListPage() {
  const { data: users, loading, error } = useAsyncData(userApi.getUsers)
  // ...
}
```

### 4.3 Extract Constants（提取常數）

**適用情境**：魔術數字、魔術字串

```tsx
// ❌ 重構前：魔術數字散落各處
export function PasswordForm() {
  const validate = (password: string) => {
    if (password.length < 8) return '密碼長度至少 8 個字元'
    if (password.length > 20) return '密碼長度不可超過 20 個字元'
  }

  useEffect(() => {
    const timer = setInterval(checkStatus, 30000)
    return () => clearInterval(timer)
  }, [])
}

// ✅ 重構後：提取為常數
// 密碼最小長度
const PASSWORD_MIN_LENGTH = 8
// 密碼最大長度
const PASSWORD_MAX_LENGTH = 20
// 狀態檢查間隔（毫秒）
const STATUS_CHECK_INTERVAL = 30000

export function PasswordForm() {
  const validate = (password: string) => {
    // 檢查密碼最小長度
    if (password.length < PASSWORD_MIN_LENGTH) {
      return `密碼長度至少 ${PASSWORD_MIN_LENGTH} 個字元`
    }
    // 檢查密碼最大長度
    if (password.length > PASSWORD_MAX_LENGTH) {
      return `密碼長度不可超過 ${PASSWORD_MAX_LENGTH} 個字元`
    }
  }

  useEffect(() => {
    // 設定狀態檢查定時器
    const timer = setInterval(checkStatus, STATUS_CHECK_INTERVAL)
    return () => clearInterval(timer)
  }, [])
}
```

### 4.4 Simplify Conditional Rendering（簡化條件渲染）

**適用情境**：JSX 中過深的三元運算子巢狀

```tsx
// ❌ 重構前：過深的條件渲染巢狀
return (
  <div>
    {loading ? (
      <Spinner />
    ) : error ? (
      <ErrorMessage message={error} />
    ) : users.length === 0 ? (
      <EmptyState />
    ) : (
      <UserTable users={users} />
    )}
  </div>
)

// ✅ 重構後：提取為早期返回或獨立函式
// 渲染內容區塊
const renderContent = () => {
  // 載入中狀態
  if (loading) return <Spinner />
  // 錯誤狀態
  if (error) return <ErrorMessage message={error} />
  // 空資料狀態
  if (users.length === 0) return <EmptyState />
  // 正常資料顯示
  return <UserTable users={users} />
}

return <div>{renderContent()}</div>
```

---

## 5. 技術規範遵守

### 5.1 Java 技術規範

重構過程中必須遵守專案的技術規範：

| 規範 | 要求 |
|------|------|
| **Lombok** | 禁止使用，必須手寫 Getter/Setter |
| **Lambda** | 禁止使用，改用傳統 for-loop |
| **Stream API** | 禁止使用，改用傳統迭代 |
| **註解** | 每行邏輯程式碼必須有繁體中文註解 |

### 5.2 React + TypeScript 技術規範

| 規範 | 要求 |
|------|------|
| **箭頭函式** | 允許使用 `=>` 箭頭函式 |
| **變數宣告** | 使用 `const`/`let`，禁止 `var` |
| **解構賦值** | 允許使用解構賦值 |
| **模板字串** | 允許使用模板字串 `` ` ` `` |
| **元件格式** | 使用函式元件，禁止 Class Component |
| **TypeScript** | 嚴格模式，Props 必須定義型別介面，禁止 `any` |
| **註解** | 每行邏輯程式碼必須有繁體中文註解 |
| **console.log** | 正式環境應移除或改用 console.error |

**React 禁用語法檢查**：
```bash
# 檢查是否誤用 var
grep -rn "\bvar\b" --include="*.tsx" --include="*.ts" src/ | grep -v "//\|/\*\|node_modules"

# 檢查是否使用 Class Component
grep -rn "extends.*Component" --include="*.tsx" src/ | grep -v "//\|/\*"

# 檢查是否使用 any 型別
grep -rn ":\s*any\b" --include="*.tsx" --include="*.ts" src/ | grep -v "//\|/\*"

# 檢查是否使用 dangerouslySetInnerHTML
grep -rn "dangerouslySetInnerHTML" --include="*.tsx" src/
```

---

## 6. Skill 串接機制（強制執行）

### 6.1 重構完成 → CodeReview

當重構完成且編譯測試通過後：

```
【Skill 串接通知】

✅ 重構階段已完成
✅ 編譯通過
✅ 測試通過（若有執行）
✅ 已找到對應的 Skill：「CodeReview」(/CodeReview)

此 Skill 將協助您進行：
- 確認重構後的程式碼符合規範
- 檢查是否有遺漏的註解
- 確認無禁用語法（Lombok/Lambda/Stream）

請輸入「OKOKYES」啟動「CodeReview」Skill，或輸入其他內容繼續對話。
```

**等待使用者確認**：
- ✅ 使用者輸入 `OKOKYES` → 立即執行 `/CodeReview` skill
- ❌ 使用者輸入其他內容 → 不啟動，繼續正常對話

### 6.2 從 CodeReview 進入重構

當 CodeReview 發現以下建議項目時，可提示使用重構 Skill：

- 方法超過 50 行
- 發現重複程式碼
- 巢狀層數過深
- 類別過於龐大

```
【建議使用重構 Skill】

CodeReview 發現以下可重構項目：
├── UserService.java: processUser() 方法共 85 行
└── OrderService.java: 與 UserService 有 3 處重複程式碼

建議使用「重構」Skill (/重構) 進行程式碼改善。
請輸入「OKOKYES」啟動「重構」Skill，或輸入其他內容繼續對話。
```

---

## 7. 禁止行為

- ❌ 禁止未掃描就開始重構
- ❌ 禁止未經使用者確認就執行重構
- ❌ 禁止一次重構多個不相關的項目
- ❌ 禁止重構後不執行測試
- ❌ 禁止重構過程中偷偷修改業務邏輯
- ❌ 禁止接受 `OKOKYES` 以外的確認詞彙啟動重構

---

## 8. 相關文件

- [PATTERNS.md](PATTERNS.md) - 程式碼壞味道與重構手法對照表
- [CHECKLIST.md](CHECKLIST.md) - 重構前後檢查清單
