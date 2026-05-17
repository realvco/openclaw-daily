# OpenClaw 每日情報 2026-04-26

> 資料涵蓋區間：2026-04-25 ～ 2026-04-26
> 資料來源：GitHub openclaw/openclaw、releasebot.io、Reddit r/openclaw、X.com、answeroverflow.com

---

## 1. 今日重點摘要

1. **v2026.4.23 正式發布**：加入 xAI 圖像生成（grok-imagine-image / grok-imagine-image-pro）、語音合成（TTS，支援 6 種聲音及 MP3/WAV/PCM/G.711 格式）及語音辨識（STT）全面支援。
2. **Codex OAuth 圖像整合**：openai/gpt-image-2 現可透過 Codex OAuth 使用，無需額外設定 `OPENAI_API_KEY`；OpenRouter 亦同步支援 image_generate 函式。
3. **Memory/Dreaming 解耦**：記憶體夢境功能與 heartbeat 分離，改為獨立輕量 agent 執行，即使 heartbeat 停用也能正常觸發夢境。
4. **安全性三項強化**：Teams 跨 bot token replay 防護、Android cleartext gateway 限制為 loopback-only、行動裝置配對要求私有 IP 或 loopback host。
5. **Discord Thread Subagent（Beta）**：子 agent 可綁定至 Discord thread，透過 `session.threadBindings.enabled=true` 啟用；下一步計畫支援 Telegram / Slack / iMessage。
6. **GPT-5.5 支援缺口（Issue #70834）**：OpenAI 於 4/23 發布 GPT-5.5，但 OpenClaw 的 openai-codex provider 尚未識別該模型，社群正等待修補。
7. **Active Memory Plugin 強化**：新增專屬記憶子 agent，配置 message/recent/full context 三種模式，並支援 `/verbose` 即時檢查。
8. **倉庫突破 74,500 stars**：主倉庫達 74,507 顆星，TypeScript 程式碼量 363,900 行，成長勢頭持續。
9. **ClawHub Auth Bug 持續存在（Issue #52949）**：gateway 的 skills-clawhub 模組發出未攜帶認證 token 的 HTTP 請求，導致 429 持續錯誤及「Available Skills」清單為空。

---

## 2. LLM 串接實戰回報

