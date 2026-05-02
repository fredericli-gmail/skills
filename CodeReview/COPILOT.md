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
| **呼叫方式** | 由 Claude 直接以 `curl`（mac/linux/windows-bash）或 `Invoke-RestMethod`（windows-ps）呼叫 API，**無外部腳本或 .py / .sh 檔案依賴** |
| **認證方式** | GitHub Personal Access Token（環境變數 `GITHUB_TOKEN` 或 `GH_TOKEN`） |

---

## 2. 前置條件檢查（啟動前強制）

在啟動 GitHub Models 引擎前，**必須依序檢查**：

```bash
# 1. 檢查 Token 環境變數（不顯示 token 內容）
test -n "${GH_TOKEN:-${GITHUB_TOKEN}}" || echo "❌ 未設定 GITHUB_TOKEN / GH_TOKEN"

# 2. 檢查工具鏈
#    mac / linux / windows-bash：需 curl + jq
#    windows-ps：需 PowerShell 5.1+（內建 Invoke-RestMethod / ConvertFrom-Json）
command -v curl && command -v jq || echo "❌ 未安裝 curl 或 jq"
```

**任一項失敗 → 自動 fallback 至 Claude 內建引擎，並提示使用者**（依 OS 給出對應修復指令，詳見 [SKILL.md](SKILL.md) §1.2.2）。

---

## 3. 呼叫介面

實際的 API 呼叫指令（含 system prompt、user prompt 組裝、jq / Invoke-RestMethod 解析）由 Claude 依當下作業系統動態組合，完整樣板請見 [SKILL.md](SKILL.md) §1.3.B「GitHub Models 引擎（依 OS 分流）」。

### 3.1 必要環境變數

| 變數 | 必要 | 說明 |
|------|------|------|
| `GITHUB_TOKEN` 或 `GH_TOKEN` | ✅ | GitHub Personal Access Token，需勾選 `models:read` 權限 |
| `COPILOT_REVIEW_MODEL` | 可選 | 覆寫預設模型（預設 `openai/gpt-4o`） |

### 3.2 輸入內容組裝原則

- **diff 模式**（範圍 `[1]`–`[3]`）：以 `git diff` 結果作為主要審查素材
- **整檔案模式**（範圍 `[4]`–`[5]`）：無 diff，改以完整檔案內容餵入，token 消耗較大
- 系統提示詞（system prompt）內含 [CHECKLIST.md](CHECKLIST.md) 的三級分類規則摘要
- 使用者提示詞（user prompt）為實際待審查的程式碼或 diff

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

完整流程（環境偵測、Prompt 組合、API 呼叫、結果解析）統一由 [SKILL.md](SKILL.md) §1.3.B 規範，本檔不重述以避免雙頭規範。下列為流程要點與此檔的銜接關係：

| 階段 | 規範來源 | 重點 |
|------|---------|------|
| 環境偵測（OS、工具鏈） | SKILL.md §1.2.0 | 偵測 `$TARGET_OS` 決定使用 curl 或 Invoke-RestMethod |
| 前置檢查（token、工具） | SKILL.md §1.2.2 | 任一項失敗即 fallback 至 Claude 引擎 |
| Prompt 組合 | SKILL.md §1.3.B「共通 System Prompt 範本」 | system prompt 含 CHECKLIST.md 規則摘要 |
| API 呼叫 | SKILL.md §1.3.B（依 OS 分流） | 內嵌 curl 或 Invoke-RestMethod，**無外部腳本** |
| 結果解析 | 本檔 §4「輸出格式 JSON Schema」 | LLM 回應需符合 §4.1 結構，否則視為 invalid_response |
| 報告呈現 | 本檔 §6 + SKILL.md §3 | 重新格式化為三級分類表格 |
| 故障處置 | 本檔 §9 + SKILL.md §1.2.2 fallback 段 | 顯示原因並切換至 Claude 引擎 |

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
| 輸入過大 / 模型 token 超限 | 程式碼或 diff 過長 | 縮小審查範圍（如改選範圍 `[1]` 僅看當前 diff），或切換至 token 上限更高的模型 |
