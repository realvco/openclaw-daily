# OpenClaw 每日情報 2026-04-27

> 資料涵蓋區間：2026-04-26 ～ 2026-04-27
> 資料來源：GitHub openclaw/openclaw、releasebot.io、newreleases.io、OpenRouter Docs、pricepertoken.com、answeroverflow.com、Medium、XDA Developers、Startup Fortune

---

## 1. 今日重點摘要

1. **v2026.4.25 TTS 全面升級（4月26日）**：正式版本帶來完整 TTS 改版，新增 `/tts latest` 指令、聊天範圍自動 TTS 開關、聲音角色（persona）設定，並覆蓋 Azure Speech、小米（Xiaomi）、Local CLI、Inworld、Volcengine 及 ElevenLabs v3 等六個新供應商。同時支援逐 agent／逐帳號的 TTS 覆蓋設定。

2. **v2026.4.25-beta.4 加入 PWA 與 Web Push（4月26日）**：Control UI 新增漸進式網頁應用（PWA）安裝能力與 Web Push 推播通知，OpenTelemetry 覆蓋範圍延伸至模型呼叫與系統程序，跨平台安裝強化（Windows、macOS、Linux）。

3. **今日 GitHub 湧入大量 regression 回報（4月27日）**：升級至 v2026.4.24 後，QQBot 初始化失敗（#72479）、Feishu + QQBot 在 Windows 無法使用（#72463）、插件服務崩潰（#72488）、`openclaw status` 失效並拋出 `PlatformAdapter not registered`（#72465）、Web UI 日誌介面空白（#72450）等 regression 問題集中爆發。

4. **DeepSeek V4 Flash 成為新安裝預設模型**：OpenClaw 將 DeepSeek V4 Flash 設為全新安裝的預設 LLM，V4 Pro 同步加入目錄。此舉引發社群熱議，尤其在華為晶片地緣政治風險尚未明朗的背景下，企業用戶對依賴中國 AI 基礎設施的態度分歧明顯。

5. **Google Meet 整合（v2026.4.24）穩定性問題浮現**：雖然 Google Meet 插件於 4月25日隨 v2026.4.24 正式上線，但今日已有用戶提交 Issue #72478，要求支援更可靠的 headless agent 加入機制，含音訊／轉錄健康狀態檢查。

6. **Claude 訂閱者使用權限縮緊（自4月4日起）**：Claude 訂閱用戶的 OpenClaw 使用不再包含在原方案中，需額外付費；但 Anthropic 已確認透過 Claude CLI 方式使用 OpenClaw 仍屬合規。