### Anthropic Claude
- **Auth 設定**：支援 `ANTHROPIC_API_KEYS`、`ANTHROPIC_API_KEY_1`/`_2` key rotation，以及 `OPENCLAW_LIVE_ANTHROPIC_KEY`。
- **踩坑：False Rate Limit**（[Issue #32828](https://github.com/openclaw/openclaw/issues/32828)）：使用 claude OAuth 模式時，所有模型顯示「API rate limit reached」，但實際 API 完全正常。根本原因是 OpenClaw hardcode 以 `rate_limit` 觸發冷卻，不論實際失敗原因。
- **社群回報**（[answeroverflow](https://www.answeroverflow.com/m/1478788645472436264)）：多位用戶反映 OAuth token 環境下特別容易觸發此誤判，重啟 gateway 後可暫時恢復。
- **推薦模型**：claude-opus-4-6（旗艦）、claude-sonnet-4-6（最佳平衡）、claude-haiku-3-5（輕量/低成本）。

### OpenAI GPT
- **新功能**：gpt-image-2 圖像生成與參考圖編輯現透過 Codex OAuth 支援，免除 `OPENAI_API_KEY` 依賴。
- **未解問題**（[Issue #70834](https://github.com/openclaw/openclaw/issues/70834)）：GPT-5.5 於 2026-04-23 發布，但 openai-codex provider 目前不認識 `gpt-5.5` 為有效模型 ID，需等待 OpenClaw 側更新。

### Google Gemini
- **Auth 設定**：`GEMINI_API_KEYS`、`GEMINI_API_KEY_1`/`_2`、`GOOGLE_API_KEY`（fallback）、`OPENCLAW_LIVE_GEMINI_KEY`。
- **使用建議**：gemini-2.5-pro 適合長文本任務（超長 context）；gemini-2.5-flash 速度最快、適合即時對話場景。
- **文件**：[Model providers - OpenClaw](https://docs.openclaw.ai/concepts/model-providers)

### xAI Grok（v2026.4.23 新增）
- 新增 grok-imagine-image / grok-imagine-image-pro 圖像生成及參考圖編輯。
- 語音支援：6 種內建聲音、MP3/WAV/PCM/G.711 TTS 輸出格式。
- 新增 grok-stt 批次音訊轉錄及 Voice Call 即時 STT 串流。
- 設定方式：`providers.xai.apiKey` 或環境變數 `XAI_API_KEY`。

### DeepSeek
- **V4 成本效益**：$0.30/M input tokens，SWE-bench Verified 評分 81%，1M token context window，快取命中折扣 90%。
- **社群評價**：適合預算有限的大規模 agent 部署，cost/quality 比最佳。
- **資料來源**：[Best AI Models for OpenClaw in 2026](https://flypix.ai/best-ai-model-openclaw/)

### Mistral
- **Devstral**：工具呼叫可靠性為 Mistral 系列最佳，多步 agent 任務中自主工具鏈表現穩定。
- **Mistral Large 2512**：262K context，適合需要長文本理解的任務。
- **v2026.4.23 新增**：Voice Call STT 串流支援（[Release Notes](https://github.com/openclaw/openclaw/releases/tag/v2026.4.23)）。
- **評測**：[Best Mistral Models for OpenClaw (2026)](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

### OpenRouter
- 單一 `OPENROUTER_API_KEY` 可存取 200+ 模型，含 DeepSeek、Llama、Mistral 等。
- v2026.4.23 起支援 OpenRouter image_generate 函式，圖像模型整合更簡便。
- [Top AI Models Used by OpenClaw on OpenRouter](https://openrouter.ai/collections/openclaw)

---

## 3. 社群反應與討論亮點

- **Discord vs Telegram 體驗對比**：用戶 Ayush（[@ayushtweetshere](https://x.com/ayushtweetshere/status/2025452054987309244)）在 X.com 表示 Discord 多 agent 任務並行管理遠優於 Telegram，可針對不同專案/目標建立獨立頻道。
- **Thread Subagent 公告**（[@onusoz on X](https://x.com/onusoz/status/2025280441888960620)）：開發者 Onur Solmaz 介紹新 beta 功能，子 agent 可在 Discord thread 中對話，下一步整合 Telegram / Slack / iMessage 及 ACP 協定。
- **r/openclaw 熱議 OpenClaw vs Hermes Agent**：社群已達 10.3 萬成員，近期最熱討論為兩大 AI agent 框架的架構與使用體驗比較。
- **Discord 加密貨幣禁令**：OpenClaw 官方 Discord 宣布禁止所有加密貨幣話題（[CoinMarketCap 報導](https://coinmarketcap.com/academy/article/openclaw-bans-all-crypto-talk-on-discord-after-clawd-token-chaos)），起因於 CLAWD token 相關混亂事件，社群反應兩極。
- **update 破壞 Discord 整合**：有用戶回報最新更新導致 Discord 功能異常（[answeroverflow](https://www.answeroverflow.com/m/1475576545958432778)），社群正在追蹤修復進度。
- **Plugin peer dependency 修復**：外部 plugin 開發者關注的 peer dependency 解析問題已在 v2026.4.23 修復，可避免安裝時產生重複 host 套件。

---

## 4. 值得追蹤的後續議題

| 議題 | 說明 | 追蹤連結 |
|------|------|---------|
| GPT-5.5 provider 支援 | openai-codex provider 需更新識別 `gpt-5.5` | [Issue #70834](https://github.com/openclaw/openclaw/issues/70834) |
| False rate limit bug | OAuth 環境下誤觸 hardcode 冷卻機制 | [Issue #32828](https://github.com/openclaw/openclaw/issues/32828) |
| ClawHub Auth Bug | gateway 不傳遞認證 token 導致 429 / Skills 空白 | [Issue #52949](https://github.com/openclaw/openclaw/issues/52949) |
| Thread Subagent 正式版 | Discord thread 綁定仍為 beta，多平台支援待完成 | [X.com 公告](https://x.com/onusoz/status/2025280441888960620) |
| Task Brain 穩定性 | v2026.3.31 引入的新架構 bug 回報持續收集 | [releasebot.io](https://releasebot.io/updates/openclaw) |
| Agent-to-Agent 通訊協定 | 社群 roadmap 中標準化 agent 間通訊協定開發進度 | [AGENTS.md](https://github.com/openclaw/openclaw/blob/main/AGENTS.md) |
| Workflow Marketplace | Lobster workflow 模板分享／交易市集計畫 | [bswen.com 指南](https://docs.bswen.com/blog/2026-03-21-openclaw-multi-agent-workflows-guide/) |

---

*報告生成時間：2026-04-26*
