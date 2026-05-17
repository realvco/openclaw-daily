# OpenClaw 每日情報 2026-05-13

> 資料涵蓋範圍：2026-05-12 至 2026-05-13（過去 24 小時）

---

## 1. 今日重點摘要

1. **v2026.5.7 為目前最新穩定版**（發布於 5 月 7 日）：新增 `openai/chat-latest` 直接 API 金鑰覆蓋支援、Tavily 搜尋/擷取憑證改由 runtime config snapshot 解析、Codex 核准模式下相同 PermissionRequest payload 在同一 session 內可記住「永遠允許」決策。

2. **v2026.5.5 大規模 channel 強化**：涵蓋 Feishu topic hydration、LINE DM 驗證、Telegram/Codex progress-draft 修正、Matrix 核准重試、Discord heartbeat 時序與指令路由修正、Slack 重連診斷、iOS LAN 配對，以及媒體傳送去重複。

3. **v2026.5.6 修正 Codex OAuth 漏洞**：被社群稱為「趕緊升級，否則 OAuth 會咬你」，屬於需儘速更新的安全性修補。

4. **GitHub 專案規模持續成長**：截至 2026-05-13，`openclaw/openclaw` 已達 **76,824 顆 ★**，目前開放中 issues 共 3,714 筆；今日新增 issues 包含 #81207、#81205（標記 bug/regression）、#81201、#81196、#81194。

5. **PR 開放數量上限調整**：OpenClaw 維護團隊將每位作者最多可同時開啟的 PR 數從 10 筆調升至 **20 筆**，目的是鼓勵貢獻者提交更高品質的工作。

6. **社群票選最佳模型為 Kimi K2.5**（截至 5 月 12 日）；但創作者 Peter Steinberger 仍明確推薦 **Claude Opus 4.6**，理由是其長文脈優勢與最佳的提示注入防禦能力。

7. **ClawHub Auth Token Bug 持續影響用戶**（Issue #52949）：Gateway 未正確傳遞 ClawHub auth token，導致 API 呼叫持續收到 429 rate limit 錯誤，且 ClawControl 中「Available Skills」清單顯示空白。

8. **CVE-2026-25253 安全漏洞仍是重大追蹤議題**：此嚴重遠端程式碼執行漏洞 CVSS 評分 8.8，利用 WebSocket origin header bypass，建議所有用戶確認已升級至修補版本。

