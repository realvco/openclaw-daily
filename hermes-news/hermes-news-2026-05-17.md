# Hermes-Agent 每日情報 2026-05-17

> 搜尋時間範圍：過去 24 小時（2026-05-16 ～ 2026-05-17）

---

## 1. 今日重點摘要

1. **v0.14.0「The Foundation Release」昨日（5/16）正式釋出**：本版累計 808 commits、633 個合併 PR、1,393 個檔案異動、新增 165,061 行程式碼，關閉 545 個 Issues（含 12 個 P0、50 個 P1），215 位社群貢獻者參與，為 Hermes-Agent 迄今規模最大的版本。
   - 來源：[GitHub Release v0.14.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.5.16)

2. **Windows 原生支援（Early Beta）正式登場**：首次提供完整 PowerShell 安裝流程、原生 subprocess/PTY 路徑、taskkill 式流程管理、MinGit 自動安裝、Microsoft Store Python stub 偵測，以及前台 Ctrl+C 保留，使 Hermes 無需 WSL 即可在 cmd.exe / PowerShell 執行。已知問題包含幽靈進程殘留、charmap 編碼錯誤等（見後述）。
   - 來源：[GitHub Release v0.14.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.5.16)

3. **OAuth-only 提供商的 OpenAI 相容本地 Proxy**：新增本地 proxy 讓 Claude Pro、ChatGPT Pro、SuperGrok 等僅支援 OAuth 的服務，可直接接入 Codex、Aider、Cline 等工具，解決大量用戶長期反映的「OAuth 提供商無法接入 OpenAI 相容工具」痛點。
   - 來源：[GitHub Release v0.14.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.5.16)

4. **xAI Grok SuperGrok OAuth 整合 + Grok 4.3 context 升至 1M tokens**：xAI Grok 新增 SuperGrok OAuth 提供商，Grok 4.3 context window 從原本上限提升至 100 萬 tokens，配合 Hermes 的 MEMORY.md 長期記憶機制可大幅降低 token 浪費。
   - 來源：[GitHub Release v0.14.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.5.16)

5. **冷啟動省去約 19 秒，瀏覽器 CDP 呼叫速度提升 180 倍**：hermes tools All-Platforms 指令執行時間從 14 秒降至 1.5 秒以內，大幅改善日常使用體驗。

6. **九項新選用技能（Optional Skills）上架**：包含 Hyperliquid 交易、Yahoo Finance 數據、API 測試、統一 EVM 多鏈支援、darwinian-evolver（進化式 agent）、OSINT 調查、pinggy-tunnel、watchers，以及大幅改版的 Notion 整合。
   - 來源：[GitHub Release v0.14.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.5.16)

7. **X（Twitter）搜尋成為一級工具**：支援 OAuth 或 API Key 雙重認證，內建 X 搜尋即可直接在 Hermes 工作流中獲取 X 貼文資料。

8. **Microsoft Teams 端對端整合**：Graph auth、webhook 監聽、pipeline runtime 與輸出交付全部就緒，Teams 正式成為 Hermes 第 19 個訊息平台（含 Teams 外掛則為第 20 個）。

9. **社群 token 成本衝擊討論持續**：Reddit 有用戶回報「兩小時輕度使用燒掉 400 萬 tokens」，Discord 社群建議新手先使用免費或低成本模型熟悉系統後再切換高成本模型。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **Claude Pro OAuth 透過本地 Proxy 接入**：v0.14.0 新增 OpenAI 相容本地 proxy，Claude Pro 訂閱者現在可透過此 proxy 將 Claude 接入 Codex、Aider 等工具，無需 API Key。
  - 來源：[GitHub Release v0.14.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.5.16)
