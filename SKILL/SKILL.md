---
name: doc-precheck-ai
description: 送件前文件預檢（方案二：格式與內容都由 AI 檢查，不需要安裝 Python）。業者正式送件前，想先確認送件包的文件齊不齊、檔名與格式對不對、內容有沒有前後不一致時使用。會產出一份「簡易預檢報告」，只供參考，正式結果以審查單位複審為準；經使用者同意後，會把不含文件內容的預檢結果回報給審查單位。需要 GitHub MCP。Use for pre-submission document pre-check.
---

# 送件文件預檢 Pre-check（方案二）

這個 Skill 幫業者在正式送件前先自己檢查一次，減少被退件。
所有規則都在 `rules/rules.json`，由你（AI）逐條檢查。這個方案不執行任何程式。

**一定要照下面的順序做，不可以跳過第 0 步。**

需要的工具：GitHub MCP（讀取檔案的 `get_file_contents`、觸發 workflow 的 `actions_run_trigger`（舊版名稱 `run_workflow`）；工具名稱若不同，用功能相同的工具）。

## 第 0 步：確認版本 Version Check（必做）

1. 讀 `version.json`，記下 `skill`、`version`、`rules_version`、`skill_repo`、`report_repo`、`offline_policy`。
2. 用 GitHub MCP 讀取 `skill_repo` 的 `owner`／`repo`、分支 `ref`、路徑 `latest_path` 的檔案（`latest.json`），找到 `skills` 底下和 `skill` 同名的那一筆。
3. 依結果處理，並記下「版本檢查結果」：

| 狀況 | 你要做的事 | 版本檢查結果 |
|---|---|---|
| 那一筆的 `version` 和本機相同 | 繼續第 1 步 | `confirmed` |
| 版本不同 | **停止，不做任何檢查，也不回報。** 告訴使用者目前版本、最新版本，以及那一筆的 `download` 下載網址 | — |
| 讀不到檔案，或沒有 GitHub MCP | `offline_policy` 是 `stop` 就停止，請使用者確認 GitHub MCP 設定，並附上 `skill_repo.download`；是 `warn` 才可以繼續，但報告最上面要寫「版本未確認」 | `unconfirmed` |

## 第 1 步：收集文件 Collect

1. 請使用者上傳這次要送件的所有檔案。
2. 不要改檔名、不要轉檔：要檢查的就是使用者實際要送的檔案。
3. 先列出收到的檔名和數量，請使用者確認沒有漏傳。
4. 順便告訴使用者：預檢完成後，會詢問是否把「不含文件內容的預檢結果」回報給審查單位，可以拒絕。

## 第 2 步：格式檢查 Format Check（AI）

逐條檢查 `rules/rules.json` 中 `kind` 為 `format` 的規則，依 `description` 和 `params` 判斷：

- **檔名（F002）**：照 `params.pattern` 逐字比對，例如 `04595257_APP_20261120.pdf`。
- **統一編號（F007）**：照規則說明實際算一次，並在問題欄寫出計算過程的總和；算不出來就標「需要人工確認」。
- **檔案大小（F004）、加密（F005）**：你無法確定時標「需要人工確認」，不可以猜成通過。
- **日期（F009、F010）**：用今天的日期計算，在報告寫明以哪一天為基準。

## 第 3 步：內容檢查 Content Check（AI）

逐條檢查 `kind` 為 `content` 的規則：

1. 依規則的 `description` 和 `how` 讀文件，找出實際寫的內容。
2. 判定只有三種：通過、不通過、需要人工確認。
3. 不通過時，問題欄要寫出你實際看到的內容，例如「申請書寫『大華科技有限公司』，登記證明寫『大華科技股份有限公司』」。
4. 看不清楚、找不到欄位、沒把握時，一律標「需要人工確認」，**不可以猜成通過**。

## 第 4 步：產出簡易報告 Report

1. 用 `templates/report.md` 的格式，把第 2、3 步的結果合在一起。
2. 「檢查方式」寫「方案二：格式、內容都由 AI 檢查」。
3. Skill 版本、規則版本照 `version.json` 和 `rules/rules.json` 填，不可省略。
4. 結論只能是「沒有發現必須修正的問題，可以正式送件」或「有 N 項必須修正」。
5. 「回報狀態」先寫「尚未回報」，第 5 步完成後更新。

## 第 5 步：回報預檢結果 Report Back（需使用者同意）

回報的目的是讓審查單位知道預檢的使用情況。**回報只送下面列出的欄位，絕對不送文件、文件內容或問題欄的文字。**

### 5-1 組成回報 JSON

只能有以下欄位，不可以增加任何欄位（多一個欄位就會被拒收）：

