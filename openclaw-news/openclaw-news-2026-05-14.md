# OpenClaw 每日情報 2026-05-14

> 資料蒐集時間：2026-05-14 | 涵蓋過去 24 小時動態

---

## 1. 今日重點摘要

1. **v2026.5.7 正式發布（2026-05-07）**：此版本新增 Plugin 發布可靠性提升機制，包含重試暫時性 ClawHub CLI 依賴安裝失敗、確保所有預期 ClawHub 套件版本在發布後得到驗證，以及支援 `openai/chat-latest` 作為直接 API key 模型覆蓋選項。

2. **v2026.5.5 通道強化上線**：本版本針對 Feishu 主題水合、LINE DM 驗證、Telegram/Codex 進度草稿修復、Matrix 審批重試、Discord 心跳時序、Slack 重連診斷、iOS LAN 配對、Gateway 關機清理、媒體發送去重等進行全面強化。

3. **v2026.5.4 啟動效能大幅提升**：Gateway 啟動路徑改為延遲載入外掛程式、排程任務、Schema、Session 及模型元數據，只載入當前需要的部分，對大型 OpenClaw 安裝（含多外掛）可顯著減少啟動時間與記憶體峰值。

4. **v2026.5.3 新增檔案傳輸外掛**：引入四個新 agent 工具：`file_fetch`、`dir_list`、`dir_fetch`、`file_write`，支援配對節點間的二進位檔案操作，單次傳輸上限 16 MB，並由操作員控制路徑政策。