7. **Kimi K2.5 登頂社群 LLM 排行榜**：根據 [pricepertoken.com](https://pricepertoken.com/leaderboards/openclaw) 截至 4月25日的開發者票選，Kimi K2.5 成為 OpenClaw 最佳模型，其次為 GLM 4.7，Claude Opus 4.6 排名第三。

8. **插件登錄機制重構（plugin registry migration）**：v2026.4.25-beta.4 對插件登錄進行遷移，改善效能與確定性；但同日回報的 Issue #72459「Plugin loader silently drops register* on missing fields」顯示遷移尚有邊緣案例待處理。

9. **新增功能需求：越南語支援與打字指示器**：社群今日提交 Feature Request #72486（Control UI 加入越南語 vi 語系）及 #72447（agent 處理中顯示打字指示器），反映國際化與 UX 改善需求。

---

## 2. LLM 串接實戰回報

### Anthropic Claude

- **訂閱政策踩坑**：2026年4月4日後，Claude 訂閱（非 API key）用戶的 OpenClaw 使用需升級方案（[XDA Developers 報導](https://www.xda-developers.com/claude-subscribers-just-lost-access-to-openclaw-and-other-third-party-toolsunless-they-pay-more/)）。
- **OAuth Token 格式陷阱**：`sk-ant-oat01-` 前綴的 OAuth token 必須使用 `{"type": "token", "provider": "anthropic", "token": "..."}` 格式——若誤用 `"type": "api_key"` 搭配 `"key"` 欄位，auth 將靜默失敗，且可能被 OpenClaw 錯誤歸類為 rate limit 觸發無限冷卻。
  - 來源：[AnswerOverflow](https://www.answeroverflow.com/m/1478788645472436264)
- **虛假 Rate Limit（Issue #32828）**：使用 Claude OAuth 模式時，所有模型均顯示「API rate limit reached」，但 API 本身完全正常。根本原因為 OpenClaw provider cooldown 邏輯硬編碼 `"rate_limit"` 觸發原因，不論實際失敗為 timeout、billing 還是 auth error，一律觸發無限冷卻且無自動恢復。
  - 來源：[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828) / [Issue #23996](https://github.com/openclaw/openclaw/issues/23996)

### DeepSeek（V4 Flash / V4 Pro）

- **V4 Pro tool-call 中斷修復（Issue #71160）**：透過 OpenRouter 使用 DeepSeek V4 Pro 時，工具呼叫後的跟進請求因缺少 `reasoning_content` 而被 API 拒絕（400 錯誤）。v2026.4.24 已補齊 assistant tool-call 重播時的 reasoning_content 佔位符，修復此問題。
  - 來源：[GitHub Issue #71160](https://github.com/openclaw/openclaw/issues/71160)
- **V4 Flash 設為預設模型**：速度快、成本低，適合大多數日常 agent 工作流。V4 Pro 適合需要深度推理的複雜任務，但 tool-call 場景需確認版本 ≥ 2026.4.24。
  - 來源：[Startup Fortune](https://startupfortune.com/openclaw-makes-deepseek-v4-flash-its-default-model-as-the-huawei-chip-question-hangs-over-the-industry/)

### OpenRouter

- **整合方式更新**：無需配置 `models.providers`，僅設定 API key，模型名稱格式為 `openrouter/<author>/<slug>` 即可直接使用。DeepSeek、Kimi、GLM 等模型均可透過此途徑接入。
  - 來源：[OpenRouter 官方文件](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)

### Gemini

- **空白回覆導致 session 停滯已修復**：Gemini 模型在輸出純 reasoning 或 planning 內容時，OpenClaw 現改為自動重試，避免 session 無聲停滯的問題。
- **Gemini Live 語音橋接**：v2026.4.24 同步帶來 Gemini Live 語音橋接支援，可整合至語音工作流。

### 跨 Provider 共通問題

- **虛假 Provider Cooldown（Issue #23996，Critical）**：無論實際錯誤類型（timeout、billing、auth），OpenClaw 統一以 `rate_limit` 觸發冷卻，導致即使 API 正常也無法自動恢復，被標記為 critical 嚴重等級。建議暫時以重啟 gateway 手動恢復，或以 `openclaw doctor --fix` 嘗試自動診斷。
  - 來源：[GitHub Issue #23996](https://github.com/openclaw/openclaw/issues/23996)

---

## 3. 社群反應與討論亮點

- **DeepSeek 預設引發路線辯論**：OpenClaw 以中國 AI 模型作為預設，社群在 Discord 與 GitHub Discussion 上熱烈討論。支持者強調 V4 Flash 的速度與成本效益；反對者擔憂地緣政治風險及資料主權，尤其是企業級部署場景。（[Startup Fortune](https://startupfortune.com/openclaw-makes-deepseek-v4-flash-its-default-model-as-the-huawei-chip-question-hangs-over-the-industry/)）

- **Medium 實戰串接文章廣傳**：Mohit Aggarwal 的文章「[I Wired OpenClaw to Claude, ChatGPT & Telegram — Here's What Actually Works](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)」在社群廣泛流傳，詳述三平台實際可行的串接方案，被認為是目前最實用的上手指南之一。

- **v2026.4.24 升級踩雷潮**：GitHub 今日（4月27日）大量 regression 回報集中在 QQBot、Feishu、Windows 平台、插件服務等方向，部分中文社群用戶反映「升級後完全無法使用」，要求官方加快 hotfix 發布。

- **MyClaw AI 降低使用門檻受關注**：[North Penn Now 報導](https://northpennnow.com/news/2026/apr/23/openclaw-without-the-headache-a-hands-on-look-at-myclaw-ai/) 稱 MyClaw AI 讓 OpenClaw 的配置門檻大幅降低，吸引非技術背景用戶投入討論，OpenClaw 生態系工具層持續豐富。

- **Discord 社群持續擴張**：「Friends of the Crustacean 🦞🤝」Discord 伺服器成員已達 174,033 人，GitHub 主倉庫 open issues 達 3,703 個、open PR 達 3,540 個，顯示社群活躍度持續提升。

- **LLM 社群排行榜成參考依據**：[pricepertoken.com OpenClaw 排行](https://pricepertoken.com/leaderboards/openclaw) 的開發者實際使用投票結果（Kimi K2.5 第一）與官方建議出現落差，社群開始討論「官方建議 vs. 實際使用體驗」的分歧。

---

## 4. 值得追蹤的後續議題

1. **v2026.4.24 Regression Hotfix 進度**：QQBot、Feishu、插件服務、Windows 平台的多個 regression 問題影響大量中文及企業用戶，需密切關注官方 hotfix 或 v2026.4.26 的發布時程。

2. **DeepSeek 預設模型的長期影響**：隨著 OpenClaw 以中國 AI 模型為預設，需持續觀察是否影響企業採用決策，以及華為晶片相關地緣政治風險的後續演變。

3. **Provider Cooldown Bug 根本修復（Issue #23996）**：此為 critical 嚴重性問題，硬編碼 `rate_limit` 觸發無限冷卻的設計缺陷若不根本解決，將持續影響所有多 provider 配置用戶的穩定性。

4. **Claude 訂閱收費政策後的用戶流向**：Claude 訂閱用戶的 OpenClaw 使用權縮緊後，是否有大量用戶轉向 DeepSeek、Kimi 或 OpenRouter，值得持續追蹤社群反應與模型使用份額變化。

5. **TTS 生態系擴張與語音工作流完整性**：v2026.4.25 大幅擴展 TTS 供應商，Talk → Voice Call → Google Meet → TTS 的完整語音工作流鏈路是否穩定可用，需實際測試與社群回報。

6. **PWA 部署模式的企業應用潛力**：Control UI 支援 PWA 安裝後，OpenClaw 可作為獨立網頁應用在企業內部部署，是否影響未來的產品定位與使用場景值得觀察。

7. **`models.providers.<id>.api` enum 問題（Issue #72477）**：gateway 啟動時的 enum mismatch 崩潰問題，若影響廣泛可能成為今日最緊急的修復項目。

---

*報告生成時間：2026-04-27*
*資料來源：GitHub、OpenRouter、Medium、XDA Developers、Startup Fortune、pricepertoken.com、answeroverflow.com、releasebot.io*
