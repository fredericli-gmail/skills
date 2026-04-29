# GitHub Models 審查引擎使用規範

> 本文件為 CodeReview Skill 的 **GitHub Models 引擎** 專屬規範。
> 主規範請見 [SKILL.md](SKILL.md)，三級分類檢查項目請見 [CHECKLIST.md](CHECKLIST.md)。

---

## 1. 引擎概覽

| 項目 | 說明 |
|------|------|
| **服務名稱** | GitHub Models（非 GitHub Copilot Code Review） |
| **API Endpoint** | `https://models.github.ai/inference/chat/completions` |
| **預設模型** | `openai/gpt-4o`（可用環境變數 `COPILOT_REVIEW_MODEL` 覆寫） |
| **腳本位置** | `~/Copilota/code_review.py` |
| **認證方式** | GitHub Personal Access Token（環境變數 `GITHUB_TOKEN` 或 `GH_TOKEN`） |

---

## 2. 前置條件檢查（啟動前強制）

在啟動 GitHub Models 引擎前，**必須依序檢查**：

```bash
# 1. 檢查腳本是否存在
test -f ~/Copilota/code_review.py || echo "❌ 腳本不存在"

# 2. 檢查 Python 3
command -v python3 || echo "❌ 未安裝 Python 3"

# 3. 檢查 Token（不顯示 token 內容）
test -n "${GH_TOKEN:-${GITHUB_TOKEN}}" || echo "❌ 未設定 GITHUB_TOKEN / GH_TOKEN"
```

**任一項失敗 → 自動 fallback 至 Claude 內建引擎，並提示使用者**。

---

## 3. 呼叫介面

### 3.1 標準呼叫

```bash
git diff HEAD | python3 ~/Copilota/code_review.py \
    --context "src/UserController.java,src/UserService.java" \
    --related "src/dto/UserDTO.java" \
    --model "openai/gpt-4o"
```

### 3.2 參數說明

| 參數 | 必要 | 說明 |
|------|------|------|
| `stdin` | ✅ | `git diff` 內容（必須由 stdin 提供） |
| `--context` | 建議 | 受影響檔案完整路徑（逗號分隔），用於完整審查 |
| `--related` | 可選 | 相關依賴檔案（逗號分隔），僅供 LLM 參考上下文 |
| `--model` | 可選 | 覆寫預設模型 |
| `--max-chars` | 可選 | 輸入字元上限（預設 200,000） |

### 3.3 環境變數

| 變數 | 必要 | 說明 |
|------|------|------|
| `GITHUB_TOKEN` 或 `GH_TOKEN` | ✅ | GitHub Personal Access Token |
| `COPILOT_REVIEW_MODEL` | 可選 | 覆寫預設模型 |

---

## 4. 輸出格式（JSON Schema）

### 4.1 成功回應（exit code 0）

```json
{
  "summary": {
    "errors": 2,
    "warnings": 3,
    "suggestions": 1
  },
  "errors": [
    {
      "file": "src/UserService.java",
      "line": 45,
      "type": "Lombok",
      "message": "發現 @Data 註解，違反專案禁用 Lombok 規範",
      "fix": "移除 @Data，改為手動撰寫 getter/setter/toString/equals/hashCode"
    }
  ],
  "warnings": [
    {
      "file": "src/UserController.java",
      "line": 78,
      "type": "Null Check",
      "message": "user.getName() 前未進行 null 檢查",
      "fix": "加入 if (user != null) 包裹後續邏輯"
    }
  ],
  "suggestions": [
    {
      "file": "src/UserService.java",
      "line": 120,
      "type": "Method Length",
      "message": "方法長度達 65 行，超過建議 50 行上限",
      "fix": "拆分為兩個職責更單一的子方法"
    }
  ],
  "_meta": {
    "model": "openai/gpt-4o",
    "input_chars": 12450,
    "context_files": ["src/UserService.java", "src/UserController.java"],
    "related_files": ["src/dto/UserDTO.java"]
  }
}
```