- **成本概估**：使用 Claude Sonnet 4.6（$3/$15 per 1M tokens）的個人部署每月約 $30–80，視 VPS 規格與 token 使用量而定。
  - 來源：[Remote OpenClaw – Hermes Agent Cost Breakdown](https://www.remoteopenclaw.com/blog/hermes-agent-cost-breakdown)
- **Token 開銷警告**：CLI 模式每次請求約有 6–8K input tokens 的 overhead；透過 Telegram/Discord 等 messaging gateway 使用時，overhead 飆升至 15–20K tokens/次，使用高單價模型時需特別注意。
  - 來源：[Remote OpenClaw – Hermes Agent Cost Breakdown](https://www.remoteopenclaw.com/blog/hermes-agent-cost-breakdown)

### GPT（OpenAI）

- **ChatGPT Pro OAuth 透過本地 Proxy 接入**：與 Claude Pro 相同，v0.14.0 新增的本地 proxy 也支援 ChatGPT Pro 訂閱。
  - 來源：[GitHub Release v0.14.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.5.16)
- **GPT-4.1 成本**：$2/$8 per 1M tokens，對多數個人部署為成本與能力的平衡選擇。
  - 來源：[BenchLM.ai LLM Pricing](https://benchlm.ai/llm-pricing)

### Gemini（Google）

- **Gemini 2.5 Pro**：$1.25/$10 per 1M tokens（200K context 以內），支援最多 100 萬 tokens 輸入，適合需要大 context 的長期任務。
- **Gemini 3.1 Flash-Lite**：$0.25/$1.50 per 1M tokens，為目前最具成本效益的新模型選項。
- **免費額度**：Google AI Studio 提供 Gemini 2.5 Flash、2.0 Flash、1.5 Pro 的免費層，速率限制約 10–30 requests/min，適合新手入門測試。
  - 來源：[Gemini API Free Tier 2026](https://pecollective.com/tools/gemini-free-tier-guide/)、[MetaCTO Gemini Pricing](https://www.metacto.com/blogs/the-true-cost-of-google-gemini-a-guide-to-api-pricing-and-integration)

### xAI Grok

- **Grok 4.3 + SuperGrok OAuth**：v0.14.0 新增，Grok 4.3 context window 升至 1M tokens；SuperGrok 訂閱者可直接 OAuth 登入，無需手動設定 API Key。
  - 來源：[GitHub Release v0.14.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.5.16)

### DeepSeek

- **成本最低主力模型之一**：DeepSeek V4 定價 $0.30/$0.50 per 1M tokens，搭配 Hetzner VPS 的預算部署每月總花費約 $6–8。
  - 來源：[Remote OpenClaw – Hermes Agent Cost Breakdown](https://www.remoteopenclaw.com/blog/hermes-agent-cost-breakdown)

### 月成本實際範例

| 設定 | 模型 | 月成本估算 |
|---|---|---|
| 預算方案 | DeepSeek V4 + Hetzner | ~$6–8（含 VPS） |
| 個人進階 | Claude Sonnet 4.6 + 中規格 VPS | ~$30–80 |
| 一般個人使用量 | 任意模型 | $2–15（預算）/ $10–60（進階） |

來源：[Remote OpenClaw](https://www.remoteopenclaw.com/blog/hermes-agent-cost-breakdown)、[Hermes Agent FAQ](https://hermes-agent.nousresearch.com/docs/reference/faq)

---

## 3. 社群反應與討論亮點

- **v0.14.0 正式釋出引起廣泛關注**：AlphaSignal 專文推薦「這個週末安裝 Hermes Agent」，強調「The Foundation Release」代表 Hermes 已達到可在任何平台穩定運行的里程碑。Windows 原生支援尤其讓非 Linux 用戶期待，但已知 Bug 也迅速被回報（見下方）。
  - 來源：[AlphaSignal](https://alphasignalai.substack.com/p/you-should-install-hermes-agent-this)

- **Token 成本衝擊是最熱討論話題**：Reddit 用戶反映 gateway 在 hermes-agent 目錄啟動時會載入開發用的 AGENTS.md 等多餘文件，導致相同對話的 token 成本增加 2–3 倍；社群建議在 home 目錄而非專案目錄啟動 Hermes 可有效避免。
  - 來源：[Hermes Agent Troubleshooting Guide](https://www.getopenclaw.ai/blog/hermes-agent-troubleshooting)

- **Discord 整合 Bug（Issue #12496）**：Discord forum 的新討論串第一則訊息會被 Hermes 忽略，直到 @mention Hermes 後才開始回應；此問題在社群中有一定討論度，開發者已記錄，尚未修復。
  - 來源：[GitHub Issue #12496](https://github.com/NousResearch/hermes-agent/issues/12496)

- **多 agent 協作可見性問題（Issue #14853）**：多 agent 共用 Discord channel 時，各 agent 無法看見彼此的歷史訊息，可能造成重複工作或協調失敗，有用戶提案加入 history injection 機制與 cascade prevention。
  - 來源：[GitHub Issue #14853](https://github.com/NousResearch/hermes-agent/issues/14853)

- **與 OpenClaw 的比較討論熱度上升**：Kilo.ai 分析 1,300 則 Reddit 評論，發現 Hermes Agent 在穩定性、記憶機制、Kanban 多 agent 工作流等方面獲得較高評價；OpenClaw 則在生態系廣度與社群規模上仍佔優勢。
  - 來源：[Kilo.ai – OpenClaw vs Hermes 2026](https://kilo.ai/articles/openclaw-vs-hermes-what-reddit-says)

---

## 4. 值得追蹤的後續議題

1. **Windows Early Beta 已知問題修復進度**：
   - 幽靈進程殘留（重啟後報「already running PID XXXX」）
   - `os.kill(pid, signal)` 在 Windows 無效導致子進程外洩
   - terminal tool 回傳 exit_code 0 但 stdout/stderr 空白
   - charmap codec 路徑編碼錯誤
   - 追蹤：[GitHub Issue #15828](https://github.com/NousResearch/hermes-agent/issues/15828)、[GitHub Issue #16201](https://github.com/NousResearch/hermes-agent/issues/16201)

2. **Issue #12496（Discord forum 首則訊息被忽略）**：影響 Discord 整合工作流，尚未修復，建議使用 Discord 的團隊持續追蹤。
   - 追蹤：[GitHub Issue #12496](https://github.com/NousResearch/hermes-agent/issues/12496)

3. **Issue #14853（多 agent Discord 協作可見性）**：history injection + cascade prevention 功能若實作，將大幅提升多 agent 協作能力，值得關注設計討論進展。
   - 追蹤：[GitHub Issue #14853](https://github.com/NousResearch/hermes-agent/issues/14853)

4. **OpenAI 相容 Proxy 的安全性與穩定性**：v0.14.0 新增的 OAuth 本地 proxy 尚屬早期實作，長時間運行的 token refresh 行為與多客戶端共用場景的穩定性值得測試與追蹤。

5. **v0.15.0 下一版方向**：v0.14.0 以「基礎穩定」為主軸，社群與開發者已開始討論下一版是否聚焦多 agent 協作、安全性強化，或進一步擴展平台支援，值得追蹤 GitHub milestone 動態。
   - 追蹤：[GitHub Releases](https://github.com/NousResearch/hermes-agent/releases)

6. **Autonomous Curator 自我改進循環**：自律策展系統（自動評分、剪枝、整合技能庫）仍屬新功能，長期運作行為與潛在誤刪重要技能的風險需社群持續回報。

---

*報告產生時間：2026-05-17 | 資料來源：GitHub（NousResearch/hermes-agent）、AlphaSignal、Remote OpenClaw、Kilo.ai、BenchLM.ai、Hermes Agent 官方文件等公開平台*
