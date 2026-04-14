# OpenClaw 每日情報 2026-04-14

> 資料收集時間：2026-04-14 | 涵蓋過去 24 小時動態

---

## 1. 今日重點摘要

1. **OpenClaw v2026.4.12 發布——LM Studio 本地模型整合與 Active Memory 深化** — 最新版本新增 LM Studio 捆綁 provider，支援本地 onboarding、執行時模型動態探索、stream preload 與 memory-search embeddings，讓完全本地運行（無需雲端 API）的自主 agent 成為可能。Active Memory plugin 進一步強化 transcript 持久化與 `/verbose` 即時監控。
   - 來源：[Release v2026.4.12 · openclaw/openclaw](https://github.com/openclaw/openclaw/releases/tag/v2026.4.12)

2. **OpenClaw v2026.4.11 發布——Claude CLI MCP Bridge 上線** — 新增透過 loopback MCP bridge 向背景 Claude CLI 執行公開 OpenClaw 工具，並改用 stdin + stream-json 部分訊息串流方式，顯著降低 Claude CLI 整合的延遲。另加入前向相容的 `openai-codex/gpt-5.4-mini` 模型，以及 ChatGPT 對話匯入功能（Memory Palace diary subtab）。
   - 來源：[Release v2026.4.11 · openclaw/openclaw](https://github.com/openclaw/openclaw/releases/tag/v2026.4.11)

3. **Anthropic 訂閱方案封鎖風波持續發酵——135K+ 用戶費用暴增 50 倍** — 自 4/4 Anthropic 宣布取消 Pro/Max 訂閱對第三方 agent 工具（含 OpenClaw）的涵蓋後，用戶轉為獨立的 Extra Usage 按量計費，實際帳單暴增 50 倍。目前社群建議替代方案為：遷移至 Claude CLI backend（由本地 `claude` 執行檔自動處理 token 刷新），或改用 Kimi K2.5 / DeepSeek V3.2 等低成本替代模型。
   - 來源：[Anthropic blocks OpenClaw from Claude subscriptions | TNW](https://thenextweb.com/news/anthropic-openclaw-claude-subscription-ban-cost) | [50x Cost Spike | ThePlanetTools.ai](https://theplanettools.ai/blog/anthropic-blocks-openclaw-third-party-tools-claude-subscriptions-april-2026) | [Medium - Fix OpenClaw Missing Auth](https://medium.com/@ulmeanuadrian/anthropic-just-cut-off-my-ai-agents-heres-how-i-fixed-it-in-20-minutes-741384d27080)

4. **Kimi K2.5 依舊領跑社群最佳模型排行（截至 4/12）** — 依 pricepertoken.com 開發者社群投票，Kimi K2.5 穩居首位，其次為 GLM 4.7、Claude Opus 4.5。OpenRouter API 呼叫量排全球第 3（僅次 Claude 與 Gemini），費用僅 Claude Opus 4.5 的約 5%（$0.50/$1.50 per M tokens）。社群稱其為「Anthropic 計費危機後的最佳逃生出口」。
   - 來源：[Best Model for OpenClaw | pricepertoken.com](https://pricepertoken.com/leaderboards/openclaw) | [Kimi K2.5 Free on OpenClaw | Vertu](https://vertu.com/ai-tools/openclaw-drops-bombshell-kimi-k2-5-becomes-first-free-premium-model/)

5. **Native Codex Provider 整合——`codex/gpt-*` 與 `openai/gpt-*` 路徑分離** — v2026.4.10 起，`codex/gpt-*` 模型使用獨立的 Codex 認證、thread 管理、模型探索與 compaction 路徑，`openai/gpt-*` 保持原 OpenAI provider 路徑，避免兩者互相干擾。新增 plugin-level app-server harness 支援自訂 Codex 工作流。
   - 來源：[OpenClaw 2026.4.10 Changelog Deep Dive | Vibe Sparking AI](https://www.vibesparking.com/en/blog/engineering/2026-04-11-openclaw-2026-4-10-changelog/) | [OpenClaw 2026.4.10 Release | AI News Detail](https://blockchain.news/ainews/openclaw-2026-4-10-release-active-memory-plugin-mlx-local-talk-mode-codex-harness-and-ssrf-hardening-latest-ai-platform-update-analysis)

6. **WhatsApp 音訊轉錄漏跑 Bug（Issue #65978，4/13 回報）** — 當 agent 處於 idle 狀態且 WhatsApp 直接傳遞音訊時，轉錄程序被跳過，訊息無法被正確處理。尚未有官方修復，建議暫時透過 `/wake` 指令手動喚醒 agent 後再發送語音訊息。
   - 來源：[Issue #65978 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/65978)

7. **QQ Bot 圖片下載故障（Issue #65917，4/13 回報）** — 從 v2026.4.2 升級至 v2026.4.11 後，QQ 頻道圖片下載功能失效，疑與新版 CQ code 解析邏輯變更有關。已有數位用戶確認可重現，社群正在整理最小復現步驟。
   - 來源：[Issues · openclaw/openclaw](https://github.com/openclaw/openclaw/issues)

8. **OpenClaw 生態統計持續創新高** — 截至 2026 年 4 月：346K+ GitHub Stars（1 月起成長 >225%）、月活躍用戶 320 萬、月網站流量 3,800 萬（月增 40%+）、Discord 社群 170,394 人、ClawHub plugin 生態超過 15,000 個社群技能。
   - 來源：[OpenClaw hit 346K GitHub stars | openclawvps.io](https://openclawvps.io/blog/openclaw-statistics)

9. **官方 Discord 禁止加密貨幣相關討論** — OpenClaw 官方 Discord 伺服器近期宣佈全面禁止比特幣及其他加密貨幣提及，以維持社群聚焦在 AI 開發討論，並防止財務投機內容分散開發者注意力。
   - 來源：[OpenClaw Shocks Community as Bitcoin and Crypto Mentions Get Banned | MEXC News](https://www.mexc.com/news/771475)

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **Anthropic 訂閱封鎖——遷移至 Claude CLI Backend 為最佳解** — 4/4 起 Pro/Max 訂閱不再涵蓋 OpenClaw，OAuth token 開始出現 60 秒 timeout。已驗證有效的遷移方式：在 OpenClaw 設定中切換至 `claude_cli` backend，由本地安裝的 `claude` 執行檔自動負責認證、token 刷新與 session 管理。純 API Key 方式不受影響，但費用為舊訂閱的約 50 倍。
  - 暫時解法：執行 `openclaw models auth setup-token` 或在 `providers.anthropic` 設定中指定 `backend: "claude_cli"`
  - 來源：[Fix OpenClaw Missing Auth After Anthropic Changes | Medium](https://medium.com/@ulmeanuadrian/anthropic-just-cut-off-my-ai-agents-heres-how-i-fixed-it-in-20-minutes-741384d27080) | [Claude Code Users Face New Fees | TipRanks](https://www.tipranks.com/news/claude-code-users-face-new-fees-for-openclaw-as-anthropic-hikes-price)

- **Claude CLI MCP Bridge（v2026.4.11 新功能）** — OpenClaw 現可透過 loopback MCP bridge 向背景 Claude CLI 執行公開 OpenClaw 工具，整合 stdin + stream-json 串流，適合不願進行完整 API 遷移、仍想維持本地 Claude 環境的用戶。
  - 來源：[Release v2026.4.11 · openclaw/openclaw](https://github.com/openclaw/openclaw/releases/tag/v2026.4.11)

- **長上下文與工具鏈穩定性仍居首** — 儘管費用大幅上漲，Claude Opus 4.5 在多步驟工具鏈、prompt-injection 防禦方面仍領先主流模型，社群建議僅將 Claude 用於高精度需求任務，日常自動化切換至 Kimi K2.5 或 DeepSeek V3.2。
  - 來源：[Best AI Model for OpenClaw 2026 | Get AI Perks](https://www.getaiperks.com/en/blogs/18-best-ai-model-openclaw)

### GPT / Codex（OpenAI）

- **Codex Provider 路徑正式獨立（v2026.4.10+）** — `codex/gpt-*` 模型使用 Codex 原生認證機制，與 `openai/gpt-*` 路徑完全分離。前向相容加入 `openai-codex/gpt-5.4-mini`（v2026.4.11），方便測試次世代 GPT 系列。
- **429 後仍無自動 fallback** — OpenClaw 偵測到 429 後將整個 provider 冷卻，不自動切換備用 provider，需在設定中明確指定 `fallback_model` 方可維持不中斷服務。
  - 來源：[OpenClaw API Rate Limit Reached — Fix & Prevention Guide](https://www.getopenclaw.ai/help/rates-limits-quota-management)

### Gemini（Google）

- **Per-model Rate Limit 觸發 Provider 全面冷卻問題** — 已知 Bug（Issue #5744）：單一 Google 模型達到 rate limit 時，會觸發整個 Google provider 的冷卻，封鎖其他仍有配額的 Gemini 模型。官方尚未修復，建議暫時拆分為多個獨立 provider 設定。
  - 來源：[Issue #5744 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/5744)

### Kimi K2.5（Moonshot AI）

- **「Anthropic 封鎖後最受歡迎替代方案」** — OpenRouter 上以 `openrouter/moonshotai/kimi-k2.5` 引用，$0.50/$1.50 per M tokens，MoE 架構優化工具使用與多步驟規劃。與 Claude 相比費用節省 95%，社群在 Anthropic 計費事件後大量湧入。API 呼叫量已佔 OpenRouter 全站第 3。
  - 多位開發者回報：用於日常 Email 管理、行事曆調度、GitHub PR 自動回覆等工作流，穩定性良好，長對話（>50 輪）後偶有 context 截斷問題。
  - 來源：[Kimi K2.5 Free on OpenClaw | Vertu](https://vertu.com/ai-tools/openclaw-drops-bombshell-kimi-k2-5-becomes-first-free-premium-model/) | [Power OpenClaw for Pennies with Kimi K2 & Codex | Substack](https://trilogyai.substack.com/p/power-openclaw-for-pennies-with-kimi)

### DeepSeek V3.2

- **最低 CP 值方案（$0.28/$0.42 per M tokens）** — 164K context window，SWE-Bench 約 60%，適合結構化確定性任務。Anthropic 封鎖後，DeepSeek 成為第二熱門替代選擇，但複雜多步驟 agent 工作流仍建議搭配 Claude 或 Kimi K2.5。
  - 來源：[OpenClaw Model Comparison: Real Cost Per Task | BetterClaw](https://www.betterclaw.io/blog/openclaw-model-comparison)

### LM Studio（本地模型，v2026.4.12 新增）

- **首個官方捆綁的本地 OpenAI 相容 provider** — v2026.4.12 新增 LM Studio 原生支援，包含 onboarding 引導、執行時模型探索、stream preload 與 memory-search embeddings。適合需要資料隱私或無網路環境的部署場景，是繼 MLX 語音之後 OpenClaw「完全本地化」拼圖的重要一塊。
  - 來源：[Release v2026.4.12 · openclaw/openclaw](https://github.com/openclaw/openclaw/releases/tag/v2026.4.12)

### OpenRouter（統一代理）

- **ClawRouter 開源——41+ 模型、<1ms 路由** — BlockRunAI 發布 ClawRouter，agent-native LLM router 專為 OpenClaw 優化，支援 41+ 模型，路由延遲低於 1ms，並支援透過 x402 協定在 Base 與 Solana 鏈上使用 USDC 進行 API 費用支付。
  - 來源：[BlockRunAI/ClawRouter · GitHub](https://github.com/BlockRunAI/ClawRouter)

---

## 3. 社群反應與討論亮點

### GitHub 熱門議題（近 24 小時）

- **Issue #65978**（2026-04-13 開啟）：WhatsApp 直接傳遞音訊至 idle agent 時轉錄被跳過，影響語音驅動工作流用戶，社群積極提供重現步驟。
- **Issue #65917**（2026-04-13 開啟）：QQ Bot 從 v2026.4.2 升級至 v2026.4.11 後圖片下載失效，疑似 CQ code 解析邏輯 regression。
- **主 Repo 統計**：356,211 Stars、72,179 Forks、5,000+ open issues、5,000+ open PRs，最近更新：2026-04-13。
- 來源：[Issues · openclaw/openclaw](https://github.com/openclaw/openclaw/issues) | [openclaw · GitHub](https://github.com/openclaw)

### Discord 社群（170,394 人）

- **Anthropic 費用風波餘震**：「#provider-help」頻道持續湧入關於費用暴增的求助貼，版主整理了「Claude CLI backend 遷移步驟」置頂貼。
- **LM Studio 本地化熱議**：v2026.4.12 發布後，本地模型話題熱度上升，用戶分享 LM Studio + Llama 3.3 70B 與 Kimi K2.5 的對比測試結果。
- **Active Memory Plugin 試用心得**：多位用戶分享開啟後 agent「記憶力」改善顯著，但部分反映 `/verbose` 模式輸出過於冗長，希望官方提供更細粒度的日誌控制。
- **加密貨幣禁令引發爭議**：部分社群成員對禁止加密貨幣討論的政策表示不滿，認為 OpenClaw 的 Web3 付款整合（如 ClawRouter 的 USDC 支付）與此政策存在矛盾。

### 部落格與媒體

- [TNW](https://thenextweb.com/news/anthropic-openclaw-claude-subscription-ban-cost)：深入報導 Anthropic 封鎖事件，分析對 OpenClaw 生態系統的長期影響。
- [Vibe Sparking AI](https://www.vibesparking.com/en/blog/engineering/2026-04-11-openclaw-2026-4-10-changelog/)：v2026.4.10 Changelog 詳細技術解析，含 Active Memory 架構圖。
- [Blink Blog](https://blink.new/blog/openclaw-2026-4-9-whats-new-update-guide)：OpenClaw 2026.4.9 新功能（Dreaming、Memory & Security）使用教學。
- [trilogyai Substack](https://trilogyai.substack.com/p/power-openclaw-for-pennies-with-kimi)：「Kimi K2 + Codex 以極低成本驅動 OpenClaw」實戰指南，附費用計算對比表。
- [MindStudio](https://www.mindstudio.ai/blog/anthropic-openclaw-ban-oauth-authentication)：技術面剖析 Anthropic OAuth 認證封鎖機制與替代方案評比。

---

## 4. 值得追蹤的後續議題

| 議題 | 說明 | 優先級 |
|------|------|--------|
| **Anthropic Extra Usage 費用正式化（持續）** | 135K+ 用戶仍在評估是否遷移 provider，Claude CLI backend 方案普及速度與官方文件完善度值得關注 | 🔴 高 |
| **WhatsApp Idle Agent 音訊轉錄 Bug（Issue #65978）** | 4/13 回報，影響語音驅動工作流，需關注官方修復時程 | 🔴 高 |
| **QQ Bot 圖片下載失效 regression（Issue #65917）** | 升級至 v2026.4.11 後出現，中文用戶受影響較多，官方確認中 | 🟠 中高 |
| **LM Studio Provider 生產穩定性驗證** | v2026.4.12 剛發布，本地模型生態整合是否具備足夠穩定性供生產環境使用，需更多社群回饋 | 🟠 中高 |
| **Google Gemini Per-model Rate Limit Bug（Issue #5744）** | 單一模型超額觸發整個 Google provider 冷卻，影響多模型並行工作流，官方尚未修復 | 🟠 中高 |
| **Active Memory Plugin 長期記憶品質評估** | v2026.4.12 加入 transcript 持久化，需觀察長期使用後的記憶召回準確率與資料安全性 | 🟡 中 |
| **ClawRouter USDC 支付合規性** | Web3 付款整合在官方 Discord 禁止加密貨幣的政策下如何定位，值得觀察官方態度 | 🟡 中 |
| **Kimi K2.5 長對話 Context 截斷問題** | 社群回報 >50 輪對話後偶有截斷，Moonshot 官方是否有 OpenClaw 專屬最佳化計畫 | 🟢 低 |

---

*本報告由自動化流程於 2026-04-14 生成，資料來源涵蓋 GitHub、X.com、Discord、社群論壇、新聞媒體與技術部落格。*

Sources:
- [Release v2026.4.12 · openclaw/openclaw](https://github.com/openclaw/openclaw/releases/tag/v2026.4.12)
- [Release v2026.4.11 · openclaw/openclaw](https://github.com/openclaw/openclaw/releases/tag/v2026.4.11)
- [Release v2026.4.10 · openclaw/openclaw](https://github.com/openclaw/openclaw/releases/tag/v2026.4.10)
- [Issues · openclaw/openclaw](https://github.com/openclaw/openclaw/issues)
- [Issue #65978 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/65978)
- [Issue #5744 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/5744)
- [Issue #32828 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/32828)
- [Anthropic blocks OpenClaw from Claude subscriptions | TNW](https://thenextweb.com/news/anthropic-openclaw-claude-subscription-ban-cost)
- [50x Cost Spike: Anthropic Cuts Off 135K Claude | ThePlanetTools.ai](https://theplanettools.ai/blog/anthropic-blocks-openclaw-third-party-tools-claude-subscriptions-april-2026)
- [Fix OpenClaw Missing Auth After Anthropic Changes | Medium](https://medium.com/@ulmeanuadrian/anthropic-just-cut-off-my-ai-agents-heres-how-i-fixed-it-in-20-minutes-741384d27080)
- [Best Model for OpenClaw — LLM Rankings | pricepertoken.com](https://pricepertoken.com/leaderboards/openclaw)
- [Kimi K2.5 Free on OpenClaw | Vertu](https://vertu.com/ai-tools/openclaw-drops-bombshell-kimi-k2-5-becomes-first-free-premium-model/)
- [OpenClaw 2026.4.10 Changelog Deep Dive | Vibe Sparking AI](https://www.vibesparking.com/en/blog/engineering/2026-04-11-openclaw-2026-4-10-changelog/)
- [OpenClaw hit 346K GitHub stars | openclawvps.io](https://openclawvps.io/blog/openclaw-statistics)
- [OpenClaw Shocks Community as Crypto Mentions Get Banned | MEXC News](https://www.mexc.com/news/771475)
- [BlockRunAI/ClawRouter · GitHub](https://github.com/BlockRunAI/ClawRouter)
- [Best AI Model for OpenClaw 2026 | Get AI Perks](https://www.getaiperks.com/en/blogs/18-best-ai-model-openclaw)
- [OpenClaw Model Comparison: Real Cost Per Task | BetterClaw](https://www.betterclaw.io/blog/openclaw-model-comparison)
- [Power OpenClaw for Pennies with Kimi K2 & Codex | Substack](https://trilogyai.substack.com/p/power-openclaw-for-pennies-with-kimi)
- [What Is the OpenClaw Ban? | MindStudio](https://www.mindstudio.ai/blog/anthropic-openclaw-ban-oauth-authentication)
- [Claude Code Users Face New Fees | TipRanks](https://www.tipranks.com/news/claude-code-users-face-new-fees-for-openclaw-as-anthropic-hikes-price)
- [Openclaw Release Notes - April 2026 | Releasebot](https://releasebot.io/updates/openclaw)
