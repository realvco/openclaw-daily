# OpenClaw 每日情報 2026-04-23

> 搜尋區間：過去 24 小時｜產出時間：2026-04-23

---

## 1. 今日重點摘要

1. **v2026.4.21 安全穩定版釋出**：約 20 小時前發布，修補 CVE-2026-25253 後的後續強化，修復特權提升漏洞、外掛相依性問題、Slack 執行緒錯位及自動化逾時浪費等多項問題；自此版起，內建圖片生成 provider 預設切換為 `gpt-image-2`。

2. **CVE-2026-25253（ClawBleed）追蹤進行中**：此 CVSS 8.8 高危 RCE 漏洞透過 WebSocket origin header 繞過竊取 auth token，曝光時全球逾 40,000 個 OpenClaw 實例受影響，其中 12,000+ 確認可遠端執行。雖然修補版本（v2026.1.29 以上）已釋出，社群仍持續追蹤未更新用戶。

3. **Active Memory 插件正式穩定**：v2026.4.10 引入、v2026.4.12 優化，在每次回覆前自動執行記憶子 agent，主動拉取偏好、歷史 context 與過去對話細節，支援 message / recent / full context 三種模式及 `/verbose` 即時診斷；但 issue #68825 記錄在 v4.15 上超過 120 秒逾時的問題尚待修復。

4. **Anthropic 訂閱配額截止影響持續發酵**：自 2026-04-04 起 Claude Pro/Max 訂閱不再涵蓋 OpenClaw 等第三方工具，用戶必須改用 API Key 或額外計費；實際花費落在 API $3–$60/月 vs Max $100–$200/月，社群反彈聲音持續。

5. **GitHub 高速開發活動**：昨日（2026-04-22）單日新增 issue #70346–#70350 及 PR #70199–#70206，涵蓋 OpenClaw 穩定性強化、gateway 費用彙總、Discord 修復等；整體 repository 24 小時內有 500+ issues 與 500+ PR 更新。

6. **OpenRouter 整合達 623+ 模型**：官方文件正式收錄 OpenRouter 為一線 provider，單一 API key 可存取 Claude、GPT-5、Gemini、DeepSeek、Grok、Mistral 等所有主流模型；ClawRouter（第三方）更進一步支援 41+ 模型、<1ms 路由及 Base/Solana USDC 付款。

7. **GPT-5 家族轉型改善**：v2026.4.14 針對 GPT-5 系列進行 explicit turn 改善及 channel provider 修復，核心程式碼大幅重構以提升整體效能。

8. **Model Auth 狀態卡片（Beta）**：目前 beta 串流新增 Model Auth Status Card，可即時顯示 token 健康狀態與 rate-limit 壓力，搭配 cloud-backed LanceDB 記憶索引，以及針對弱規格本地端部署的 `localModelLean` 精簡模式。

9. **`/tasks` 原生任務面板上線**：用戶可在聊天中輸入 `/tasks` 查看所有連結的背景任務及其狀態，進一步完善 OpenClaw 的 agent 工作流管理能力。

---

## 2. LLM 串接實戰回報

### 2-1. Anthropic Claude 系列

