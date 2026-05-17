# OpenClaw 每日情報 — 2026-05-04

> 資料搜集範圍：過去 24 小時（2026-05-03 ～ 2026-05-04）

---

## 1. 今日重點摘要

1. **v2026.5.2 正式發布，插件管理與 Gateway 效能大幅強化**：npm-first 安裝流程涵蓋外部插件安裝、更新、doctor 修復、依賴回報，以及 beta channel fallback；`openclaw plugins list --json` 新增 package 依賴安裝狀態輸出，腳本可無需載入插件即可偵測缺失依賴。

2. **Gateway 冷啟動效能優化（Lazy-loading）**：v2026.5.2 將早期 runtime discovery、shutdown hooks、cron、channel-config schema metadata、restart sentinels 及 maintenance timers 等全數改為 readiness 完成後才載入，顯著縮短 Gateway 啟動時間。

3. **無效設定現在「fail closed」**：Gateway 啟動與熱重載不再自動還原無效設定，錯誤設定會直接阻止啟動，修復責任明確移交給 `openclaw doctor --fix`，避免以錯誤狀態靜默運行。

4. **Kimi K2.5 成為社群投票第一名模型**：截至 2026-05-02，pricepertoken.com 的社群排行榜顯示 Kimi K2.5 位居 OpenClaw 最佳模型榜首，其後依序為 GLM 4.7 與 Claude Opus 4.6；Kimi K2.6 在 SWE-Bench Pro 上以 58.6% 超越 GPT-5.4（57.7%）。

5. **Grok 4.3 正式加入 bundled catalog 並成為 xAI 預設模型**：v2026.5.2 將 Grok 4.3 設為 xAI provider 的預設 chat 模型，同步更新 TypeBox 1.1.37、AWS SDK 3.1041.0、Microsoft Teams 2.0.9 等依賴套件。

6. **Thread Bindings 架構重構**：以 `threadBindings.spawnSessions` 取代原本分離的 subagent/ACP thread-spawn 開關，thread-bound spawns 預設啟用，簡化多代理協作設定。

7. **WhatsApp 支援 Channel/Newsletter 外送目標**：v2026.5.2 新增對 `@newsletter` 明確路由的支援，不再強制走 DM 路由，並附帶 channel session metadata。

8. **假「API rate limit reached」全域 Bug（Issue #32828）持續未修**：底層 API 完全正常的情況下，OpenClaw 仍對所有模型顯示 rate limit 警告。官方尚未給出修補時間線，建議設定 fallback model 或透過 `openclaw models status` 手動確認各 provider 狀態。

