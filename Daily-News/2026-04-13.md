# OpenClaw 每日情報 2026-04-13

> 資料來源期間：2026-04-12 ～ 2026-04-13｜以繁體中文整理

---

## 1. 今日重點摘要

1. **GitHub 今日新增多筆 Issue 與 PR（#65636、#65630、#65626、#65625、#65624、#65617、#65614、#65613）**：其中 #65617 與 #65613 為功能增強需求，#65614 帶有 `regression` 標籤為確認回歸 Bug，顯示 v2026.4.10 發布後社群進入密集回饋期，issue 數量較前一週明顯增加。

2. **v2026.4.10 發布後持續追蹤：`openclaw exec-policy` 命令落地狀況**：新 `show`/`preset`/`set` 子命令上線後，社群開始測試在 CI/CD 環境中整合 exec approval 設定的可行性。部分用戶回報在 macOS Sequoia 搭配 Docker 的環境下，`exec-policy preset minimal` 套用後仍需手動重啟 daemon 才能生效。

3. **Anthropic OAuth 封鎖衝擊持續發酵（4/4 生效，第 9 天）**：每日仍有新用戶回報「LLM request rejected: You're out of extra usage」錯誤，目前確立的三條遷移路徑為：① 購買 Anthropic Extra Usage 點數、② 改用獨立 API 金鑰（`ANTHROPIC_API_KEY`）、③ 安裝 `claude` CLI 並切換 provider 至 `claude-cli`。社群估計受影響用戶已超過萬人。

4. **Issue #32828：「所有模型均顯示 API rate limit reached」假陽性問題持續困擾用戶**：即使底層 API 完全正常，OpenClaw 仍顯示 rate limit 錯誤。根因確認為 Gateway 重啟時 AbortError 觸發錯誤的 cooldown 計數器累加。本問題在 v2026.4.10 中尚無完整修復，被標記為 `high-priority`。

5. **DeepSeek V3.2 成為社群最熱門低成本方案**：$0.28/M input、$0.40/M output，164K context window，支援 function calling 與整合式 reasoning，透過 OpenRouter 串接穩定性優於直連 DeepSeek API（後者在尖峰時段偶發 503/504 errors）。社群建議：簡單任務用 DeepSeek V3.2，複雜推理留給 Claude Opus 4.6。

6. **ClawRouter（BlockRunAI）持續引發關注**：這款 agent-native LLM router 支援 41+ 模型、< 1ms 路由決策，並整合 USDC 支付（Base & Solana via x402）。定位為 OpenClaw 專用智慧路由層，被認為是 OpenRouter 在 agent 場景的有力競爭者，GitHub 星數在過去兩週快速成長。

7. **v2026.4.9 SSRF 安全修補：interaction-driven navigation bypass 正式關閉**：本次修補確認修復了 browser 互動觸發的 main-frame navigation 能繞過 SSRF quarantine 的問題，runtime-control 環境變數與 browser-control overrides 也不再允許從不受信任的 workspace `.env` 檔案注入。企業部署用戶建議升級。

8. **Google Gemini Issue #5744 仍未修復：單一模型 rate limit 觸發全 provider 冷卻**：連續兩個版本（v2026.4.9 與 v2026.4.10）皆未見修復，使用 Gemini 多模型架構的用戶目前的主要 workaround 為透過 OpenRouter 路由以繞過 OpenClaw 的 provider-level cooldown 邏輯。

