# OpenClaw 每日情報 2026-04-28

> 資料蒐集範圍：2026-04-27 至 2026-04-28（過去 24 小時）

---

## 1. 今日重點摘要

1. **v2026.4.25 正式發布**：TTS 語音系統全面升級，新增 Azure Speech、Xiaomi MiMo、ElevenLabs v3、Inworld、Volcengine、Local CLI 共六個語音服務商，支援 `/tts latest`、聊天室自動 TTS、per-agent/per-account 個別覆蓋設定。（[來源](https://phemex.com/news/article/openclaw-v2026425-launches-with-enhanced-tts-and-new-voice-providers-76611)）

2. **OpenTelemetry 全面擴展**：模型呼叫、Token 用量、工具迴圈、Harness 執行、外送傳遞、記憶體壓力等場景均新增可觀測性指標，屬性基數受限設計確保低開銷。（[來源](https://releasebot.io/updates/openclaw)）

3. **瀏覽器自動化強化**：Tab URL 安全性提升、iframe 感知角色快照、CDP 就緒調校、headless 一次性啟動（`openclaw browser start --headless`），並深化 browser doctor 探針能力。（[來源](https://github.com/openclaw/openclaw/releases)）

4. **Plugin 冷啟動優化**：Plugin 啟動路徑移至冷儲存持久化 registry，大幅減少廣域 manifest 掃描，使 plugin 更新、修復、provider 探索更具確定性。（[來源](https://releasebot.io/updates/openclaw)）

5. **GPT-5.5 支援需求湧現**：OpenAI 於 4 月 23 日發布 GPT-5.5，OpenClaw 尚未將其納入 model catalog，導致使用 `openai-codex/gpt-5.5` 配置的用戶在 gateway warmup 時出現「Unknown model」錯誤，相關 Issue #70834、#70854 已開啟追蹤。（[來源](https://github.com/openclaw/openclaw/issues/70834)）

6. **升版 scope 權限 Bug**：從 2026.4.15 升至 2026.4.21 後，`paired.json` 裝置 scope 被重置為僅 `['operator.read']`，導致 spawn subagent 與 native approval 操作出現「scope upgrade pending approval」錯誤，最新版本已部分修復。（[來源](https://github.com/openclaw/openclaw/issues/70687)）

7. **社群規模突破新高**：Discord「Friends of the Crustacean 🦞🤝」成員突破 18 萬；Reddit r/openclaw 訂閱數達 45 萬，社群持續高速成長。（[來源](https://open-claw.social/open-claw-community.html)）

8. **Kimi K2.5 登上社群 LLM 排行榜首**：截至 4 月 27 日，依 pricepertoken.com 社群票選，Kimi K2.5 超越 Claude Opus 4.6 成為 OpenClaw 用戶評選最佳模型，GLM 4.7 緊隨其後。（[來源](https://pricepertoken.com/leaderboards/openclaw)）

9. **Dreaming 功能引發討論**：v2026.4.9 推出的「Dreaming」功能持續在社群引發架構討論，關注其在長時間 agent 工作流中的背景記憶整合行為。（[來源](https://phemex.com/news/article/openclaw-unveils-version-202649-with-new-dreaming-feature-71941)）

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **Anthropic 封鎖訂閱方案存取**：Anthropic 已封鎖 Claude 訂閱方案（Pro/Max）對 OpenClaw 的使用，目前只能透過 API Key 存取。Tier 1 需帳戶累計儲值至少 $5 USD（自 2026 年 2 月起生效）。（[來源](https://www.getaiperks.com/en/blogs/18-best-ai-model-openclaw)）
- **Opus 4.6 — 最高品質但昂貴**：長上下文處理與 prompt injection 防護最佳，適合高安全需求場景，但費用顯著高於 Sonnet。（[來源](https://haimaker.ai/blog/best-models-for-clawdbot/)）
- **Sonnet 4.5 — 最佳 CP 值**：電子郵件自動化、行事曆管理、網頁瀏覽等日常任務的推薦首選，費用約為 Opus 的幾分之一。（[來源](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)）
- **OAuth rate limit 誤判**：多名使用者反映搭配 Claude OAuth 時持續出現「API rate limit reached」假警報，實際並未超出配額，為已知 Bug（Issue #32828）。（[來源](https://www.answeroverflow.com/m/1478788645472436264)）

### GPT（OpenAI）

- **GPT-5.5 尚未納入 catalog**：GPT-5.5 於 4 月 23 日發布，透過 Codex OAuth 流程對 Plus/Pro/Business/Enterprise 訂閱者開放，但 OpenClaw v2026.4.22 起的 model catalog 尚未收錄，導致設定失敗。（[來源](https://github.com/openclaw/openclaw/issues/70854)）
- **GPT-5.4 懶惰問題**：用戶反映 GPT-5.4 在多輪 agent 任務中有「幾次嘗試後直接放棄」的行為，穩定性不如 Claude。（[來源](https://haimaker.ai/blog/best-models-for-clawdbot/)）

### DeepSeek

- **V4 成本導向首選**：DeepSeek V4 在成本敏感場景表現優異，被列為 2026 年推薦 model stack 的低成本主力。（[來源](https://skywork.ai/skypage/en/openclaw-deepseek-configuration/2048618380404060161)）
- **官方文件已完整支援**：OpenClaw 官方文件提供完整的 DeepSeek provider 設定說明。（[來源](https://docs.openclaw.ai/providers/deepseek)）

### OpenRouter

- **ClawRouter 新專案**：BlockRunAI 開源 ClawRouter，為 OpenClaw 專屬 LLM 路由器，支援 41+ 模型、<1ms 路由延遲，並支援 Base 與 Solana 鏈上 USDC 付款（x402 協定）。（[來源](https://github.com/BlockRunAI/ClawRouter)）
- **動態路由策略**：建議將簡單背景任務路由至 Gemini 2.5 Flash-Lite（$0.50/M tokens），重量級推理保留給頂級模型，以最佳化整體成本。（[來源](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)）

### 通用踩坑紀錄

| 問題類型 | 描述 | 建議解法 |
|---------|------|---------|
| 假 rate limit | 雙 provider 同時誤進 cooldown，即使未觸及配額 | `openclaw doctor --fix` |
| gateway token 失效 | Auth 過期或 scope 不足 | `openclaw models auth setup-token` |
| 升版 scope 重置 | 2026.4.15→2026.4.21 導致 paired.json scope 清空 | 升至最新版或手動重新 pair |
| GPT-5.5 Unknown model | catalog 尚未收錄 | 暫時回退 GPT-5.4 或等待修復 |

---

## 3. 社群反應與討論亮點

### Discord（18 萬成員）

- 社群重複討論三大痛點：**安全模型尚不成熟**（不適合個人以外的使用情境）、**升版偶爾無預警破壞既有 skills**、**Windows 支援明顯落後 macOS/Linux**。
- 用戶積極分享個性化 `SOUL.md` 設定檔，相關討論亦蔓延至 r/MachineLearning 與 r/selfhosted。

### Reddit r/openclaw（45 萬成員）

- GPT-5.5 Codex OAuth 整合問題引發多篇討論串，社群用戶紛紛分享臨時繞過方案。（[來源](https://www.answeroverflow.com/m/1497481318446137396)）
- 關於「OpenClaw 在升版後變得無用」的帖子獲得高度關注（GitHub Discussions #188842），版本遷移文件不足為主要抱怨點。（[來源](https://github.com/orgs/community/discussions/188842)）

### 部落格與媒體

- Medium 用戶 Mohit Aggarwal 發表「將 OpenClaw 接上 Claude、ChatGPT 和 Telegram」實戰心得，詳述真正能跑通的設定組合，引發廣泛轉發。（[來源](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)）
- Kimi K2.5 登頂 pricepertoken.com OpenClaw 排行榜，在社群引發「中系模型崛起」討論。（[來源](https://pricepertoken.com/leaderboards/openclaw)）
- DataCamp 發表「2026 年 9 個 OpenClaw 實作專案」，涵蓋 Reddit bot 到自癒伺服器，吸引新手開發者關注。（[來源](https://www.datacamp.com/blog/openclaw-projects)）

---

## 4. 值得追蹤的後續議題

1. **GPT-5.5 Catalog 支援進度**（Issue #70834、#70854）：OpenAI Codex 的 GPT-5.5 整合何時正式進入 OpenClaw model catalog，預計近期版本更新中解決。

2. **Scope 升版 Bug 完整修復確認**：2026.4.21 引入的 scope 重置問題，後續版本聲稱已修復，但社群仍有回報案例，需持續觀察。

3. **Windows 支援改善**：社群三大痛點之一，目前無具體 roadmap，但 issue 持續累積，值得關注官方回應。

4. **Dreaming 功能架構成熟度**：v2026.4.9 引入後持續在社群引發討論，特別是長時間 agent 記憶整合的穩定性與可預測性，後續版本迭代值得追蹤。

5. **ClawRouter 生態發展**：基於區塊鏈付款的 LLM 路由器是否能在社群取得採用，以及 BlockRunAI 後續的模型接入計畫。

6. **安全模型強化**：社群明確指出現行安全模型僅適合個人使用，企業場景尚缺乏成熟方案，官方是否有相關計畫值得持續關注。

---

*報告產生時間：2026-04-28 | 資料來源：GitHub、Reddit、Discord、Medium、各技術部落格*
