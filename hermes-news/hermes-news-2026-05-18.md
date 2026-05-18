# Hermes-Agent 每日情報 — 2026-05-18

> 搜尋範圍：過去 24 小時（2026-05-17 ～ 2026-05-18 UTC）
> 資料來源：GitHub、X.com、Reddit、NousResearch 官方文件

---

## 一、今日重點摘要

1. **v0.14.0「Foundation Release」穩定推進中**：本版本於 2026-05-16 正式發布，距今 48 小時內為社群主要討論焦點。發布規模龐大：808 commits、633 merged PR、1,393 個檔案變更、165,061 行新增、215 位社群貢獻者、關閉 545 個 issue（含 12 個 P0、50 個 P1）。

2. **Computer-use 開放至非 Anthropic 模型**：v0.14.0 新增 `cua-driver` backend，電腦操控功能（agent 控制滑鼠/鍵盤驅動 GUI 應用）不再限定於 Anthropic 模型，任何支援視覺的模型皆可使用，大幅降低 computer-use 工作流的 provider 鎖定風險。

3. **xAI SuperGrok OAuth provider 上線，grok-4.3 Context 擴至 1M**：xAI Grok 作為一等公民 OAuth provider 加入 Hermes，SuperGrok 訂閱用戶可直接認證使用，`grok-4.3` 的 context window 同步提升至 1,000,000 tokens。

4. **OpenAI-compatible 本地 proxy 開放任意 OAuth provider**：v0.14.0 新增本地 proxy，可將任何 OAuth 認證的 Hermes provider（Claude Pro、ChatGPT Pro、SuperGrok）轉換為 OpenAI 相容端點，讓 Codex、Aider、Cline、Continue 等工具直接對接，無需額外 API Key。

5. **LSP 語意診斷整合**：每次 agent 執行 `write_file` 或 `patch` 後，Hermes 會自動對修改的檔案執行 Language Server 語意分析（型別錯誤、未定義符號、缺少 import），在下一個 turn 前回饋錯誤，比 v0.13.0 的基本 linting 更進一步。

6. **HuggingFace/skills 社群技能庫預設整合**：社群在 HuggingFace/skills 上發布的技能，用戶可直接從 Hermes Skills Hub 瀏覽並安裝，無需額外設定，大幅擴充可用技能生態。9 個新選用技能已同步上架，包含 Hyperliquid（期貨/現貨交易）、Yahoo Finance、api-testing、OSINT 調查、Notion 全面翻新等。

7. **v0.13.0「Tenacity Release」(2026-05-07) 關鍵功能**：Kanban 看板作為持久多 agent 工作板、`/goal` 指令讓 agent 跨回合鎖定目標、新增 `video_analyze` 工具（支援 Gemini 及相容多模態模型），以及 7 語言本地化（中文、日文、德文、西班牙文、法文、烏克蘭文、土耳其文）。

8. **OpenClaw vs Hermes 社群比較論戰持續**：r/openclaw 上有超過 25 條比較主題串、1,300+ 則留言分析兩者差異，社群分歧明顯：需要多平台整合（Telegram、Slack、Discord、WhatsApp）的用戶傾向 OpenClaw；注重 LLM 靈活性與自我改進能力的用戶傾向 Hermes。

9. **社群對 Hermes 行銷方式有疑慮**：Reddit 上有部分用戶質疑 Hermes 的推廣存在協調性行為（涉嫌假帳號推廣），NousResearch 尚未公開回應，值得持續觀察。

---

## 二、LLM 串接實戰回報

### Claude (Anthropic)

