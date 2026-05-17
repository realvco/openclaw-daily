# OpenClaw 每日情報 2026-04-09

> 資料收集時間：2026-04-09 | 涵蓋過去 24 小時動態

---

## 1. 今日重點摘要

1. **OpenClaw v2026.4.9 更新——安全強化與 REM 記憶改進** — 發布 REM Backfill、Diary Timeline UI、SSRF 與 node exec injection 強化，以及 character-vibes QA evals，繼續深化 Dreaming 記憶系統的穩定性與安全性。
   - 來源：[OpenClaw 官方 X 帳號](https://twitter.com/@openclaw)

2. **OpenClaw v2026.4.7-1 發布——內建影片音樂生成** — 新增 `video_generate` 工具讓 agent 可直接生成影片；支援跨多個認證 provider 的自動 fallback（圖片、音樂、影片）；新增 `openclaw infer` CLI 整合多模態推理入口；恢復 memory-wiki 知識層與 Webhook 入站插件。
   - 來源：[Release v2026.4.7 · openclaw/openclaw](https://github.com/openclaw/openclaw/releases/tag/v2026.4.7) | [Phemex News](https://phemex.com/news/article/openclaw-launches-version-202647-with-enhanced-features-71603)

3. **Kimi K2.5 奪社群最佳模型榜首（2026-04-09）** — 依 pricepertoken.com 開發者實際投票，Kimi K2.5 登頂，其次為 GLM 4.7、Claude Opus 4.5。Kimi K2.5 在 SWE-Bench 達 76.8%，AIME 2025 達 96.1%，費用僅 Claude Opus 4.5 的 5%。
   - 來源：[OpenClaw LLM Rankings | pricepertoken.com](https://pricepertoken.com/leaderboards/openclaw) | [Kimi K2.5 Tech Blog](https://www.kimi.com/blog/kimi-k2-5)

4. **嚴重 Bug：誤報 Rate Limit（Issue #32828）** — 即使底層 API 正常，OpenClaw 仍對所有模型顯示「API rate limit reached」，根本原因為 provider cooldown 以 `rate_limit` 硬編碼觸發，影響 provider 整體層級（非單一 model），一旦冷卻長達 30–60 分鐘全系列模型皆受阻。
   - 來源：[Issue #32828](https://github.com/openclaw/openclaw/issues/32828) | [AnswerOverflow 討論](https://www.answeroverflow.com/m/1478788645472436264)

5. **ClawWork 開源——以 OpenClaw 作 AI 同事創造 $15K/11 小時** — HKUDS 發布 ClawWork 專案，利用 OpenClaw 作為 AI 「同事」自動執行工作，展示了生產環境中 agent 工作流的商業潛力。
   - 來源：[HKUDS/ClawWork on GitHub](https://github.com/HKUDS/ClawWork)

6. **awesome-openclaw-agents — 162 個生產就緒 agent 模板** — mergisi/awesome-openclaw-agents 提供 19 大類別的 SOUL.md 設定，涵蓋從客服 bot 到程式碼審查 agent，社群持續提交貢獻。
   - 來源：[mergisi/awesome-openclaw-agents](https://github.com/mergisi/awesome-openclaw-agents)

7. **OpenClaw 生態規模突破 346K GitHub Stars** — 從 1 月的 106,124 顆星成長至現在的 346,000+（成長 >225%），Discord 社群達 168,405 人，已遷移至非盈利基金會，獲 OpenAI 支持，可用 skill 從 2,857 增至 13,729 個。
   - 來源：[KDnuggets](https://www.kdnuggets.com/openclaw-explained-the-free-ai-agent-tool-going-viral-already-in-2026)

8. **X.com 創作者自動化熱潮** — 多位創作者分享以 OpenClaw 自動化整個內容流程（研究→撰寫→排程），24/7 運行。但直接串接 Twitter API 需 $100/月，社群建議改用 OpenTweet 作為橋接。
   - 來源：[Prajwal Tomar on X](https://x.com/PrajwalTomar_/status/2025496783904969119) | [OpenTweet Blog](https://opentweet.io/blog/openclaw-twitter-setup-complete-guide)

9. **多語言控制 UI 正式支援繁體中文** — v2026.4.7 Control UI 加入本地化，包含繁體中文、簡體中文、日文、韓文、德文、法文、西班牙文等，大幅降低非英語用戶門檻。
   - 來源：[Releasebot - Openclaw April 2026](https://releasebot.io/updates/openclaw)

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **OAuth Rate Limit 誤觸發（普遍問題）** — 社群大量回報以 Claude OAuth 串接時出現「API rate limit reached. Please try again later.」，即使 Anthropic API 完全正常可用。確認為 OpenClaw 端 cooldown 邏輯 bug，非 Claude 本身問題。一旦冷卻觸發，30–60 分鐘內 Claude 全系列模型皆被封鎖（包含 Sonnet、Opus）。
  - 暫時解法：執行 `openclaw doctor --fix` 或手動執行 `openclaw models auth setup-token` 重置憑證
  - 來源：[AnswerOverflow](https://www.answeroverflow.com/m/1478788645472436264) | [Issue #32828](https://github.com/openclaw/openclaw/issues/32828)

- **OAuth 大 context window 不相容** — 使用 1M token context window 版本時，OAuth 認證會失效，需切換至 Claude Sonnet 4.6 或 Opus 4.6 的 200K 標準版。OAuth 認證格式需為 `{ "type": "token", "provider": "anthropic", "token": "..." }`，使用 `"type": "api_key"` 格式會導致錯誤。
  - 來源：[OpenClaw Fix 429 Rate Limit - LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)

- **長上下文與工具鏈穩定性領先** — Claude 在多步驟工具鏈、prompt-injection 防禦方面持續優於其他主流模型，creator Peter Steinberger 明確推薦 Anthropic Max/Pro + Opus 4.6 作為 coding agent 首選。
  - 來源：[Get AI Perks - Best AI Model for OpenClaw](https://www.getaiperks.com/en/blogs/18-best-ai-model-openclaw)

### GPT-5.4（OpenAI）

- **Computer Use 原生支援** — GPT-5.4 原生支援 Computer Use，在 OSWorld benchmark 達 75% 成功率，是目前唯一原生支援此功能的主流 LLM，對需要 GUI 操作的 agent 工作流具顯著優勢。
- **429 後無自動 fallback** — OpenClaw 偵測 429 後將 provider 整體冷卻，不會自動切換備用，需手動設定 `fallback_model` 設定才能維持不中斷服務。
  - 來源：[LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)

### Gemini（Google）

- **免費額度「最友善」** — 每分鐘 60 次、每日 1,000 次免費請求，被社群稱為「free-tier hero」。Gemini thought-signature sanitation 已在 OpenClaw gateway 中實裝，減少推理過程雜訊。
- **Gemma 4 本地化零費用方案** — OpenClaw + Gemma 4 + Ollama 組合可完全本地運行，適合隱私需求高或離線環境。
  - 來源：[OpenClaw Docs - Model Providers](https://docs.openclaw.ai/concepts/model-providers)

### DeepSeek

- **V3.2 CP 值最高** — $0.28/M input、$0.40/M output，164K context window。SWE-bench 約 60%（vs Claude Opus 4.5 的 89%），適合結構化確定性任務，複雜多步驟自適應推理場景仍建議搭配 Claude。
  - 來源：[haimaker.ai - Best DeepSeek Models for OpenClaw](https://haimaker.ai/blog/best-deepseek-models-for-openclaw/)

### Kimi K2.5（Moonshot AI）

- **社群投票榜首，成本僅 Claude 5%** — 透過 OpenRouter 以 `openrouter/moonshotai/kimi-k2.5` 使用，MoE 架構，特別針對工具使用、多步驟規劃與程式碼生成優化。SWE-Bench Verified 76.8%、AIME 2025 96.1%、BrowseComp 78.4%（agent swarm 模式）。OpenRouter API 呼叫量排名全球第 3（僅次於 Claude 和 Gemini）。
  - 來源：[pricepertoken.com](https://pricepertoken.com/leaderboards/openclaw) | [Kimi K2.5 Hugging Face](https://huggingface.co/moonshotai/Kimi-K2.5)

### Grok（xAI）

- **Grok 4.1 Fast — 200 萬 token 視窗** — 費用 $0.20/$0.50（input/output per M tokens），比 Claude Opus 4.5 便宜約 75 倍，在客服與深度研究工作流中有顯著成本優勢，超大上下文視窗特別適合需要處理大量文件的場景。
  - 來源：[haimaker.ai - Best Models for OpenClaw](https://haimaker.ai/blog/best-models-for-clawdbot/)

### OpenRouter（統一代理）

- **原生支援，單金鑰管理多 provider** — 以 `openrouter/<author>/<slug>` 格式引用模型（如 `openrouter/anthropic/claude-sonnet-4.5`），無需額外設定 `models.providers`，可依任務類型自動路由，有效降低整體費用。
  - 來源：[OpenRouter OpenClaw 整合指南](https://openrouter.ai/docs/guides/guides/openclaw-integration)

---

## 3. 社群反應與討論亮點

### GitHub 熱門議題（近 24 小時）

- **Issue #63639**（2026-04-09 由 copernicus-agent 開啟）：功能增強請求，狀態 Open。
- **Issue #63638**（2026-04-09 由 vincentyan2023 開啟）：不當行為 bug 回報（無 crash）。
- **Issue #63631**（2026-04-09 由 taozhiyuai 開啟）：詳情待確認。
- **Issue #32828（高優先）**：誤報 Rate Limit——provider cooldown 邏輯需重構，社群持續追蹤。
- **Discussion #188842**：「Openclaw useless now after update」——v2026.4.x 更新後功能退化廣泛討論，社群反應強烈。
  - 來源：[GitHub Discussion](https://github.com/orgs/community/discussions/188842)
- 主 repo 目前共有約 11,000+ issues、6,500+ pull requests，每日新增數十筆。

### Discord 社群（168,405 人）

- **Memory + Dreaming 組合熱議**：Memory-Wiki + REM Backfill 組合被認為標誌 OpenClaw 正式進入「有持久記憶的 AI agent」時代。
- **零成本方案「Gemini + DeepSeek via OpenRouter」**：被稱為個人開發者最佳零成本高效率組合。
- **Twitter 自動化討論**：多位用戶分享將 OpenClaw 用於內容創作自動化，但直接 Twitter API 月費 $100 成為門檻，OpenTweet 橋接方案受到關注。

### 部落格與媒體

- [DataCamp](https://www.datacamp.com/blog/openclaw-projects)：整理 2026 年 9 個 OpenClaw 實作專案（Reddit digest bot 到自我修復伺服器）。
- [Cybernews](https://cybernews.com/best-web-hosting/run-ai-discord-agent-with-openclaw/)：深入介紹 OpenClaw Discord bot 架構搭建。
- [KDnuggets](https://www.kdnuggets.com/openclaw-explained-the-free-ai-agent-tool-going-viral-already-in-2026)：「OpenClaw Explained: The Free AI Agent Tool Going Viral Already in 2026」
- [Skywork AI](https://skywork.ai/skypage/en/openclaw-latest-updates-2026/2037437426655117312)：2026 年 OpenClaw 最新更新完整指南。

---

## 4. 值得追蹤的後續議題

| 議題 | 說明 | 優先級 |
|------|------|--------|
| **誤報 Rate Limit 根本修復（Issue #32828）** | Provider cooldown 邏輯重構，影響所有串接 Claude OAuth 用戶，多個重複 issue 積累中 | 🔴 高 |
| **Kimi K2.5 生產穩定性驗證** | 剛登頂社群排行榜，實際長期多步驟任務表現仍需更多資料支撐 | 🟠 中高 |
| **v2026.4.7 影片音樂生成穩定性** | 新功能初版，provider fallback 邏輯、API 相容性與輸出品質需持續觀察 | 🟡 中 |
| **ClawWork 商業模式觀察** | HKUDS/ClawWork 以 OpenClaw 創造 $15K/11h 的商業案例，後續擴展值得追蹤 | 🟡 中 |
| **Claude OAuth 冷卻機制調整** | 30–60 分鐘冷卻對生產環境衝擊過大，社群要求可設定時長或手動重置，官方回應待確認 | 🟠 中高 |
| **非盈利基金會轉型後路線圖** | OpenAI 入股後的開源承諾與未來功能方向，社群密切關注 | 🟡 中 |
| **Memory-Wiki × Obsidian 雙向同步** | 目前僅單向匯入，社群期待完整雙向同步功能 | 🟢 低 |
| **Twitter API 成本替代方案成熟度** | OpenTweet 橋接方案是否具備足夠穩定性供生產環境使用，待更多用戶回饋 | 🟢 低 |

---

*本報告由自動化流程於 2026-04-09 生成，資料來源涵蓋 GitHub、X.com、Discord、社群論壇、新聞媒體與技術部落格。*

Sources:
- [GitHub - openclaw/openclaw](https://github.com/openclaw/openclaw)
- [Releases · openclaw/openclaw](https://github.com/openclaw/openclaw/releases)
- [OpenClaw — Personal AI Assistant](https://openclaw.ai/)
- [Best Model for OpenClaw — LLM Rankings | pricepertoken.com](https://pricepertoken.com/leaderboards/openclaw)
- [Issue #32828 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/32828)
- [HKUDS/ClawWork](https://github.com/HKUDS/ClawWork)
- [mergisi/awesome-openclaw-agents](https://github.com/mergisi/awesome-openclaw-agents)
- [KDnuggets - OpenClaw Explained](https://www.kdnuggets.com/openclaw-explained-the-free-ai-agent-tool-going-viral-already-in-2026)
- [Releasebot - Openclaw April 2026](https://releasebot.io/updates/openclaw)
- [haimaker.ai - Best Models for OpenClaw](https://haimaker.ai/blog/best-models-for-clawdbot/)
- [haimaker.ai - Best DeepSeek Models for OpenClaw](https://haimaker.ai/blog/best-deepseek-models-for-openclaw/)
- [Get AI Perks - Best AI Model for OpenClaw 2026](https://www.getaiperks.com/en/blogs/18-best-ai-model-openclaw)
- [LaoZhang AI Blog - OpenClaw Rate Limit](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)
- [AnswerOverflow - Claude OAuth Rate Limit](https://www.answeroverflow.com/m/1478788645472436264)
- [DataCamp - 9 OpenClaw Projects](https://www.datacamp.com/blog/openclaw-projects)
- [Kimi K2.5 Tech Blog](https://www.kimi.com/blog/kimi-k2-5)
- [Prajwal Tomar on X](https://x.com/PrajwalTomar_/status/2025496783904969119)
- [Phemex News - OpenClaw v2026.4.7](https://phemex.com/news/article/openclaw-launches-version-202647-with-enhanced-features-71603)
- [GitHub Discussion #188842](https://github.com/orgs/community/discussions/188842)
- [OpenClaw Docs - Model Providers](https://docs.openclaw.ai/concepts/model-providers)
