# OpenClaw 每日情報 2026-05-15

> 資料蒐集時間：2026-05-15 | 涵蓋過去 24 小時動態

---

## 1. 今日重點摘要

1. **新加坡 IMDA 發布 OpenClaw 安全警示（2026-05-14）**：新加坡資訊通信媒體發展局（IMDA）正式發布公告，警告企業不得將 OpenClaw 部署於關鍵任務系統，強調「全能型單一 Agent」設計模式帶來的高風險，並建議改採多 Agent 窄職責架構搭配最小權限原則。

2. **Anthropic 恢復第三方 Agent 使用 Claude 訂閱（2026-05-13）**：繼 4 月 4 日切斷 Claude Pro/Max 訂閱額度給第三方工具後，Anthropic 於 5 月 13 日宣布重新允許 OpenClaw 等工具使用 Claude 訂閱，並預告 6 月 15 日起推出 Agent SDK 積分制度（Pro \$20、Max 5x \$100、Max 20x \$200）。

3. **v2026.5.7 關鍵版本細節確認（2026-05-07 發布）**：本版本確認新增 `openai/chat-latest` 直接 API-key 模型覆蓋選項、Plugin 發布可靠性強化（失敗重試與版本驗證）、Security 授權鉤子改進、Cron 指令 JSON 輸出含計算狀態，以及 Discord/Telegram/WhatsApp/Codex 各平台修復。

4. **假速率限制 Bug（Issue #32828）持續困擾用戶**：OpenClaw 在 API 完全正常運作的情況下，仍對所有已設定的 LLM Provider 回報「API rate limit reached」，導致 Agent 工作流程中斷。OpenClaw 官方確認此為已知 Bug，暫時解法是執行 `openclaw doctor --fix`。

5. **Claude OAuth Token 每 8 小時失效問題受關注**：使用 Claude OAuth 訂閱（而非 API Key）的用戶面臨 Token 每 8 小時過期、Refresh Endpoint 本身有嚴格速率限制、且一旦觸發就鎖定 6 小時的三重困境，社群建議改用 API Key 方式認證。

6. **Gemini 3.1 Pro 取代 Claude Opus 成社群推薦首選**：多個社群評測（2026 年 5 月）將 Gemini 3.1 Pro 列為 OpenClaw 工作流的最佳預設模型，理由是：原生百萬 token 上下文、最低成本（每 Job）、以及原生多模態支援，尤其在大型程式碼審查任務中表現突出。