- **費用陷阱**：v2026.4.14 後 Claude Opus 4.6 仍是首選推理模型，但 Anthropic 訂閱截止後費用顯著上升；Haiku 4.5 被多位用戶推薦用於 Tier 1 結構化任務，可省下 80–90% token 成本。
- **OAuth token 誤判 cooldown**：Issue #23996 記錄 provider cooldown 原因被硬編碼為 `rate_limit`，導致 OAuth token（如 ChatGPT Plus 訂閱）的真實錯誤（逾時、帳單問題、auth 失效）全部被誤判為 rate limit，造成無限冷卻。
  - 來源：[Bug #23996 - openclaw/openclaw](https://github.com/openclaw/openclaw/issues/23996)

### 2-2. Google Gemini

- **免費層成為 2026 年「平價英雄」**：Gemini 2.5 Flash/Pro 提供 60 RPM、1,000 RPD 免費配額，吸引大量個人開發者採用。
- **Token 暴增警告**：Gemini 2.5 Pro 在少數幾十次 API 呼叫內即消耗 1.9M+ input tokens，已確認為模型本身行為而非 OpenClaw bug，但缺乏預警機制仍造成用戶費用衝擊。
  - 來源：[OpenClaw Rate Limit Fix Guide - LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)

### 2-3. DeepSeek / OpenRouter

- **透過 OpenRouter 接入 DeepSeek V3 & R1**：官方格式 `openrouter/<author>/<slug>`，不需額外 `models.providers` 設定；Medium 用戶分享了 DeepSeek 透過 OpenRouter 接入的完整教程。
  - 來源：[Connect DeepSeek to OpenClaw via OpenRouter - Medium](https://medium.com/@oo.kaymolly/connect-deepseek-to-openclaw-via-openrouter-7eb19ef61a84)
- **ClawRouter 開源路由器**：支援 41+ 模型、毫秒級路由、Base/Solana 鏈上 USDC 付款，適合需要多 provider 動態切換的進階用戶。
  - 來源：[BlockRunAI/ClawRouter - GitHub](https://github.com/BlockRunAI/ClawRouter)

### 2-4. xAI Grok / Mistral

- Grok 3 & 4、Mistral Large & Codestral 均已列入 OpenClaw 官方 provider 插件，可熱重載切換，無需重啟 agent。
- 社群普遍反映 Mistral Codestral 在程式碼生成任務中表現穩定，但 context window 限制仍是痛點。
  - 來源：[6 best LLM routers for OpenClaw in 2026 - DEV Community](https://dev.to/sophiaashi/6-best-llm-routers-for-openclaw-in-2026-17oa)

### 2-5. 通用踩坑紀錄

- **Gateway 不傳遞 ClawHub auth token**：Issue #52949 記錄 gateway 未將 ClawHub auth token 帶入 API 呼叫，導致持續性 429 及「Available Skills」清單為空。
  - 來源：[Bug #52949 - openclaw/openclaw](https://github.com/openclaw/openclaw/issues/52949)
- **快速修復指令**：`openclaw doctor --fix` 可自動解決大多數 401 Unauthorized、429 Rate Limit、gateway token 不符等常見錯誤。
  - 來源：[OpenClaw API Key Error Troubleshooting - AI Free API](https://www.aifreeapi.com/en/posts/openclaw-api-key-error-troubleshooting-guide)

---

## 3. 社群反應與討論亮點

- **Discord「Friends of the Crustacean 🦞🤝」（173,236 成員）**：今日活躍討論包含 Active Memory 插件的實際效能評測、Anthropic 訂閱截止後的替代方案比較，以及 CVE-2026-25253 的修補進度確認。
  - 社群邀請連結：[discord.com/invite/clawd](https://discord.com/invite/clawd)

- **GitHub Discussions #188842**：標題為「Openclaw useless now after update」的討論串在社群引發共鳴，主要抱怨 Anthropic 訂閱截止後費用暴增，及 Active Memory 插件在低配硬體上的逾時問題。
  - 來源：[Discussion #188842 - github.com/orgs/community](https://github.com/orgs/community/discussions/188842)

- **OpenClaw Ecosystem Digest**：agents-radar issue #544 彙整了 2026-04-12 的生態系概況，包含 ClawRouter、Composio 整合、各 LLM 排行等，是觀察社群脈動的重要定期資源。
  - 來源：[Issue #544 - duanyytop/agents-radar](https://github.com/duanyytop/agents-radar/issues/544)

- **r/OpenClaw 熱門討論**：「如何在 Anthropic 訂閱截止後降低 OpenClaw 使用成本」成為本週最熱門話題，主要建議包括：Haiku 4.5 分流、OpenRouter 整合、本地 Ollama 模型搭配 `localModelLean` 模式。

---

## 4. 值得追蹤的後續議題

| # | 議題 | 優先度 | 狀態 |
|---|------|--------|------|
| 1 | **CVE-2026-25253 未更新實例清查**：全球仍有大量舊版實例暴露，建議自查版本是否 ≥ v2026.1.29 | 🔴 高 | 持續追蹤 |
| 2 | **Active Memory 120s 逾時 bug（Issue #68825）**：v4.15 上的 qmd update chain 逾時問題尚無 ETA | 🟡 中 | 待修復 |
| 3 | **Provider cooldown 誤判修復（Issue #23996）**：硬編碼 `rate_limit` 原因導致 OAuth token 用戶受害，PR 仍在審查中 | 🟡 中 | 審查中 |
| 4 | **Gateway ClawHub auth token 未傳遞（Issue #52949）**：影響 ClawControl 技能列表顯示，尚無官方修復時程 | 🟡 中 | 待修復 |
| 5 | **Anthropic 訂閱新政的長期影響**：API Key 強制計費後，OpenClaw 用戶的實際月費分佈與流失率值得持續觀察 | 🟢 低 | 長期觀察 |
| 6 | **GPT-5 家族 explicit turn 相容性**：v2026.4.14 的改善是否完整解決 GPT-5 在複雜 agent 工作流的相容問題，需實測驗證 | 🟢 低 | 社群回報中 |

---

## 來源索引

- [Releases · openclaw/openclaw](https://github.com/openclaw/openclaw/releases)
- [OpenClaw v2026.4.21 Release - Efficient Coder](https://www.xugj520.cn/en/archives/openclaw-2026-4-21-release-updates.html)
- [OpenClaw 2026.4.20 Release - Efficient Coder](https://www.xugj520.cn/en/archives/openclaw-2026-release-update-fixes.html)
- [OpenClaw 2026.4.10: Active Memory Plugin - OpenClaw Playbook Blog](https://www.openclawplaybook.ai/blog/openclaw-2026-4-10-release-codex-active-memory/)
- [OpenClaw v2026.4.12: Active Memory & Exec-Policy CLI - OpenClaw Launch](https://openclawlaunch.com/news/openclaw-v2026-4-12-active-memory-codex-lm-studio-exec-policy)
- [CVE-2026-25253 - OpenClaw RCE Vulnerability - SOCRadar](https://socradar.io/blog/cve-2026-25253-rce-openclaw-auth-token/)
- [NVD - CVE-2026-25253](https://nvd.nist.gov/vuln/detail/CVE-2026-25253)
- [Bug #23996: False provider cooldown - openclaw/openclaw](https://github.com/openclaw/openclaw/issues/23996)
- [Bug #52949: Gateway auth token bypass - openclaw/openclaw](https://github.com/openclaw/openclaw/issues/52949)
- [OpenClaw + Claude Code Costs 2026 - shareuhack.com](https://www.shareuhack.com/en/posts/openclaw-claude-code-oauth-cost)
- [OpenRouter Integration - OpenRouter Docs](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)
- [ClawRouter - BlockRunAI/ClawRouter GitHub](https://github.com/BlockRunAI/ClawRouter)
- [Best Models for OpenClaw - haimaker.ai](https://haimaker.ai/blog/best-models-for-clawdbot/)
- [OpenClaw Model Providers Docs](https://docs.openclaw.ai/concepts/model-providers)
- [Active Memory Docs - OpenClaw](https://docs.openclaw.ai/concepts/active-memory)
- [OpenClaw Release Notes April 2026 - Releasebot](https://releasebot.io/updates/openclaw)
- [OpenClaw Ecosystem Digest 2026-04-12 - agents-radar Issue #544](https://github.com/duanyytop/agents-radar/issues/544)
- [Discussion #188842 - GitHub Community](https://github.com/orgs/community/discussions/188842)