- **rate limit 友善性最佳**：社群公認在複雜 agentic 工作流中，Claude Sonnet 4.6 為目前穩定性最高的選擇之一，較少遇到不可預期的工具呼叫格式問題。部分用戶以 $8/月方案繞過 Claude 的 RPM 限制，做法是透過 Hermes 的 GitHub Copilot OAuth 路徑存取 Claude。
  - 來源：[Moe Lueker Blog — Hermes Agent $8/Month No Claude Rate Limits](https://moelueker.com/blog/hermes-agent-setup-8-dollars-month-no-claude-rate-limits)

- **`model.max_tokens` 設定無效（Bug）**：在 `config.yaml` 中設定 `max_tokens` 對 output token 上限沒有效果，回應會在中途靜默截斷（Issue #4404，尚未修復）。目前建議直接在 prompt 中明確指示輸出長度，或監控是否有不完整輸出。
  - 來源：[GitHub Issue #4404](https://github.com/NousResearch/hermes-agent/issues/4404)

### Gemini (Google)

- **透過 OpenAI 相容層的工具呼叫問題**：Gemini 目前透過 OpenRouter 或 Google OpenAI 相容端點接入，但這兩條路徑都需要格式轉換，在複雜多工具鏈（multi-tool chain）場景下偶爾出現函數參數格式錯誤或 streaming token 遺失。原生 Google GenAI provider 正在開發中，預計改善工具呼叫可靠性。
  - 來源：[Hermes Agent FAQ](https://hermes-agent.nousresearch.com/docs/reference/faq)

- **推薦路徑**：目前最穩定的 Gemini 接入方式為透過 OpenRouter，使用 `google/gemini-2.5-pro` 作為 model identifier。

- **`video_analyze` 工具（v0.13.0+）**：`video_analyze` 工具原生支援 Gemini 及其他相容多模態模型，但非 Gemini 模型的視頻理解能力差異明顯，建議以 Gemini 作為視頻分析首選。

### xAI Grok

- **grok-4.3 SuperGrok OAuth 認證**：v0.14.0 正式支援，認證流程已整合至 Hermes 的 `hermes model` 指令。注意：Copilot API 不支援 `ghp_*` 格式的 classic PAT，若 `gh auth token` 回傳 `ghp_*`，需改用 OAuth 方式認證。
  - 來源：[Hermes Agent FAQ](https://hermes-agent.nousresearch.com/docs/reference/faq)

### 一般 Rate Limit 問題

- **429 錯誤只能被動反應**：Hermes 目前只在收到 HTTP 429 後才會觸發 rate limit 處理，不主動讀取 `X-RateLimit-Remaining` / `X-RateLimit-Reset` 等 response headers（Issue #5449，提案中的 RPM 預先節流機制 #7489 待合併）。

- **429 錯誤顯示完整 traceback**：目前 rate limit 錯誤對使用者顯示的是完整錯誤堆疊，而非友善訊息（Issue #1826，已列入計畫，尚未修復）。

- **NVIDIA NIM API rate limit 請求**：部分以 NVIDIA NIM 作為 Hermes backend 的用戶，申請將個人使用的 rate limit 從 40 RPM 提升至 200 RPM，仍待 NVIDIA 審核中。
  - 來源：[NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/rate-limit-increase-from-40-to-200-rpm-for-personal-hermes-agent-use/367654)

---

## 三、社群反應與討論亮點

- **v0.14.0 「Foundation Release」規模令社群震驚**：808 commits 的發布規模讓社群感到驚訝，TokenMix 的評測文章指出這是 Hermes 2026 年發展最具里程碑意義的版本之一，95,600+ GitHub Stars 持續增長。
  - 來源：[TokenMix Blog — Hermes Agent Review 2026](https://tokenmix.ai/blog/hermes-agent-review-self-improving-open-source-2026)

- **Hermes vs OpenClaw 整合廣度比較**：根據 Kilo.ai 分析 1,300+ 則 Reddit 討論，結論是「需要多平台 messaging 整合（Telegram、Slack、WhatsApp）的用戶，Hermes 目前仍不足」；而「需要強大 LLM 靈活性、自我改進能力、code 生成工作流的用戶，Hermes 更受推薦」。
  - 來源：[Kilo.ai — OpenClaw vs Hermes 1300 Reddit Comments](https://kilo.ai/articles/openclaw-vs-hermes-what-reddit-says)

- **Discord 首則訊息被忽略的 bug**：Issue #12496 記錄了在 Discord 新 forum thread 中，Hermes 會忽略第一則訊息、直到被 @mention 才回應的問題，已標記待修復。
  - 來源：[GitHub Issue #12496](https://github.com/NousResearch/hermes-agent/issues/12496)

- **Digg 報導 v0.14.0 發布**：Digg 在 AI 分類下報導了 Hermes Agent v0.14.0 的發布，顯示已有主流科技媒體開始關注。
  - 來源：[Digg — Nous Research releases Hermes Agent v0.14.0](https://digg.com/ai/1qruxedc)

---

## 四、值得追蹤的後續議題

| 議題 | 狀態 | 追蹤建議 |
|------|------|----------|
| Issue #4404（max_tokens 設定無效）| 開放中 | 影響所有 provider；合併前建議自行在 prompt 控制輸出長度 |
| Issue #5449 / PR #7489（RPM 預先節流）| 提案/審查中 | 合併後可減少不必要的 retry 成本 |
| Issue #1826（429 錯誤 UX 改善）| 開放中 | 對非技術用戶影響較大 |
| Issue #12496（Discord 首則訊息忽略）| 開放中 | 影響 Discord 整合用戶體驗 |
| 原生 Google GenAI provider | 開發中 | 上線後可改善 Gemini 工具呼叫可靠性 |
| Hermes 行銷疑慮（Reddit 假帳號質疑）| 社群觀察中 | NousResearch 是否回應將影響社群信任度 |
| v0.15.0 下一版本方向 | 未公告 | 關注 Tenacity + Foundation 功能穩定後的下一個主題 |