7. **ClawRouter 第三方路由器引發關注**：BlockRunAI 推出 [ClawRouter](https://github.com/BlockRunAI/ClawRouter)，支援 41+ 模型、路由延遲 <1ms，並整合 Base 及 Solana 鏈上 USDC 付款（via x402 協議），被視為 OpenClaw 多 Provider 生態的新型基礎設施。

8. **Anthropic 回應第三方工具 Token 消耗問題**：Anthropic 說明部分 Claude 訂閱用戶透過 OpenClaw 等工具消耗了遠超訂閱定價的 Token 量（部分用戶 \$20 訂閱卻消耗數百至數千美元價值的 Token），是 4 月強制切斷的根本原因；新 Agent SDK 積分制為官方解方。

9. **v2026.5.3–v2026.5.5 近期版本回顧（五月上旬）**：v2026.5.3 引入四工具檔案傳輸 Plugin（`file_fetch`/`dir_list`/`dir_fetch`/`file_write`，單次 16MB 上限）、v2026.5.4 加入 Twilio 撥入 Google Meet 搭配 Gemini 實時語音、v2026.5.5 針對 LINE/Telegram/Discord/Slack/Matrix 等通道進行全面強化。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **OAuth Token 陷阱**：使用 Claude OAuth 方式授權的用戶回報 Token 每 8 小時自動過期，而 Refresh Endpoint 本身有嚴格速率限制，一旦超出就鎖定 6 小時無法刷新。長期運行的 OpenClaw Agent 工作流建議改用 Direct API Key，避免被 OAuth 生命週期打斷。
- **假速率限制誤判**：Issue [#32828](https://github.com/openclaw/openclaw/issues/32828) 與 [#48603](https://github.com/openclaw/openclaw/issues/48603) 記錄 OpenClaw 在 API 正常的情況下仍回報速率限制，觸發內建冷卻機制，影響所有設定的模型。暫行修復：`openclaw doctor --fix`。
- **Claude Max 額度耗盡問題**：有用戶反映 Claude Max 訂閱額度在密集的 Agent 工作流下快速耗盡，部分用戶轉為將 GitHub Copilot 訂閱設為 API 端點作為輔助路由。
- **Anthropic 積分制（6/15 上線）**：新制度下 Pro 帳戶獲得 \$20 Agent SDK 積分、Max 5x 獲得 \$100、Max 20x 獲得 \$200，可用於 claude -p CLI 命令及第三方 SDK 工具呼叫。
- **來源**：[AnswerOverflow - Claude OAuth 速率限制](https://www.answeroverflow.com/m/1478788645472436264)、[VentureBeat - Anthropic 恢復 OpenClaw 支援](https://venturebeat.com/technology/anthropic-reinstates-openclaw-and-third-party-agent-usage-on-claude-subscriptions-with-a-catch)、[ShareUHack - Claude 費用分析](https://www.shareuhack.com/en/posts/openclaw-claude-code-oauth-cost)

### GPT（OpenAI）

- **`openai/chat-latest` 新別名支援（v2026.5.7）**：可追蹤最新 ChatGPT Instant API 別名，無需更動穩定的預設模型設定，但社群提醒此別名指向「移動目標」，版本切換時行為可能改變。
- **Codex 審批模式改進**：v2026.5.7 強化 Codex 審批模式的持久決策記憶，減少重複確認提示。
- **GPT-5.5 上下文限制**：GPT-5.5 標準 API 上下文為 128K tokens，與 Gemini 3.1 Pro / Claude Opus 4.7 的 1M token 相比差距明顯，複雜長上下文任務建議選用後兩者。
- **來源**：[LaoZhang AI Blog - OpenClaw LLM Setup 2026](https://blog.laozhang.ai/en/posts/openclaw-llm-setup)、[DEV Community - Best LLM for OpenClaw 2026](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### Gemini（Google）

- **社群評測推薦首選**：多份 2026 年 5 月測評將 Gemini 3.1 Pro 列為 OpenClaw 最佳預設，原因：1M token 上下文、最低單次 Job 費用、原生多模態、Google Meet 實時語音整合原生支援。
- **Google Meet 語音整合（v2026.5.4）**：Twilio 撥入 Google Meet 搭配 Gemini 實時串流已上線，包含背壓感知緩衝（backpressure-aware buffering）與打斷佇列清除（barge-in queue clearing），適合語音型 Agent 場景。
- **Gemini MCP 整合**：[Composio 教學](https://composio.dev/toolkits/gemini/framework/openclaw) 提供完整 Gemini MCP 與 OpenClaw 整合步驟。
- **來源**：[DEV Community - Best LLM for OpenClaw 2026](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)、[haimaker.ai Blog](https://haimaker.ai/blog/best-models-for-clawdbot/)

### DeepSeek

- **OpenRouter 路由整合**：透過 OpenRouter 使用 DeepSeek 是降低多模型工作流成本的主流方案，OpenRouter 作為聚合層可對 DeepSeek、Mistral 等多個 Provider 統一使用單一 API 端點與憑證。
- **Cache TTL 支援**：DeepSeek 在 OpenRouter 管理的快取中具備 cache-TTL 資格，但不支援 Anthropic cache marker，混用時需注意。
- **`reasoning_content` 工具呼叫 Bug**（已修復於 v2026.5.2）：多輪工具呼叫中 `reasoning_content` 未被回傳導致後續呼叫失敗，影響複雜 Agent 工作鏈，目前已在 v2026.5.2 修復。
- **來源**：[OpenClaw DeepSeek Docs](https://docs.openclaw.ai/providers/deepseek)、[GitHub Issue #71160](https://github.com/openclaw/openclaw/issues/71160)

### OpenRouter

- **統一多 Provider 憑證**：使用 OpenRouter 作為 LLM 聚合層，可對 DeepSeek、Mistral、Gemma 等 12+ Provider 使用單一 API Key，以 `openrouter/<author>/<slug>` 格式引用模型。
- **ClawRouter 新路由工具**：BlockRunAI 的 [ClawRouter](https://github.com/BlockRunAI/ClawRouter) 支援 41+ 模型，路由延遲低於 1ms，並整合 Base / Solana 鏈上 USDC 付款（x402 協議），為進階用戶提供更靈活的路由策略。
- **免費模型輪替策略**：社群有免費 OpenRouter 模型輪替指南，可在預算有限的情況下維持 Agent 運作。
- **來源**：[Remote OpenClaw - OpenRouter Guide](https://www.remoteopenclaw.com/blog/openrouter-free-models-openclaw-guide)、[BlockRunAI/ClawRouter](https://github.com/BlockRunAI/ClawRouter)

### Mistral

- **Devstral 2512 為 Agent 工作流推薦**：若 OpenClaw 工作流以 Agent Loop 和程式碼生成為主，社群建議換用 Devstral 2512，在長工具呼叫鏈、寫-測-迭代循環中表現優於通用模型。
- **自架成本優勢**：Mistral 在本地或私有雲自架情境下因授權彈性受歡迎，搭配 Ollama 後端可在弱硬體上運行輕量版本。
- **來源**：[haimaker.ai - Best Mistral Models for OpenClaw 2026](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

---

## 3. 社群反應與討論亮點

- **IMDA 警告引發企業用戶關注**：新加坡 IMDA 的 2026-05-14 公告指出，OpenClaw 連結 Slack 頻道時可能接受頻道內任意參與者的指令（缺乏額外身份驗證），並警告用戶訊息、檔案、電子郵件可能被傳送至第三方 AI Provider。企業用戶開始討論如何實施最小權限設定與多 Agent 窄職責架構。（[The Online Citizen](https://theonlinecitizen.com/2026/05/14/singapore-warns-against-unrestricted-use-of-open-claw-ai-agents-on-sensitive-systems)）

- **Anthropic 政策反覆引發不滿**：Hacker News 上針對 Anthropic「4 月禁止 → 5 月恢復」策略的討論激烈（[HN #47844269](https://news.ycombinator.com/item?id=47844269)），部分用戶批評政策缺乏一致性影響企業採用信心，也有人認為 Agent SDK 積分制是合理商業化方向。

- **Gemini 3.1 Pro 崛起：「最佳 OpenClaw 搭配模型」**：[DEV Community 評測文章](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4) 引發熱議，討論焦點集中在大上下文任務（>200K token）時 Gemini vs Claude 的效能與費用取捨，社群普遍認為 Claude 在推理品質與指令遵循上仍略勝，但 Gemini 在成本控制上更具優勢。

- **4 月 Gateway 崩潰後信任問題仍在**：2026.4.26 更新觸發 Gateway 崩潰與 Discord 斷線問題後，PiunikaWeb 報導顯示部分長期用戶宣稱轉投 Hermes Agent，社群討論 OpenClaw 的更新穩定性問題是否得到根本改善。

- **官方 Discord「Friends of the Crustacean 🦞」活躍**：Discord 伺服器目前成員逾 176,000 人，是最主要的即時技術支援與 Bug 回報場所，多數 Claude OAuth 問題、速率限制疑難排解第一手資訊都出現在此處。（[邀請連結](https://discord.com/invite/clawd)）

- **多 Agent 架構成主流討論方向**：結合 IMDA 警告與社群案例分享，「22 Agent 協同公司」等多 Agent 架構討論持續發酵，開發者開始關注 OpenClaw 的 Agent 委派（delegation）與授權邊界設定最佳實踐。

---

## 4. 值得追蹤的後續議題

1. **Anthropic Agent SDK 積分制（6/15 上線）**：新積分制度能否真正解決 Claude 在 OpenClaw 工作流中的費用問題，以及積分耗盡後的降級行為，需在 6 月中旬後持續觀察社群回饋。

2. **Issue #32828 假速率限制 Bug 修復進度**：此 Bug 影響廣泛（所有 Provider 皆受影響），目前仍為 open 狀態，需追蹤官方是否在近期版本納入正式修復。

3. **IMDA 安全建議的落地實施**：企業用戶如何回應新加坡 IMDA 的安全指引（最小權限、多 Agent 窄職責、持續監控），相關最佳實踐文件與 OpenClaw 官方安全配置指引值得關注。

4. **Claude OAuth 8 小時失效問題的官方解法**：目前只有「改用 API Key」作為替代方案，Anthropic 與 OpenClaw 是否計畫改善 OAuth Token 刷新機制尚不明確。

5. **`openai/chat-latest` 別名版本追蹤**：此別名指向不斷更新的 ChatGPT Instant API，模型切換時可能影響 Agent 行為的相容性，需觀察社群遭遇的相容性問題。

6. **ClawRouter 生態系發展**：BlockRunAI 的 ClawRouter 帶入鏈上付款（USDC/x402）新模式，若生態成熟可能改變 OpenClaw 用戶的 Provider 選擇方式，值得持續觀察採用率。

7. **Gemini 實時語音整合穩定性（v2026.5.4）**：Twilio + Google Meet + Gemini 語音整合尚屬新功能，社群實際使用回饋與延遲測試資料值得追蹤，特別是背壓緩衝在高負載下的表現。

---

## 來源連結

- [GitHub - openclaw/openclaw Releases](https://github.com/openclaw/openclaw/releases)
- [GitHub Issue #32828 - 假速率限制 Bug](https://github.com/openclaw/openclaw/issues/32828)
- [GitHub Issue #48603 - 速率限制功能請求](https://github.com/openclaw/openclaw/issues/48603)
- [GitHub - BlockRunAI/ClawRouter](https://github.com/BlockRunAI/ClawRouter)
- [The Online Citizen - IMDA OpenClaw 安全警示](https://theonlinecitizen.com/2026/05/14/singapore-warns-against-unrestricted-use-of-open-claw-ai-agents-on-sensitive-systems)
- [Let's Data Science - IMDA warns about OpenClaw](https://letsdatascience.com/news/imda-warns-about-openclaw-agent-risks-01760888)
- [VentureBeat - Anthropic 恢復 OpenClaw 支援](https://venturebeat.com/technology/anthropic-reinstates-openclaw-and-third-party-agent-usage-on-claude-subscriptions-with-a-catch)
- [Hacker News #47844269 - Anthropic 允許 OpenClaw CLI 再次使用](https://news.ycombinator.com/item?id=47844269)
- [TechCrunch - Anthropic 4 月政策變更](https://techcrunch.com/2026/04/04/anthropic-says-claude-code-subscribers-will-need-to-pay-extra-for-openclaw-support/)
- [DEV Community - Best LLM for OpenClaw 2026](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)
- [haimaker.ai - Best Models for OpenClaw](https://haimaker.ai/blog/best-models-for-clawdbot/)
- [haimaker.ai - Best Mistral Models for OpenClaw](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)
- [LaoZhang AI Blog - OpenClaw API Rate Limit 429 Fix](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)
- [LaoZhang AI Blog - OpenClaw LLM Setup 2026](https://blog.laozhang.ai/en/posts/openclaw-llm-setup)
- [AnswerOverflow - Claude OAuth 速率限制討論](https://www.answeroverflow.com/m/1478788645472436264)
- [ShareUHack - OpenClaw Claude 費用比較](https://www.shareuhack.com/en/posts/openclaw-claude-code-oauth-cost)
- [OpenClaw DeepSeek Docs](https://docs.openclaw.ai/providers/deepseek)
- [Remote OpenClaw - OpenRouter Free Models Guide](https://www.remoteopenclaw.com/blog/openrouter-free-models-openclaw-guide)
- [Blink Blog - What's New in OpenClaw May 2026](https://blink.new/blog/openclaw-whats-new-may-2026)
- [MindStudio - OpenClaw April 2026 Model-Agnostic Provider](https://www.mindstudio.ai/blog/openclaw-april-2026-model-agnostic-provider-manifest)
- [PiunikaWeb - OpenClaw 2026.4.26 Gateway Crashes](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)
- [Composio - Gemini MCP with OpenClaw](https://composio.dev/toolkits/gemini/framework/openclaw)
- [Discord - Friends of the Crustacean 🦞](https://discord.com/invite/clawd)