5. **假速率限制觸發冷卻 Bug 仍存在**：Issue [#32828](https://github.com/openclaw/openclaw/issues/32828) 與 [#23996](https://github.com/openclaw/openclaw/issues/23996) 回報 OpenClaw 在未真正觸及 API 限制的情況下仍將多個 Provider 置入冷卻，導致使用者無法正常使用所有模型。

6. **OpenRouter + DeepSeek V4 Pro 工具呼叫 Bug 修復**：Issue [#71160](https://github.com/openclaw/openclaw/issues/71160) 確認 DeepSeek V4 Pro 在多輪工具呼叫時因 `reasoning_content` 未被傳回 API 而失敗，已於 v2026.5.2 修復。

7. **專案星數突破 369,000**：截至 2026 年 5 月，OpenClaw 已成為 GitHub 上星數最高的非聚合類軟體專案之一（250,829+ 星於 2026 年 3 月突破，最新約 369,000 星），社群活躍度持續攀升。

8. **安全警示 CVE-2026-25253**：研究人員發現 OpenClaw 存在重大遠端代碼執行漏洞，網路上已有超過 135,000 個暴露的 OpenClaw 實例，其中逾 50,000 個直接易受 RCE 攻擊，強烈建議立即更新並限制外部存取。

9. **Beta 版新功能預告**：目前 Beta 流包含 Model Auth 狀態卡（顯示 token 健康狀態與速率限制壓力）、LanceDB 雲端記憶體索引支援，以及 `localModelLean` 模式（為弱硬體自架模型精簡預設工具集）。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **整體評分最高**：在多個社群測評中，Claude Opus 4.7 在長上下文任務、提示詞注入抵抗力及多步驟工具使用上均優於競品。SWE-bench Pro 成績達 **64.3%**（Anthropic 官方評估），且實際使用中常見「自我核查」行為。
- **OAuth 速率限制誤判**：使用 Claude OAuth 訂閱的用戶回報在 API 完全正常的情況下仍收到「API rate limit reached」錯誤，社群伺服器有大量相關討論 ([AnswerOverflow](https://www.answeroverflow.com/m/1478788645472436264))。
- **Claude Max 訂閱上限問題**：有用戶表示 Claude Max 訂閱很快達到使用上限，因此設定代理路由，將 CoPilot 訂閱作為 API 端點使用 ([Medium](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc))。
- **來源**：[Best LLM for OpenClaw: Gemini 3.1 Pro vs GPT-5.5 vs Claude Opus 4.7 (2026)](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### GPT（OpenAI）

- **v2026.5.7 新增 `openai/chat-latest` 支援**：可使用此別名追蹤最新 ChatGPT Instant API，無需修改穩定的預設模型設定。
- **Doctor 修復注意事項**：`doctor --fix` 在 2026.5.5 後版本中可能不當覆寫 Codex OAuth 路由，更新 v2026.5.7 後才解決此問題，建議使用者更新前先備份設定。
- **來源**：[OpenClaw Version 2026.5.7 Update](https://www.fintechextra.com/news/openclaw-version-202657-update-key-enhancements-and-fixes-481)

### DeepSeek

- **OpenRouter + DeepSeek V4 Pro 工具呼叫失敗**：在多輪工具呼叫中，後續請求被拒絕，錯誤訊息為「The reasoning_content in the thinking mode must be passed back to the API」，導致對話必須切換模型或重置才能繼續。已於 **v2026.5.2** 修復。
- **快取支援**：DeepSeek 在 OpenRouter 管理的快取中具備 cache-TTL 資格，但不支援 Anthropic cache marker。
- **來源**：[GitHub Issue #71160](https://github.com/openclaw/openclaw/issues/71160)

### Grok（xAI）

- **v2026.5.2 新增 Grok 4.3 支援**：與 OpenAI Codex 一起在 5 月初版本中獲得整合支援。
- **來源**：[OpenClaw 2026.5.2: Codex, Grok 4.3 & What's New](https://www.buildfastwithai.com/blogs/openclaw-2026-5-2-release)

### OpenRouter

- **ClawRouter 第三方路由器**：BlockRunAI 推出 [ClawRouter](https://github.com/BlockRunAI/ClawRouter)，專為 OpenClaw 設計的 LLM 路由器，支援 41+ 模型、<1ms 路由延遲，並支援 Base 及 Solana 上的 USDC 付款（via x402 協議）。
- **原生整合**：OpenClaw 對 OpenRouter 有原生支援，只需設定 API key 並以 `openrouter/<author>/<slug>` 格式引用模型即可。
- **來源**：[OpenRouter Integration Docs](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)

### Mistral / Gemini

- **Gemini 3.1 Pro vs GPT-5.5 比較評測**：在 dev.to 上有詳細測評文章，Gemini 在特定多媒體任務（尤其是 Google Meet / Voice Call 整合）表現突出；Mistral 在自架場景下因成本優勢受到歡迎。
- **Realtime Gemini 語音支援**：v2026.5.4 包含 Twilio 撥入的 Gemini 實時語音、節奏音訊串流、背壓感知緩衝與打斷清隊功能。
- **來源**：[Best LLM for OpenClaw (2026) - DEV Community](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

---

## 3. 社群反應與討論亮點

- **「2026 年是個人 Agent 元年」**：社群中出現大量以此為主題的討論，OpenClaw 被視為個人自動化與 AI 整合的核心工具，星數持續暴增。

- **TweenerClaw Discord 新社群成立**：定期舉辦 OpenClaw Talk 系列分享會，第三場主題為「22 個 Agent 組成的公司」，展示以 OpenClaw 建立完整公司工作流的實戰案例。官方 Discord 伺服器邀請連結：[Friends of the Crustacean](https://discord.com/invite/clawd)

- **連續版本問題引發抱怨**：2026.4.26 更新後的 Gateway 崩潰及 Discord 問題引發社群不滿，PiunikaWeb 有專文報導。部分用戶表示「連續幾個有問題的版本」正在消耗信任感。

- **安全漏洞社群反應激烈**：CVE-2026-25253 的公開後，社群對預設暴露設定的批評聲浪高漲，有討論希望官方在安裝流程加入強制的網路隔離提醒。

- **Robbie Allen「22 Agent 公司」案例**：社群熱議如何以 OpenClaw 建立多 Agent 協同工作的完整公司架構，此案例在 TweenerTimes 電子報 [Round 3](https://www.tweenertimes.com/p/openclaw-talk-round-3-the-22-agent) 中有詳細介紹。

- **GitHub 自動化整合受關注**：LumaDock 發布教學文章，介紹如何使用 OpenClaw 自動化 PR 審查與 CI 監控，對開發者工作流整合呼聲漸高。

---

## 4. 值得追蹤的後續議題

1. **CVE-2026-25253 修補進度**：目前已有 50,000+ 個易受攻擊的暴露實例，需持續追蹤官方是否發布強制安全更新或網路隔離指引。

2. **Beta 功能正式發布時程**：Model Auth 狀態卡、LanceDB 雲端記憶體及 `localModelLean` 模式預計進入穩定版的時間點尚未公告，值得持續關注。

3. **Issue #32828 假速率限制 Bug 修復**：此 Bug 影響廣泛，仍是 open 狀態，需關注官方是否在近期版本中修復。

4. **OpenAI `chat-latest` 別名穩定性**：此別名指向「移動目標」（changing API alias），長期穩定性存疑，需觀察社群回饋與官方文件更新。

5. **ClawRouter 生態系統發展**：第三方 LLM 路由器 ClawRouter 帶來了 USDC 付款與 41+ 模型支援的新模式，值得觀察是否成為 OpenClaw 生態系的重要基礎設施。

6. **`localModelLean` 對自架用戶的影響**：此模式若進入穩定版，將對使用弱硬體跑 Ollama/本地模型的用戶帶來顯著改善，關注測試回饋。

7. **Google Meet 語音整合成熟度**：v2026.5.4 引入的 Gemini 實時語音功能尚屬新鮮，社群實際使用回饋與穩定性報告值得追蹤。

---

## 來源連結

- [Blink Blog - What's New in OpenClaw May 2026](https://blink.new/blog/openclaw-whats-new-may-2026)
- [FintechExtra - OpenClaw v2026.5.7](https://www.fintechextra.com/news/openclaw-version-202657-update-key-enhancements-and-fixes-481)
- [GitHub - openclaw/openclaw Releases](https://github.com/openclaw/openclaw/releases)
- [GitHub Issue #32828 - False rate limit](https://github.com/openclaw/openclaw/issues/32828)
- [GitHub Issue #71160 - DeepSeek V4 Pro tool-call bug](https://github.com/openclaw/openclaw/issues/71160)
- [DEV Community - Best LLM for OpenClaw 2026](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)
- [Medium - I Wired OpenClaw to Claude, ChatGPT & Telegram](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)
- [OpenRouter - OpenClaw Integration](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)
- [LaoZhang AI Blog - OpenClaw API Rate Limit Fix](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)
- [PiunikaWeb - OpenClaw 2026.4.26 Gateway Crashes](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)
- [TweenerTimes - OpenClaw Talk Round 3](https://www.tweenertimes.com/p/openclaw-talk-round-3-the-22-agent)
- [BuildFastWithAI - OpenClaw 2026.5.2: Codex, Grok 4.3](https://www.buildfastwithai.com/blogs/openclaw-2026-5-2-release)
- [GitHub - BlockRunAI/ClawRouter](https://github.com/BlockRunAI/ClawRouter)
- [VibesParking - OpenClaw 2026.4.1 Deep Dive](https://www.vibesparking.com/en/blog/tools/openclaw/2026-04-02-openclaw-2026-4-1-stable-analysis/)