9. **5 月 3 日大量 PR 湧入**：包括 memory 診斷修復、plugin keyed store community access agents、skill 安全 checklist、memory slot 設定、cron session sweep、不完整 tool-use turns 修復、語音 provider 設定、session 修復，以及讓 `/steer` 在只有一個 run 執行時參數可省略等多項改進。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **Sonnet 4.6（$3/$15）為 OpenClaw 推薦預設**：多數使用情境下效能與成本的最佳平衡點；Opus 4.7 因定價調降，已有使用者升為日常主力；Haiku 4.5（$1/$5）適合高流量 cron 任務與 webhook handler。
- **Rate Limit 假報錯（Issue #32828）**：即使 Anthropic API 完全正常，OpenClaw 仍可能在 UI 顯示 rate limit 錯誤。暫時解法：執行 `openclaw doctor --fix` 或設定 fallback model（如 Haiku 4.5）。
- **Auth Cooldown 機制**：連續驗證失敗後 OpenClaw 自動觸發 30–60 分鐘 cooldown，顯示「No available auth profile for anthropic (all in cooldown)」，確認 API key 格式為 `sk-ant-api03-` 開頭。
- **`tool_use.input` 欄位缺失錯誤**：出現 `LLM request rejected: messages.content.tool_use.input field required` 時，表示 session history 損毀，執行 `/new` 開啟新 session 可解決。
- 來源：[Issue #32828](https://github.com/openclaw/openclaw/issues/32828)、[Auth 排錯完整指南](https://www.aifreeapi.com/en/posts/openclaw-api-key-error-troubleshooting-guide)、[Claude 選型建議](https://pricepertoken.com/leaderboards/openclaw)

### Kimi（Moonshot）

- **Kimi K2.6 更新**：SWE-Bench Pro 58.6%，超越 GPT-5.4（57.7%），在 OpenClaw 的 agentic 工具測試中表現領先；多步驟、長鏈任務與中英雙語 agent 場景推薦使用。
- **K2.5 仍是社群票選第一**：截至本報告生成時，社群對 K2.5 的長期穩定性信賴度高於剛發布的 K2.6。
- 來源：[Kimi K2.6 Tech Blog](https://www.kimi.com/blog/kimi-k2-6)、[pricepertoken.com OpenClaw 排行](https://pricepertoken.com/leaderboards/openclaw)、[DeepInfra 最佳模型](https://deepinfra.com/blog/best-models-openclaw-agentic-workloads)

### DeepSeek

- **DeepSeek V4 Pro** 為重度 coding 與 tool-use 場景推薦；**DeepSeek V4 Flash** 為最低成本短問答選項。
- 成本比較：DeepSeek-V3-0324 輸入 token 費用約為 Kimi K2 的 2.5 倍便宜（$0.20/M vs $0.50/M on DeepInfra）。
- 建議透過 OpenRouter 路由，可用單一 API key 存取 300+ 模型，有效規避單一 provider rate limit。
- 來源：[DeepSeek vs Kimi 對比](https://openclawlaunch.com/blog/deepseek-v4-vs-kimi-k2-6-comparison-2026)、[DeepSeek 替代模型設定指南](https://www.getopenclaw.ai/en/help/deepseek-minimax-alternative-models)

### xAI Grok

- **Grok 4.3** 已成為 v2026.5.2 的 bundled xAI 預設模型；原先的 Grok 4.1 Fast（$0.20/M input，2M context）仍是預算首選。
- Grok 4.20 Beta（2M context + 多代理推理）持續測試中，正式版釋出後可能改變多代理工作流組合。
- 來源：[v2026.5.2 Release Notes](https://newreleases.io/project/github/openclaw/openclaw/release/v2026.5.2)、[Grok 模型排行](https://pricepertoken.com/leaderboards/openclaw)

### Google Gemini

- **單一模型 Rate Limit 拖累整個 Provider（Issue #5744）**：某 Google 模型觸發 rate limit 後，OpenClaw 將整個 Google provider 設為 cooldown，其他仍有配額的 Gemini 模型也受影響，問題尚未修復。
- 建議搭配 Composio 橋接進行 Gemini MCP 整合，可自動管理 auth 狀態。
- 來源：[Issue #5744](https://github.com/openclaw/openclaw/issues/5744)

### OpenRouter

- 單一 API key 覆蓋 300+ 模型，適合在多家 provider 間輪替以避開 rate limit；OpenClaw 原生支援 `openrouter/<author>/<slug>` 格式，免額外設定 `models.providers`。
- 來源：[OpenRouter 整合文件](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)

---

## 3. 社群反應與討論亮點

- **v2026.4.26 Gateway 當機後遺症仍在討論**：升級後的 Docker/Unraid 使用者（Issue #73846）回報 gateway 永遠無法進入 healthy 狀態，`openclaw doctor --fix` 在 2026.4.26 版本下甚至會卡住（hang）。目前官方建議回滾至 v2026.4.23（`npm install -g openclaw@2026.4.23`），或等待 v2026.5.x 系列的修復。
  - 來源：[Piunikaweb 報導](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)、[Issue #73846](https://github.com/openclaw/openclaw/issues/73846)

- **「update 後全壞掉」模式引發社群焦慮**：部分使用者表示 v2026.4.24、.25、.26 連續三版都出現重大問題，社群開始討論是否應延遲一到兩個版本再升級的「保守更新策略」。
  - 來源：[Openclaw useless after update 討論](https://github.com/orgs/community/discussions/188842)

- **Discord「Friends of the Crustacean」突破 17.5 萬人**：目前成員數達 175,473，成長速度被拿來與 Midjourney Discord 早期爆發期相比，#models、#help 頻道活躍度極高，成為 LLM 選型第一手討論場所。
  - 來源：[OpenClaw Discord 100K 里程碑報導](https://openclaw.report/news/openclaw-discord-100k-members)

- **r/openclawsetup 社群以安裝排錯為主力討論**：本日多篇貼文聚焦 Gateway 啟動失敗、模型 auth 設定，以及如何用 `openclaw doctor --fix` 自動修復常見問題。
  - 來源：[r/openclawsetup](https://thehiveindex.com/communities/r-openclawsetup/)

- **Medium 實測文引發轉發**：Mohit Aggarwal 發表「I Wired OpenClaw to Claude, ChatGPT & Telegram — Here's What Actually Works」，詳述三家 LLM 串接的實際踩坑紀錄，包含 auth 設定細節與 rate limit 應對策略，在社群廣泛流傳。
  - 來源：[Medium 文章](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)

---

## 4. 值得追蹤的後續議題

1. **Issue #32828 假 Rate Limit Bug 修補時程**：此全域性問題影響所有 provider，官方至今未給出預計修補版本，持續是最多人回報的 open bug。

2. **Issue #5744 Google Provider Cooldown 範圍過大問題**：Gemini 使用者的核心痛點，per-model limit 應只影響該模型而非整個 provider，修復後將大幅改善 Gemini 的穩定性。

3. **Docker/Unraid Gateway 健康狀態修復（Issue #73846）**：`openclaw doctor --fix` 在容器環境下無效的問題，影響自架 NAS 與 homelab 用戶，預計在 v2026.5.x 系列完整解決。

4. **Grok 4.20 Beta 正式版釋出時程**：多代理推理功能若轉正式，可能成為 OpenClaw 多代理工作流的新預設選項，值得密切觀察。

5. **Kimi K2.6 在 OpenClaw 社群的採用速度**：剛更新的 K2.6 在 benchmark 上已超越 GPT-5.4，若社群投票數隨之上升，可能取代 K2.5 成為新的推薦預設模型。

6. **v2026.5.x 插件生態系 npm-first 遷移影響**：外部插件安裝流程的重大重構，開發插件的社群開發者需確認現有插件是否相容新的安裝機制。

---

*報告生成時間：2026-05-04 | 資料來源：GitHub Issues/PRs、Releasebot、newreleases.io、pricepertoken.com、Medium、piunikaweb.com、Discord 社群、r/openclawsetup*