9. **Gateway 啟動效能顯著改善**（v2026.5.4）：啟動路徑改為 lazy-loading，涵蓋 plugin、runtime discovery、cron、schema、sessions 及 model metadata，大型安裝環境啟動時間明顯縮短。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **OAuth 導致 rate limit 誤判**：多名用戶回報使用 Claude OAuth 授權後，持續收到「API rate limit reached. Please try again later.」錯誤，但 API 實際上完全正常。已知 bug（Issue #32828），可暫時改用 API Key 方式繞過。
  - 來源：[Friends of the Crustacean Discord - Claude OAuth rate limit](https://www.answeroverflow.com/m/1478788645472436264)

- **Cooldown 機制陷阱**：OpenClaw 偵測到 Anthropic 連續 auth 失敗後，會觸發 30–60 分鐘的 cooldown，並顯示「No available auth profile for anthropic (all in cooldown)」，期間完全無法使用 Claude，需手動重設或等待逾時。
  - 來源：[Fix Every OpenClaw Error: API Key, Rate Limits, Gateway Token — LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)

- **Creator 推薦 Claude Opus 4.6**：Peter Steinberger 在訪談中明確表示 Anthropic Pro/Max + Opus 4.6 是他個人日常使用的組合，主因是長上下文處理能力與提示注入防禦性能。
  - 來源：[Best AI Model for OpenClaw 2026: Claude vs GPT vs DeepSeek — Get AI Perks](https://www.getaiperks.com/en/blogs/18-best-ai-model-openclaw)

### GPT / OpenAI

- **openai/chat-latest 支援正式加入**（v2026.5.7）：可透過直接 API 金鑰覆蓋指定 `openai/chat-latest`，方便追蹤 OpenAI 最新模型而不需手動更新模型名稱。
  - 來源：[OpenClaw Version 2026.5.7 Update — FintechExtra](https://www.fintechextra.com/news/openclaw-version-202657-update-key-enhancements-and-fixes-481)

- **實戰串接案例**：有開發者成功將 OpenClaw 接上 Claude + ChatGPT + Telegram，並透過 Telegram 訊息觸發 Claude Code 自動更新並重新部署 Vercel 托管的網站。
  - 來源：[I Wired OpenClaw to Claude, ChatGPT & Telegram — Medium](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)

### DeepSeek

- **API 服務穩定性問題**：DeepSeek V3 API 在尖峰時段頻繁拋出 503 錯誤，社群建議搭配 fallback model 設定或透過 OpenRouter 路由以降低衝擊。定價方面 DeepSeek V3.2 僅需 $0.28/$0.42 per M tokens，是預算有限用戶的首選，但需接受不穩定風險。
  - 來源：[OpenClaw Best Model Selection Guide — LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-best-model-selection-guide)

### Gemini（Google）

- **Gemini 3 tool-call 修復**（v2026.5.7）：修正了 Gemini 3 工具呼叫的 thought-signature replay 問題，並加入 fallback signatures 機制，改善 Gemini 在 agent 工作流中的相容性。
  - 來源：[Release openclaw 2026.5.7 · GitHub](https://github.com/openclaw/openclaw/releases/tag/v2026.5.7)

- **Gemini 2.5 Flash 受到推薦**：社群評測中 Gemini 2.5 Flash 因其業界最慷慨的免費額度與快速回應速度，成為輕量任務與測試場景的熱門選擇。
  - 來源：[Best LLM for OpenClaw: Gemini 3.1 Pro vs GPT-5.5 vs Claude Opus 4.7 — DEV Community](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### Mistral

- **穩定性勝出但性價比居中**：Mistral 在歐洲資料主權、多語言支援方面有優勢，API 穩定性優於 DeepSeek，但定價高於 DeepSeek V3.2，功能上限則不及 Claude 與 GPT-5。
  - 來源：[Best Mistral Models for OpenClaw — haimaker.ai Blog](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

### OpenRouter

- **200+ 模型單一金鑰整合**：OpenRouter 是目前最方便的多模型實驗方案，用一個 API key 即可切換 DeepSeek、Llama、Mistral 等 200+ 模型，社群推薦在確定主力模型前用 OpenRouter 做橫向比較。
  - 來源：[LLM Rankings — OpenRouter](https://openrouter.ai/rankings)

---

## 3. 社群反應與討論亮點

- **Discord 伺服器成員突破 176,202 人**，官方 Discord（Friends of the Crustacean 🦞🤝）是最活躍的即時問答場所，每日有大量 LLM 設定、錯誤排查問題湧入。

- **4 月底大規模 gateway 崩潰事件餘波**：2026.4.26 更新引發大規模 gateway 崩潰與 Discord 整合失效，社群仍在討論「OpenClaw 缺乏可靠穩定版 channel」的根本問題，要求官方建立更嚴謹的 QA 流程。
  - 來源：[OpenClaw 2026.4.26 update triggering gateway crashes — Piunika Web](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)

- **ClawSweeper 自動化 issue 管理引發討論**：官方新工具 ClawSweeper 每週掃描所有 issue 與 PR 並建議可關閉的項目，部分社群成員對自動關閉機制表示疑慮，擔心有效回報被誤判。
  - 來源：[GitHub - openclaw/clawsweeper](https://github.com/openclaw/clawsweeper)

- **X.com 官方帳號 @openclaw 積極更新**：持續發布版本公告、社群徵稿與使用案例分享，每次 release 後的貼文互動量維持高水準。
  - 來源：[OpenClaw (@openclaw) on X](https://x.com/openclaw)

- **社群評測：Kimi K2.5 爆冷拿下票選第一**：在 pricepertoken.com 社群票選中，Kimi K2.5 超越 Claude 與 GPT 系列奪冠，但開發者提醒 Kimi 在複雜 agent 工作流中的穩定性仍有待驗證。
  - 來源：[Best Model for OpenClaw — Price Per Token](https://pricepertoken.com/leaderboards/openclaw)

---

## 4. 值得追蹤的後續議題

1. **CVE-2026-25253 後續修補確認**：CVSS 8.8 的 WebSocket origin header bypass RCE 漏洞，需確認所有部署環境已升級至含修補的版本，官方應發布明確的受影響版本範圍說明。

2. **Issue #52949 Gateway auth token 修復進度**：ClawHub auth token 未正確傳遞導致持續 429 及 Available Skills 空白，此問題影響範圍廣，需追蹤官方修復時程。

3. **OpenClaw 穩定版 channel 需求**：社群持續呼籲建立類似 `stable` / `nightly` 的發布 channel，避免重演 4 月底大規模崩潰事件，值得關注官方是否納入 roadmap。

4. **Kimi K2.5 在複雜 agent 工作流的實測**：社群票選第一但缺乏系統性基準測試，預期近期會有更多開發者發布實測對比報告。

5. **v2026.5.7 後的 Codex 核准模式行為變更**：`allow-always` 決策記憶機制的引入可能影響安全稽核流程，企業用戶需評估是否調整 admin scope 設定。

6. **OpenRouter vs 直接 API 的成本與延遲權衡**：隨著更多用戶採用 OpenRouter 做多模型路由，社群預期將出現更多系統性的延遲與成本對比數據。

---

*報告生成時間：2026-05-13 | 資料來源：GitHub、Medium、DEV Community、Discord、X.com 等公開管道*
