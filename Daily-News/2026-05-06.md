# OpenClaw 每日情報 — 2026-05-06

> 資料搜集範圍：過去 24 小時（2026-05-05 ～ 2026-05-06）

---

## 1. 今日重點摘要

1. **v2026.5.4 正式發布（13 小時前上線）**：npm 最新穩定版已更新至 2026.5.4，延續 .3 版對插件安裝、頻道穩定性與 Gateway 程序管理的修復路線，具體 changelog 正由社群成員整理中，建議關注 GitHub Releases 頁面。

2. **Issue #77350：DeepSeek V4 Pro on OpenRouter 出現 400 回歸錯誤**：v2026.5.3-1 引入回歸，對 OpenRouter 路由的 DeepSeek V4 Pro 發出帶有 `reasoning_effort` 參數的請求，導致 400（`Invalid option`）錯誤；目前 OpenRouter 端不接受此欄位，受影響用戶需暫時改用 DeepSeek V3.2 或繞過 `reasoning_effort` 設定，官方正排查中。

3. **Issue #23996：OAuth token auth 觸發無限 provider 冷卻**：hardcoded `rate_limit` 原因碼與 OAuth token 認證機制衝突，一旦觸發即進入永久 cooldown 且無自動復原，完全封鎖所有代理回覆；此 bug 與上週持續未修的 Issue #32828（全域假 rate limit）互為相關，正式修補前建議改用標準 API Key 而非 OAuth token。

4. **Active Memory 新增 operator 層級的對話過濾控制**：`allowedChatIds` 與 `deniedChatIds` 兩個 per-conversation 過濾器正式合入，讓 Memory Recall 只在指定範圍內的對話啟動，對多頻道、多工作區部署的企業用戶而言是重要的隱私隔離機制。

5. **Agent 角色從「收件匣助手」升級為「即時協作者」**：官方路線圖揭示下一波 Agent 能力將涵蓋加入視訊會議、即時聆聽與發言、匯出會議成品、自動恢復分頁，以及在對話中途向 agent brain 請求 tool-backed 協助，是架構層級的重大躍進。

6. **外部插件化進程加速（2026.5.1 起）**：Google Chat、LINE、Matrix、Mattermost、BlueBubbles 等重量級插件正式移出核心套件，改為 npm 發布後按需安裝，核心包體積顯著縮小，插件版本管理亦更靈活。

7. **Discord 與 Telegram 高並發穩定性補丁落地**：訊息排序錯亂、thread-bound session 丟失回覆、bot 確認訊息但未送達回應等三個邊緣案例已在本週修復，Discord 社群（「Friends of the Crustacean 🦞🤝」）反映穩定性明顯提升。

8. **社群模型票選：Kimi K2.5 仍居首，Claude Opus 4.6 排名第二**：截至 2026-05-05，pricepertoken.com 排行榜 Kimi K2.5 維持第一，Claude Opus 4.6 緊追第二，GLM 4.7 第三；Gemini 3.1 Pro 因大 context、低成本、免費 dev tier 被多數教程推薦為「新手預設」，Claude Opus 4.7 則以最高 SWE-bench Pro 成績獲嚴格程式碼審查場景青睞。

