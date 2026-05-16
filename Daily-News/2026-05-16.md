# OpenClaw 每日情報 2026-05-16

> 資料來源截止時間：2026-05-16（過去 24 小時）

---

## 1. 今日重點摘要

1. **v2026.5.7 正式發布**：最新版本修復了 Cron job 中 `payload.model` 儲存為 `"default"`、`"null"` 或空值的持久化錯誤，`openclaw doctor --fix` 可自動修復既有排程任務。

2. **Telegram 授權邏輯強化**：v2026.5.7 修正了 Telegram DM、群組頻道及 callback 的 `accessGroup:*` 寄件者白名單驗證順序，確保在套用數字 sender-ID 檢查前先通過白名單授權。

3. **Agent 交付狀態修正**：出站交付無 adapter 結果時，系統現在正確回傳 `deliverySucceeded=false`，解決空交付路徑偽裝成成功的問題。

4. **OpenAI `chat-latest` 別名支援**：新增 `openai/chat-latest` 作為直接 API Key 的模型覆寫選項，方便使用者試用 ChatGPT Instant API 移動別名而不影響穩定預設模型。

5. **GitHub Issue #52949：Gateway 未傳遞 ClawHub Auth Token**：回報 Gateway 未正確傳遞 ClawHub auth token 至 API 呼叫，導致持續出現 429 Rate Limit 及「Available Skills」清單空白的問題，目前仍為開放狀態。

6. **Bug 回報 #82347：新版本出現 regression**：5 月 15 日由 kwangwonkoh 提出，標記為 bug + regression，詳情待進一步確認。

7. **PR #82317：Telegram 進度預覽 Lane**：giodl73-repo 於 5 月 15 日提交，為 Telegram 內的 agent 新增進度助手預覽 lane 功能。

8. **PR #82333：SecretRef Plugin Manifest 合約**：joshavant 提交，新增 plugin manifest 合約以支援 SecretRef provider 整合，加強憑證管理機制。

