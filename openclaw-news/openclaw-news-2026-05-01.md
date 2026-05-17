# OpenClaw 每日情報 2026-05-01

> 資料蒐集範圍：2026-04-30 ～ 2026-05-01（過去 24 小時）

---

## 1. 今日重點摘要

1. **版本 2026.4.27 正式發布**：約 10 小時前推出，帶來 Codex Computer Use 整合、DeepInfra 新供應商、Tencent Yuanbao / QQBot 頻道擴充，以及 manifest-first 插件與模型目錄架構調整。
2. **TTS 全面升級**：新增 `/tts latest` 指令、對話範圍自動 TTS 開關、Persona 設定，並覆蓋 Azure Speech、Xiaomi、Local CLI、Inworld、Volcengine、ElevenLabs v3 等供應商。
3. **Memory 基礎設施化**：Dreaming 記憶儲存改為獨立檔案（不再膨脹每日記憶筆記）、`memory_get` 讀取邊界收緊、啟動與技能的 prompt 預算削減。
4. **Model Auth 狀態卡（Beta）**：新增 Token 健康度與 Rate-limit 壓力的視覺化狀態卡，便於監控各 LLM provider 憑證狀態。
5. **2026.4.26 升級引發大規模崩潰**：大量用戶更新後出現 Gateway 啟動循環崩潰、高 CPU 使用率、高延遲等問題；Discord、通訊類頻道尤為嚴重，4.24、4.25 也有類似紀錄。
6. **4 月 30 日 GitHub Issues 爆發**：單日新增逾 10 筆 Issue（#75300、#75298、#75297、#75290、#75289、#75283 等），多數標記為 Bug 或崩潰回歸問題。
7. **社群 LLM 排名更新**：截至 4 月 30 日社群票選最佳模型為 Kimi K2.5（第 1）、GLM 4.7（第 2）、Claude Opus 4.6（第 3）；多模型路由策略（60-70% 低成本 / 20-30% 均衡 / 10-15% 旗艦）逐漸成為主流。
8. **安全警報持續發酵**：研究人員發現 135,000+ 台 OpenClaw 實例暴露於公網，逾 50,000 台直接存在 RCE 漏洞；ClawHub 技能市集約 12%（341/2,857）技能被確認為惡意程式。
9. **ClawRouter 開源**：BlockRunAI 推出 ClawRouter，支援 41+ 模型、<1ms 路由，整合 Base / Solana x402 USDC 付款，針對 OpenClaw agent 工作流設計的 LLM 路由器。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **Auth 格式確認**：API Key 需以 `sk-ant-api03-` 開頭，可用 `openclaw models status` 驗證有效性；Gateway Token 需透過 `openclaw models auth setup-token` 重新產生。
- **Rate Limit 踩坑**：429 錯誤同時受「每分鐘請求數」、「輸入 token 數/分鐘」、「輸出 token 數/分鐘」三個維度限制，任一觸碰均觸發 429；已知 Bug（Issue #32828）：OpenClaw 在未實際達到限制時誤將雙供應商同時設為冷卻。
- **來源**：[Fix Every OpenClaw Error: API Key, Rate Limits, Gateway Token — LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error) ／ [Issue #32828 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/32828)

### GPT（OpenAI）/ Gemini（Google）

- 用戶實測成功將 Claude、ChatGPT、Telegram Bot 串接並觸發 Claude Code 自動重新部署 Vercel 服務；Gemini 亦在支援清單中。
- 建議初學者以 Claude Sonnet 或 GPT-4o 作為主模型起點。
- **來源**：[I Wired OpenClaw to Claude, ChatGPT & Telegram — Medium](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)

### DeepSeek

- 官方文件直接提供 DeepSeek 供應商設定頁；社群建議用於一般推理任務，性價比高。
- 進階用戶採用 OpenRouter `openrouter/<author>/<slug>` 格式統一管理多家供應商（含 DeepSeek），省去個別 API Key 管理。
- **來源**：[DeepSeek - OpenClaw docs](https://docs.openclaw.ai/providers/deepseek) ／ [OpenRouter Integration Guide](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)

### Mistral / Devstral

- 社群評測：Devstral 2512 在 agent 迴圈與程式碼生成工作流中表現突出，推薦作為 coding 主力模型。
- **來源**：[Best Mistral Models for OpenClaw (2026) — haimaker.ai](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

### DeepInfra（新增）

- 2026.4.27 正式納入預設供應商清單，支援模型探索、媒體生成/編輯、TTS、Embeddings，並附帶供應商自有上線政策。
- **來源**：[OpenClaw 2026.4.27 Release Notes — GitHub](https://github.com/openclaw/openclaw/releases/tag/v2026.4.27)

### OAuth Token / 假性冷卻 Bug

- Issue #23996：硬編碼 `rate_limit` 原因在 OAuth Token Auth 情境下觸發無限冷卻，即便 API 完全正常。PR #74433 修正 `doctor` 指令，當 `OPENCLAW_GATEWAY_TOKEN` 環境變數覆蓋 `gateway.auth.token` 設定時發出警告。
- **來源**：[Issue #23996 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/23996) ／ [PR #74433 · openclaw/openclaw](https://github.com/openclaw/openclaw/pulls)

---

## 3. 社群反應與討論亮點

### 升級崩潰引發遷移潮

[piunikaweb 報導](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)指出，2026.4.26 更新後 Gateway 崩潰問題廣泛出現，社群反彈強烈。部分長期用戶宣布切換至 **Hermes Agent**，稱其更新較為穩定、Bug 較少。Friends of the Crustacean 🦞 Discord（174,927 成員）成為集中討論此問題的主要場域。

### X.com 官方公告

OpenClaw 官方 X 帳號（[@openclaw](https://x.com/openclaw)）發文宣傳 2026.4.26 亮點：「Google Live Talk 優化、更佳 Ollama/local model 支援、Claude + Hermes 設定遷移、One-command Matrix E2EE」，被社群稱為「大版本」，但隨即引發崩潰浪潮。

### ClawHub 安全疑慮

安全審計發現 ClawHub 技能市集中約 12% 技能為惡意程式，社群開始呼籲加強審核機制並建議只安裝來源可信的技能。

### localModelLean 模式

新增 `localModelLean` 模式：自動移除較重的預設工具，適用於 Raspberry Pi 等弱硬體上的本地模型部署，收到 self-hosted 社群正面回應。

### ClawRouter 引起關注

[BlockRunAI/ClawRouter](https://github.com/BlockRunAI/ClawRouter) 在社群引發討論，以 <1ms 路由延遲與 Web3 原生付款整合（USDC on Base/Solana）為賣點，定位為 agent-native LLM 路由器。

---

## 4. 值得追蹤的後續議題

1. **2026.4.26/4.27 穩定性**：Gateway 崩潰問題是否在 4.27 中得到根本修復，需觀察社群回報；若持續出現，官方是否會發布緊急 patch（4.28）。
2. **Hermes Agent 崛起**：部分用戶遷移至 Hermes Agent，需持續關注 Hermes 的功能追趕速度及 OpenClaw 的回應策略。
3. **ClawHub 安全機制**：官方是否會對技能市集引入更嚴格的審核流程或程式碼掃描，防止惡意技能進一步擴散。
4. **RCE 漏洞修補進度**：135,000+ 暴露實例的 RCE 漏洞官方修補狀況，以及是否發布安全通報或強制更新。
5. **Kimi K2.5 / GLM 4.7 整合體驗**：社群票選排名前兩的新興模型，官方文件與穩定性支援尚待完善，實際踩坑紀錄值得持續蒐集。
6. **DeepInfra 供應商成熟度**：剛納入預設清單，媒體生成/TTS 等新功能的實際穩定性與費用結構需後續驗證。
7. **Model Auth 狀態卡**：Beta 功能反應如何，是否有助於減少用戶的 rate limit / auth 失效困擾，可追蹤後續 Issue 回報。

---

*報告產生時間：2026-05-01 | 資料來源：GitHub、Reddit、Discord、Medium、X.com 及各技術部落格*