9. **OpenClaw 主倉庫 PR 數量持續攀升**：目前開放 PR 達 3,449 筆（較上週增加約 21 筆），已關閉 PR 累計 38,324 筆，GitHub Stars 超過 25 萬，生態擴張速度仍為業界頂尖。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **訂閱封鎖影響持續（第 32 天）**：自 2026-04-04 起 Anthropic 禁止以 Claude Pro/Max 訂閱 OAuth token 用於第三方工具，所有 OpenClaw 用戶必須改用 `sk-ant-api03-` 格式的 Anthropic API Key；`openclaw doctor --fix` 可自動偵測並修正大多數 auth 設定。
- **Issue #23996 直接影響 OAuth 用戶**：使用 OAuth token（含部分企業 SSO 情境）的用戶在觸發冷卻後無法自動恢復，建議立即切換至 API Key 模式。
- **Claude Opus 4.7 嚴格指令遵循獲好評**：在代碼審查、多步驟規劃任務中，Opus 4.7 的指令遵循率顯著優於同級競品，多位部落客實測後推薦作為「嚴格 code review agent」的預設模型。
- 來源：[Issue #23996](https://github.com/openclaw/openclaw/issues/23996)、[Auth 錯誤完整排查指南（aifreeapi.com）](https://www.aifreeapi.com/en/posts/openclaw-api-key-error-troubleshooting-guide)、[Best LLM for OpenClaw 2026（DEV Community）](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### DeepSeek

- **Issue #77350：DeepSeek V4 Pro on OpenRouter 400 回歸錯誤**：v2026.5.3-1 對 OpenRouter 路由的請求附帶 `reasoning_effort` 欄位，OpenRouter 端回傳 400（`Invalid option`），屬版本回歸，預計在 2026.5.4 後修復；短期繞過方式：直接呼叫 DeepSeek 官方 API（非 OpenRouter 路由）或降版至 v2026.5.2。
- **V4 Flash 在高流量場景的 503 問題**：尖峰時段 DeepSeek API 端出現 503，建議在 OpenClaw 設定中啟用 OpenRouter 作為 fallback 路由，或設定指數退避重試。
- **成本優勢仍然突出**：DeepSeek V3.2 輸入費用約 $0.20/MTok，為 Kimi K2 的 2.5 倍便宜，適合高吞吐量、成本敏感的 cron 排程任務。
- 來源：[Issue #77350（GitHub）](https://github.com/openclaw/openclaw/issues/77350)、[OpenRouter DeepSeek 文件](https://openrouter.ai/deepseek)、[Rate Limit 修復指南（laozhang.ai）](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)

### Google Gemini

- **Gemini 3.1 Pro 成為新手推薦預設**：大 context window（適合大型代碼審查）、最低成本/job、免費 dev tier 三項優勢使其成為多數 OpenClaw 教程的「開箱預設」選擇。
- **Issue #5744 仍未修復**：單一 Google 模型達到 rate limit 時，整個 Google provider 被置入 cooldown，其他仍有配額的 Gemini 模型亦受阻，中高流量用戶需透過 OpenRouter 路由各別模型以規避此問題。
- **Gemini 2.5 Flash 免費 tier 適合低流量場景**：適用於低優先度的背景任務、定時摘要等高延遲容忍場景。
- 來源：[Best LLM for OpenClaw: Gemini 3.1 Pro vs GPT-5.5 vs Claude Opus 4.7（DEV Community）](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)、[Issue #5744](https://github.com/openclaw/openclaw/issues/5744)

### GPT（OpenAI）

- **Codex + /goal 跨步驟 agentic 工作流**：v2026.5.2 引入的 `/goal` 指令搭配 ChatGPT Pro 訂閱及 Codex plugin，可描述高層目標讓 OpenClaw 跨多步驟自主執行，是目前最長鏈 agentic 能力之一。
- **GPT-5.5 benchmark 被 Kimi K2.6 超越**：K2.6 在 SWE-Bench Pro 達 58.6%，超過 GPT-5.5 的 57.7%，但社群對 GPT 整合的工具呼叫穩定性仍有較高評價。
- 來源：[OpenClaw 2026.5.2 Release（buildfastwithai.com）](https://www.buildfastwithai.com/blogs/openclaw-2026-5-2-release)

### Mistral

- **Devstral 2512 為 agent loop 首選**：在需要長鏈工具呼叫、多步驟代碼生成的 OpenClaw agent 工作流中，Devstral 2512 的工具呼叫可靠性為 Mistral 系列最高；一般對話場景仍推薦 Large 2512。
- **穩定性優於 DeepSeek**：實測回報 Mistral 端幾乎無 503 問題，對穩定性要求高的生產環境優先考慮。
- 來源：[Best Mistral Models for OpenClaw 2026（haimaker.ai）](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

### Grok（xAI）

- **Grok 4.3 成為 xAI 預設**：v2026.5.2 已將 Grok 4.3 設為 bundled catalog 的 xAI 預設模型，Grok 4.20 Beta（2M context + 多代理推理）正式版仍未釋出。
- 來源：[OpenClaw 2026.5.2 Release](https://www.buildfastwithai.com/blogs/openclaw-2026-5-2-release)

### OpenRouter

- **原生支援 `openrouter/<author>/<slug>` 格式**：無需額外設定 `models.providers`，是跨 provider fallback 的最低阻力方案，可存取 300+ 模型，並可在多家 provider 間自動輪替以規避 rate limit。
- **OpenClaw 社群 Token 使用排行**：OpenRouter 定期更新 OpenClaw 用戶的模型 token 使用排行，反映真實世界的模型選型趨勢，可作為選型參考。
- 來源：[OpenRouter OpenClaw 整合文件](https://openrouter.ai/docs/guides/guides/openclaw-integration)、[Top AI Models Used by OpenClaw on OpenRouter](https://openrouter.ai/collections/openclaw)

---

## 3. 社群反應與討論亮點

- **Medium 實測長文持續熱傳**：Mohit Aggarwal 的「I Wired OpenClaw to Claude, ChatGPT & Telegram — Here's What Actually Works」詳述三家 LLM 串接的完整踩坑過程，含 auth 設定細節、rate limit 應對、Telegram bot 整合，是近期引用量最高的入門實測文。
  - 來源：[Medium 文章](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)

- **「更新後 OpenClaw 完全無用」討論串仍在發酵**：GitHub Community Discussion #188842 的「Openclaw useless now after update」貼文持續累積回覆，主要聚焦 v2026.4.26 造成的 Gateway 崩潰問題，部分用戶因此在 Docker Compose 中 pin 至特定版本，「等兩個版本再升」的保守策略獲越來越多認同。
  - 來源：[Discussion #188842](https://github.com/orgs/community/discussions/188842)、[PiunikaWeb 報導](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)

- **OpenClaw vs Hermes Agent 遷移討論升溫**：howdoiuseai.com 發表「Why everyone is switching from OpenClaw to Hermes Agent in 2026」引發廣泛討論，聚焦 agent 自主度、LLM 相容性、插件生態與部署難度的比較；OpenClaw 陣營認為生態成熟度仍有明顯優勢，Hermes Agent 支持者則強調更簡潔的架構。
  - 來源：[Why everyone is switching from OpenClaw to Hermes Agent（howdoiuseai.com）](https://www.howdoiuseai.com/blog/2026-04-24-why-everyone-is-switching-from-openclaw-to-hermes-)

- **DataCamp 發布「9 OpenClaw 專案建議」**：涵蓋從 Reddit bot 到自癒伺服器的九個實用場景，成為社群新人的熱門入門資源，在 Discord #getting-started 頻道被多次轉發。
  - 來源：[9 OpenClaw Projects to Build in 2026（DataCamp）](https://www.datacamp.com/blog/openclaw-projects)

- **ClawSweeper 每週自動分類 Issues/PRs**：官方維護的 ClawSweeper 工具每週掃描所有 issues 與 PR 並建議可關閉項目，有效協助維護者管理 3,449 筆開放 PR，社群稱其為「最有用的維護輔助工具」。
  - 來源：[GitHub openclaw/clawsweeper](https://github.com/openclaw/clawsweeper)

---

## 4. 值得追蹤的後續議題

1. **Issue #77350 DeepSeek V4 Pro on OpenRouter 400 回歸**：v2026.5.3-1 引入，預計 v2026.5.4 或後續修復，DeepSeek + OpenRouter 用戶應密切追蹤 patch 時程。

2. **Issue #23996 OAuth Token 觸發無限 Provider 冷卻**：與 Issue #32828（假全域 rate limit）相互關聯，兩者合計影響使用 OAuth auth 或遭誤判為 rate limit 的所有用戶，建議追蹤官方修補進度。

3. **Issue #5744 Google Provider 過大 Cooldown 範圍**：Gemini 單一模型限速拖累整個 provider 的問題仍未修復，中高流量 Gemini 用戶的核心痛點，修復後將大幅改善工作流穩定性。

4. **v2026.5.4 完整 Changelog 確認**：發布 13 小時內 changelog 尚未全面整理，建議關注 GitHub Releases 頁面及 Releasebot 通知，確認是否包含 Issue #77350 的修復。

5. **Agent 即時協作功能的開發進度**：官方路線圖揭示的下一波 agent 能力（加入會議、即時語音、成品匯出）若正式落地，將是 OpenClaw 架構的重大轉型，值得持續追蹤具體 ETA。

6. **Kimi K2.6 社群採用速度**：benchmark 已超越 GPT-5.5，若本週社群投票數明顯上升，可能取代 K2.5 成為新預設推薦模型。

7. **Anthropic API 費用衝擊長期評估**：訂閱封鎖已滿一個月，pay-as-you-go 實際費用數據逐漸累積，社群對「API 費用 vs 訂閱」的成本效益討論值得持續追蹤。

---

*報告生成時間：2026-05-06 | 資料來源：GitHub Issues/PRs、npm、Releasebot、pricepertoken.com、haimaker.ai、DEV Community、Medium、howdoiuseai.com、DataCamp、buildfastwithai.com、openrouter.ai、PiunikaWeb、Discord 社群、r/openclaw、X.com*