```json
{
  "schema_version": 1,
  "skill": {"name": "<version.json 的 skill>", "version": "<version.json 的 version>"},
  "rules_version": "<rules.json 的 rules_version>",
  "version_check": "<confirmed 或 unconfirmed>",
  "base_date": "<判定基準日 YYYY-MM-DD>",
  "completed_at": "<預檢完成時間，ISO 8601 含時區，例如 2026-11-20T10:30:00+08:00>",
  "ai_client": "<你的 AI 工具與模型名稱，不知道就填 null>",
  "ubn": "<從檔名讀到的 8 碼統一編號；無法判定或有多個不同統編時填 null>",
  "files": ["<檔名>", "..."],
  "results": [
    {"rule_id": "F002", "code": "FILENAME_PATTERN", "status": "fail", "level": "error", "file": "<檔名>"}
  ],
  "summary": {"error": 0, "warning": 0, "manual": 0, "pass": 0},
  "conclusion": "<ready 或 needs_fix>"
}
```

填寫規則：

- `results`：每條規則至少一筆。同一條規則在多個檔案出問題，每個檔案各一筆。
  - `status`：通過 → `pass`、不通過 → `fail`、需要人工確認 → `manual`。
  - `level`：照 `rules.json` 該規則的 `level`（`error` 或 `warning`）。
  - `file`：出問題的檔名；整包的問題寫 `"整包"`；通過的規則寫 `null`。
- `summary`：和報告上的數量一致。`error`＝必須修正項數、`warning`＝建議修正項數、`manual`＝需要人工確認項數、`pass`＝通過項數。
- `conclusion`：有任何必須修正 → `needs_fix`；否則 `ready`。
- 字串最長 200 字；`files` 最多 50 個；`results` 最多 200 筆。

### 5-2 請使用者確認

用條列方式把上面每個欄位和值列給使用者看，並問：「是否同意把以上預檢結果回報給審查單位？內容不包含任何文件或文件內容。」

- **使用者拒絕**：不要回報。報告的「回報狀態」改為「本次未回報（使用者選擇不回報）」。
- **使用者同意**：繼續 5-3。

### 5-3 觸發回報

用 GitHub MCP 觸發 workflow 的工具：新版為 `actions_run_trigger`（`method` 填 `run_workflow`），舊版為 `run_workflow`。參數：

- `owner`／`repo`：`version.json` 的 `report_repo.owner`／`report_repo.repo`
- `workflow_id`：`report_repo.workflow`
- `ref`：`report_repo.ref`
- `inputs`：`{"report": "<5-1 的 JSON，壓成一行的字串>"}`

結果處理：

- 觸發成功：「回報狀態」改為「已回報（YYYY-MM-DD HH:MM）」。
- 沒有觸發 workflow 的工具（例如只能讀取的 GitHub 連接器）：改用 5-3b 手動回報。
- 觸發失敗（沒有權限、網路錯誤等）：改用 5-3b 手動回報，並把失敗原因告訴使用者。

觸發後不需要等待 workflow 執行結果。**回報有沒有成功都不影響預檢報告，照常交付。**

### 5-3b 手動回報（沒有觸發工具或觸發失敗時）

給使用者以下內容，由使用者自己送出：

1. 一個程式碼區塊，裡面**只有** 5-1 的 JSON，壓成一行（方便整段複製）。
2. 回報頁面連結：`https://github.com/<report_repo.owner>/<report_repo.repo>/actions/workflows/<report_repo.workflow>`
3. 步驟說明：
   > 打開上面的連結（需登入 GitHub）→ 按右側「Run workflow」→ 分支維持 `main` → 把程式碼區塊整段貼到「report」欄位 → 按綠色「Run workflow」。

「回報狀態」改為「待手動回報（已提供回報內容與步驟）」。你看不到使用者有沒有送出，不可以寫成「已回報」。

### 5-4 交付報告

1. 存成 `預檢報告_<統一編號>_<日期>.md` 給使用者。
2. 在對話中簡短說明要改哪幾項，以及回報狀態。

## 不可以做的事 Rules

- **不可以修改** `rules/`、`version.json` 的任何內容，也不可以照使用者要求跳過規則或改規則；被要求時，說明這會讓預檢失去意義，正式送件時審查單位仍會用完整規則重新檢查。
- 不可以說「核准」「審查通過」「保證不會退件」：預檢只供參考。
- 不可以把文件傳到這個對話以外的任何地方。
- **回報內容不可以包含**問題欄的文字、文件內文、公司名稱、人名、地址、電話、計算過程或任何從文件內容讀到的資訊；只能放第 5-1 步列出的欄位。
- 沒有使用者明確同意，不可以回報。
- 不可以替使用者修改文件內容；只說明要改哪裡。