9. **Discord 社群突破 10 萬成員**：官方 Discord「Friends of the Crustacean」在約六週內從零成長至 100,000 成員，速度超過 Midjourney 當年四個月的里程碑。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **OAuth Rate Limit 假錯誤**（Issue #32828）：使用 Claude OAuth 的用戶反映系統顯示「API rate limit reached. Please try again later.」，但外部直接測試 API 完全正常。確認為 OpenClaw 內部 provider cooldown 機制錯誤觸發，與實際 Claude API 狀態無關。
  - 來源：[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828) | [AnswerOverflow 討論](https://www.answeroverflow.com/m/1478788645472436264)

- **Cooldown 原因硬編碼問題**（Issue #23996）：`rate_limit` 原因被硬編碼為唯一 cooldown 觸發原因，無論實際失敗類型為 timeout、billing 或 auth 錯誤，均以同一代碼回報，造成除錯困難。
  - 來源：[GitHub Issue #23996](https://github.com/openclaw/openclaw/issues/23996)

- **Claude Opus 4.7 效能評估**：在 SWE-bench Pro 測試中取得 64.3% 成績，指令遵從性強，自我檢查行為在實際使用中表現突出，適合嚴格程式碼審查工作流。
  - 來源：[DEV Community 比較文章](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### GPT（OpenAI）

- **`openai/chat-latest` 別名支援**：v2026.5.7 新增此選項，讓用戶在不更動穩定設定下試用 ChatGPT Instant API。
- **GPT-5.5 在自主 agent 場景領先**：Terminal-Bench 2.0 基準測試中表現最佳，但限制在 128K tokens 以下的 context。
  - 來源：[DEV Community 比較文章](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### Gemini（Google）

- **Gemini 3.1 Pro 為多數部署的預設推薦**：大 context 程式碼審查、最低成本、免費開發者方案、原生多模態，被視為 OpenClaw 工作流的最佳起點。
- **Google Meet 整合**：OpenClaw 已可加入 Google Meet 會議，透過 Gemini Live 轉錄音訊並匯出出席記錄。
  - 來源：[MindStudio 部落格](https://www.mindstudio.ai/blog/openclaw-april-2026-model-agnostic-provider-manifest)

### DeepSeek

- **DeepSeek V4 Flash / V4 Pro 支援**：適合高量任務且需要可靠推理但不需要前沿模型效能的場景，每 token 成本優勢明顯，受成本敏感部署青睞。
  - 來源：[OpenClaw 官方文件](https://docs.openclaw.ai/providers/deepseek)

### OpenRouter

- **單一 endpoint 路由多 provider**：無需為各 provider 分別設定憑證，以 `openrouter/<author>/<slug>` 格式引用模型，OpenRouter 自動處理路由至 DeepSeek、Mistral 等。
  - 來源：[OpenRouter 整合文件](https://openrouter.ai/docs/guides/guides/openclaw-integration)

### Mistral

- **Devstral 工具呼叫穩定性最佳**：在 Mistral 產品線中，Devstral 在多步驟自主工具鏈情境下表現優於 Mistral Large，適合 OpenClaw agent 工作流。Mistral 模型可透過 OpenRouter 或 haimaker.ai 接入。
  - 來源：[haimaker.ai 部落格](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

---

## 3. 社群反應與討論亮點

### Discord 社群

- **#showcase 頻道政策調整**：官方將 `#showcase` 頻道定位調整為「展示你和 agent 實際完成的事」，包括自動化工作流、實驗，而非自我推廣，社群反應正面。
  - 來源：[OpenClaw.report](https://openclaw.report/news/openclaw-discord-100k-members)

### v2026.4.26 更新引發的 Gateway 崩潰

- 4 月底更新後，大量用戶回報 Discord Gateway 崩潰問題，論壇討論熱烈。社群成員建議回滾至 v2026.4.25 beta 版本作為臨時解法。
  - 來源：[Piunika Web 報導](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)

### X.com 社群動態

- Aakash Gupta 在 X 發文：「This may be the safest way to run OpenClaw.」引發討論，多人探討在本機與雲端環境部署的安全性差異。
  - 來源：[X.com](https://x.com/aakashgupta/status/2022546760804278448)

### 模型切換實戰心得（Medium 文章）

- Mohit Aggarwal 撰文分享「同時串接 OpenClaw + Claude + ChatGPT + Telegram」的實際踩坑紀錄，涵蓋 auth 設定、webhook 衝突及模型切換相容性，獲得廣泛回響。
  - 來源：[Medium 文章](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)

---

## 4. 值得追蹤的後續議題

1. **Issue #52949（Gateway 未傳遞 ClawHub Auth Token）**：影響所有使用 ClawHub 技能的用戶，導致 429 持續發生與技能列表空白，官方尚未給出修復時程。

2. **Issue #82347（Regression Bug）**：5 月 15 日剛提出，標記為 regression，需持續追蹤是否影響最新穩定版本。

3. **PR #82326（OpenAI Auth Provider 統一化）**：sallyom 提交的統一 OpenAI auth provider 選擇器命令 PR，若合併將影響所有 OpenAI 相關設定流程。

4. **`openclaw channels list` 指令重構**：v2026.5.7 將 model auth/usage 詳情移至 `openclaw models auth list`、`openclaw status` 及 `openclaw models list`，使用者需更新既有的自動化腳本與文件。

5. **Diagnostic Liveness Sampler 問題**（Issue #79915）：診斷採樣器在 Gateway 冷啟動期間誤觸警告，影響可觀測性判斷，需關注後續修復進度。

6. **多 provider 混合路由工作流普及趨勢**：社群越來越多人採用「本地 Gemma 4 初步分類 → GPT-5.5 實作 → Claude 最終審查」的分層模型架構，相關最佳實踐文件值得持續關注。

---

*報告生成時間：2026-05-16 | 資料來源：GitHub、DEV Community、Medium、Reddit、Discord、X.com、各技術部落格*