### 4.2 錯誤回應

| Exit Code | error 欄位 | 處置 |
|-----------|-----------|------|
| 1 | `missing_token` | 提示使用者設定環境變數，fallback 至 Claude 引擎 |
| 1 | `empty_diff` | 提示使用者本次無異動，跳過審查 |
| 2 | `invalid_response` | 顯示 raw_response，fallback 至 Claude 引擎 |
| 3 | `http_error` | 顯示錯誤碼與訊息，fallback 至 Claude 引擎 |
| 4 | `exception` | 顯示例外類型與訊息，fallback 至 Claude 引擎 |

---

## 5. SKILL 整合流程

### 5.1 收集審查素材

```bash
# 取得異動檔案清單
CHANGED=$(git diff --name-only HEAD)

# 組合為逗號分隔字串
CONTEXT=$(echo "$CHANGED" | tr '\n' ',' | sed 's/,$//')

# 取得 diff
git diff HEAD > /tmp/code_review_diff.patch
```

### 5.2 執行審查

```bash
cat /tmp/code_review_diff.patch | python3 ~/Copilota/code_review.py \
    --context "$CONTEXT" \
    > /tmp/code_review_result.json 2> /tmp/code_review_stderr.log

REVIEW_EXIT=$?
```

### 5.3 解析結果並格式化為 SKILL 報告

| Exit Code | 動作 |
|-----------|------|
| 0 | 解析 JSON → 套用 SKILL.md §3 報告格式 |
| 1, 2, 3, 4 | 顯示錯誤 → 詢問使用者是否 fallback 至 Claude 引擎 |

---

## 6. 報告呈現規則

GitHub Models 引擎輸出後，**必須**：

1. **顯示原始 JSON 摘要**（讓使用者了解 LLM 觀點）
2. **重新格式化為 SKILL.md §3 表格**（保持與 Claude 引擎一致的視覺體驗）
3. **附註引擎來源**：
   ```
   📌 審查引擎：GitHub Models（model=openai/gpt-4o）
   ```

---

## 7. 安全紅線

- ❌ **絕對禁止**將 PAT token 寫入任何檔案（包含本 SKILL、settings.json、commit log）
- ❌ **絕對禁止**在訊息或日誌中印出 token 完整字串
- ❌ **絕對禁止**將使用者程式碼上傳至 GitHub Models 以外的服務
- ✅ **必須**僅從環境變數讀取 token
- ✅ **必須**在 token 不存在時主動引導使用者設定，而非要求其提供

---

## 8. 模型選擇參考

| 模型 | 適用場景 | 備註 |
|------|---------|------|
| `openai/gpt-4o` | **預設**，平衡速度與品質 | 推薦 |
| `openai/gpt-4o-mini` | 快速初篩 | 程式碼審查能力較弱 |
| `mistral-ai/Codestral-2501` | 程式碼專長 | 非繁中輸出可能較差 |
| `meta/Llama-3.3-70B-Instruct` | 替代方案 | token 上限不同 |

切換方式：
```bash
export COPILOT_REVIEW_MODEL="openai/gpt-4o-mini"
```

---

## 9. 故障排除

| 症狀 | 可能原因 | 解決方式 |
|------|---------|---------|
| `missing_token` | 未設定環境變數 | `export GITHUB_TOKEN=<token>` 並寫入 `~/.zshrc` |
| `http_error 401` | Token 過期 / 權限不足 | 重新建立 PAT，確認勾選 `models:read` 權限 |
| `http_error 429` | 觸發 Rate Limit | 等待數分鐘或更換模型 |
| `invalid_response` | 模型未遵循 JSON 格式 | 已內建 regex 擷取救援；仍失敗則 fallback Claude |
| 輸入截斷警告 | 程式碼超過 200K 字元 | 縮小 `--related` 範圍或調高 `--max-chars` |