9. **GPT-5.4 Unknown model 問題（Issue #37623）追蹤中**：`openai-codex/gpt-5.4` 在 runtime 仍顯示 Unknown model，官方建議暫時改用 `openai-codex/gpt-5.4-mini`，預計在下一版本修復，Issue #36817 同步追蹤中。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **遷移路徑確立，但 Extra Usage 費用陷阱持續出現**：自 4/4 OAuth 封鎖後，多位用戶回報啟用 Extra Usage 後費用意外暴增。根因之一為 Bug #17589（Gateway 重啟時 AbortError 觸發重送 100K+ token context），建議在 Anthropic 帳戶後台設定每日消費上限（Daily spending limit）並啟用費用告警。
- **Claude Opus 4.6 仍為 coding agent 首選**：200K context window、tool use 穩定，但成本較高。社群建議以 Claude Sonnet 4.6 處理中度複雜任務，Opus 4.6 僅用於需要最高推理能力的場景，可降低 50–70% API 費用。
- **Extra Usage 計費方式確認**：按 token 計費，無月費上限，超量部分直接計入信用卡。多位用戶建議搭配 `openclaw-budget-guard`（社群插件）設定 per-session token 上限。
- **來源**：[OpenClaw Extra Usage 說明](https://help.apiyi.com/en/openclaw-claude-llm-request-rejected-extra-usage-fix-en.html)、[LaoZhang AI Blog: Rate Limit 修復指南](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)

### OpenAI / Codex（GPT-5 系列）

- **`openai-codex` 與 `openai` 雙路徑並存造成混淆**：v2026.4.10 新增捆綁式 Codex provider 後，兩條路徑的 auth 流程、thread 管理、compaction 行為均有差異。社群建議新用戶直接使用 `openai-codex` 路徑，舊有 workflow YAML 需更新 provider 設定。
- **GPT-5.4 runtime 不支援（Issue #37623）**：`openai-codex/gpt-5.4` 設定後顯示 Unknown model；`openai-codex/gpt-5.4-mini` 可正常使用，token 成本更低，但推理能力有所取捨。
- **GPT-5.4 的 1M context window 為超長上下文場景首選**：針對需要完整 codebase ingestion 的 agent 工作流，GPT-5.4 的 1M token context window 具備明顯優勢，但目前仍有 runtime 識別問題需解決。
- **來源**：[Issue #37623](https://github.com/openclaw/openclaw/issues/37623)、[GPT-5 OpenClaw 設定指南（GlobalGPT）](https://www.glbgpt.com/hub/openclaw-api-complete-guide/)

### Google Gemini

- **Issue #5744 長期未解：per-model rate limit 觸發全 provider 冷卻**：當 `gemini-3.1-pro-preview` 達到 TPM（tokens per minute）上限時，整個 Google provider 進入冷卻，即使 `gemini-3-flash-preview` 有充足 quota 也無法使用。目前 workaround：改用 OpenRouter 路由 Google 模型，可繞過 OpenClaw 的 provider-level cooldown。
- **費用最佳化建議**：簡單任務路由至 `gemini-3-flash-preview`（較 Claude Opus 便宜約 90%），複雜任務留給 Claude 或 GPT-5，整體費用可降 40–60%。
- **Veo 影片生成修復（v2026.4.9）**：停止送出 `numberOfVideos` 不支援欄位，Gemini Developer API Veo 執行不再報錯。
- **來源**：[Issue #5744](https://github.com/openclaw/openclaw/issues/5744)、[OpenClaw Model Providers 文件](https://docs.openclaw.ai/concepts/model-providers)

### DeepSeek

- **DeepSeek V3.2 整合推薦設定**：透過 OpenRouter 路由（`deepseek/deepseek-v3.2`）為目前最穩定的連接方式，直連 DeepSeek 官方 API 在尖峰時段有 503/504 timeout 風險。V3.2 的 thinking-mode tool calling 讓 multi-step agent 工作流更可靠，V3/V3.1 則不支援此功能。
- **成本比較**：DeepSeek V3.2（$0.28/$0.40 per M tokens）vs Claude Opus 4.6（約 $15/$75 per M tokens），複雜 coding 任務成本差距可達 100 倍。
- **OpenRouter 5.5% 費用加成**：透過 OpenRouter 路由 DeepSeek 需支付額外 5.5% 費用，但換取的穩定性與 fallback 能力對生產環境而言通常值得。
- **來源**：[DeepSeek via OpenRouter 教學（Medium）](https://medium.com/@oo.kaymolly/connect-deepseek-to-openclaw-via-openrouter-7eb19ef61a84)、[haimaker.ai DeepSeek V3.2 比較](https://haimaker.ai/blog/best-deepseek-models-for-openclaw/)

### OpenRouter（多模型路由）

- **290+ 模型統一入口，為 OpenClaw 最常用的多模型解決方案**：一個 API key 串接 Claude、GPT、Gemini、DeepSeek、Grok、Mistral 等，並提供自動 fallback 機制，避免單一 provider 故障造成 agent 停擺。
- **ClawRouter 競合格局成形**：BlockRunAI 的 ClawRouter 以 agent-native 路由切入，< 1ms 決策延遲 + Web3 USDC 支付，針對 OpenClaw agent 場景做了深度最佳化，補足 OpenRouter 在 agent orchestration 方面的不足。
- **推薦路由策略**：簡單任務 → `deepseek/deepseek-v3.2` 或 `google/gemini-3-flash-preview`；中度複雜 → `claude-sonnet-4-6`；高複雜推理 → `claude-opus-4-6` 或 `openai/gpt-5.4`。
- **來源**：[6 best LLM routers for OpenClaw（DEV Community）](https://dev.to/sophiaashi/6-best-llm-routers-for-openclaw-in-2026-17oa)、[ClawRouter GitHub](https://github.com/BlockRunAI/ClawRouter)

### Grok（xAI）

- **1.8M context window 為最大賣點**：透過 OpenRouter 串接，適合需要超長上下文的 agent 工作流（如全量 codebase 分析、長期對話記憶）。tool calling 穩定，multimodal 支援。
- **目前無原生 OpenClaw Grok provider**，主流方式為透過 OpenRouter 路由，直接使用 xAI API 需自行封裝 provider 設定。

### Mistral

- **歐洲合規場景首選**：Mistral 的資料處理在歐盟境內，適合對 GDPR 有嚴格要求的企業用戶。透過 OpenRouter 可直接存取 Mistral Large 2 等模型。
- **成本效益佳**：推理能力介於 GPT-4 mini 與 GPT-4 之間，定價具競爭力，適合作為 Claude/GPT 的 fallback 模型。

---

## 3. 社群反應與討論亮點

### X.com

- **各大 AI 帳號持續追蹤 v2026.4.10 後續**：多個 AI Native 相關帳號整理 OpenClaw 生態最新動態，社群對 Active Memory Plugin 的評價分歧——重度用戶普遍正面，但也有人擔憂記憶資料的隱私與安全性。
- **`exec-policy` 實戰分享湧現**：開發者在 X.com 分享使用新 `exec-policy preset restrictive` 設定後，agent 不再意外執行破壞性操作的心得，引發廣泛轉發。反映社群對 agent 安全管控的強烈需求。
- **「OpenClaw 自動化商業應用」討論熱度持續**：以 OpenClaw 打造內容生成流水線（研究→撰寫→排程）的實作分享持續吸引關注，顯示工具逐漸從個人生產力工具向商業自動化場景擴展。

### Reddit（r/openclaw）

- **「v2026.4.10 踩坑彙整」討論串延燒**：延續昨日熱門討論，今日新增多條回覆記錄在 macOS/Windows 環境下的 plugin 相容性問題。社群建議在更新前執行 `openclaw config validate` 並備份 `~/.openclaw/config.yaml`。
- **「GitHub Discussion #188842：更新後 OpenClaw 變得無用」**：用戶回報 v2026.4.10 更新後部分自訂插件失效，原因為 Plugin SDK command-auth 路徑拆分導致舊版插件找不到 export。解決方案：更新插件以改用新的 `openclaw/plugin-sdk/command-status` subpath。
- **Active Memory 隱私討論**：社群關注 Active Memory Plugin 記憶資料儲存位置（`~/.openclaw/memory/`），擔憂是否有加密、是否可完全刪除，以及跨 session 的記憶混入問題。官方文件尚不完整，討論持續中。

### Discord（Friends of the Crustacean 🦞🤝，169,545 成員）

- **#help 頻道今日熱議：Anthropic Extra Usage 費用控管**：多位用戶分享費用暴增的排查過程，確認 Bug #17589（Gateway 重啟觸發重送）為主要元兇之一，建議在 `openclaw.json` 設定 `maxRetries: 0` 作為臨時 workaround。
- **#plugin-dev 頻道：Plugin SDK 遷移教學**：社群成員發布 `command-auth` → `command-status` 遷移指南，協助受影響的插件開發者快速更新。
- **官方確認下一個版本重點**：根據 Discord 公告，下一個版本將聚焦於修復 Issue #37623（GPT-5.4 Unknown model）、Issue #5744（Gemini provider cooldown）及 Bug #17589（AbortError 費用爆炸）。

### 生態規模

- OpenClaw GitHub Stars：**250,829+**
- 官方 Discord 成員：**169,545 人**
- 社群 Skills Marketplace：**5,700+ 技能**
- 支援 messaging/service 整合：**50+ 頻道**

---

## 4. 值得追蹤的後續議題

1. **Issue #32828（假陽性 rate limit）完整修復進度**：被標記為 `high-priority`，Gateway 重啟觸發錯誤 cooldown 計數器的根本問題尚未解決。直接影響所有 provider 的穩定性，預計官方將在下一個版本優先處理。

2. **GPT-5.4 Runtime 支援（Issue #37623 & #36817）**：官方 Discord 確認下一個版本將修復，但無明確時程。目前 workaround 為使用 `openai-codex/gpt-5.4-mini`，但推理能力有所取捨。

3. **Active Memory Plugin 隱私與資料安全文件補完**：`~/.openclaw/memory/` 目錄的加密機制、資料保留政策、跨 session 隔離設計均未有完整說明，預計官方將在近期補充文件。企業用戶部署前應謹慎評估。

4. **Google Gemini per-model cooldown 問題（Issue #5744）修復時程**：連續兩個版本未見修復，官方是否計入下一個版本仍不確定。重度使用 Gemini 多模型架構的用戶應持續追蹤，考慮以 OpenRouter 路由作為長期 workaround。

5. **Plugin SDK `command-auth` → `command-status` 遷移衝擊範圍**：GitHub Discussion #188842 顯示受影響的插件不少，Marketplace 上的第三方插件是否已同步更新尚未確認。建議插件使用者在更新 OpenClaw 本體前，先確認所有依賴插件的相容性。

6. **ClawRouter 實際效能驗證**：BlockRunAI 宣稱 < 1ms routing 決策的實際表現尚待社群大規模測試驗證，其 Web3 支付整合（USDC on Base & Solana）對主流用戶的實用性也有待觀察。

7. **`openclaw exec-policy` 在企業 CI/CD 的最佳實踐**：新命令上線後，如何整合進 GitHub Actions、Jenkins 等 CI/CD 流水線並設定最小權限 exec policy，預計將成為社群下一階段的討論重點。

---

*報告生成時間：2026-04-13 | 資料來源涵蓋 GitHub、X.com、Reddit、Discord、Hacker News、Medium 等平台*

### 主要來源連結

- [OpenClaw GitHub Releases](https://github.com/openclaw/openclaw/releases)
- [OpenClaw GitHub Issues](https://github.com/openclaw/openclaw/issues)
- [Issue #32828：假陽性 rate limit](https://github.com/openclaw/openclaw/issues/32828)
- [Issue #5744：Gemini per-model rate limit 觸發 provider 冷卻](https://github.com/openclaw/openclaw/issues/5744)
- [Issue #37623：GPT-5.4 Unknown model](https://github.com/openclaw/openclaw/issues/37623)
- [GitHub Discussion #188842：更新後插件失效](https://github.com/orgs/community/discussions/188842)
- [OpenClaw Model Providers 文件](https://docs.openclaw.ai/concepts/model-providers)
- [Fix Every OpenClaw Error：完整排錯指南（LaoZhang AI）](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)
- [Fix OpenClaw Rate Limit 429：完整指南（LaoZhang AI）](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)
- [OpenClaw Extra Usage 修復說明（apiyi.com）](https://help.apiyi.com/en/openclaw-claude-llm-request-rejected-extra-usage-fix-en.html)
- [DeepSeek via OpenRouter 教學（Medium）](https://medium.com/@oo.kaymolly/connect-deepseek-to-openclaw-via-openrouter-7eb19ef61a84)
- [haimaker.ai：DeepSeek V3.2 vs V3 for OpenClaw](https://haimaker.ai/blog/best-deepseek-models-for-openclaw/)
- [ClawRouter GitHub（BlockRunAI）](https://github.com/BlockRunAI/ClawRouter)
- [6 best LLM routers for OpenClaw（DEV Community）](https://dev.to/sophiaashi/6-best-llm-routers-for-openclaw-in-2026-17oa)
- [Openclaw Release Notes - April 2026（Releasebot）](https://releasebot.io/updates/openclaw)
- [OpenClaw v2026.4.9 深度解析（NewClawTimes）](https://newclawtimes.com/articles/openclaw-2026-4-9-memory-dreaming-security-hardening-android/)

> **郵件狀態：** Gmail MCP 僅提供草稿建立工具（`gmail_create_draft`），缺少 `gmail_send_draft` 傳送工具，無法直接發送郵件。已嘗試建立草稿至 kim.x@realvco.com，主旨：OpenClaw 每日情報 2026-04-13，請至 Gmail 草稿匣確認並手動傳送。**郵件未直接發送**。
