# OpenClaw 每日情報 2026-04-24

> 搜尋範圍：過去 24 小時（2026-04-23 ～ 2026-04-24）

---

## 1. 今日重點摘要

1. **v2026.4.22 正式發布**：新增 xAI 完整多媒體支援，包含圖像生成（grok-imagine-image / grok-imagine-image-pro）、文字轉語音（6 種 xAI 語音）、語音識別（grok-stt），以及即時語音通話串流轉錄，為 Grok 系列整合帶來重大躍升。

2. **Pi/模型目錄更新至 0.68.1**：OpenCode Go 目錄改由 pi 統一管理，新增 opencode-go/kimi-k2.6、Qwen、GLM、MiMo、MiniMax 等模型條目，不再依賴插件維護的別名。

3. **Kimi K2.6 多輪工具呼叫修復**：v2026.4.22 修正 Moonshot Kimi 在 OpenAI 相容傳輸層上，native `tool_call` ID 被過度消毒導致多輪 agentic 流程在 2–3 輪後中斷的問題，並新增 `sanitizeToolCallIds` opt-out 選項。

4. **OpenAI Codex OAuth 安全改善**：移除從 `~/.codex` 複製 OAuth 憑證至 agent auth store 的機制，改用瀏覽器登入或裝置配對，並修正舊版 per-agent OAuth profile 遮蔽新身份的問題。

5. **Tokenjuice 整合**：新增 tokenjuice 作為 opt-in 插件，自動壓縮 Pi embedded 執行中的 exec/bash 工具輸出結果，有效降低 token 消耗。

6. **Linux 子行程 OOM 分數調整**：Gateway 管理的子行程（supervisor、PTY、MCP stdio、瀏覽器）現在在 Linux 上透過 `/bin/sh` shim 提高自身 `oom_score_adj`，確保在 cgroup 記憶體壓力下核心優先終止暫時性工作者而非長期 gateway（可用 `OPENCLAW_CHILD_OOM_SCORE_ADJ=0` 停用）。

7. **Active Memory 插件 120 秒逾時 Bug（Issue #68825）**：從 v2026.3.24 升級至 v2026.4.15 後，啟用 Active Memory 插件的 blocking sub-agent 呼叫會一律逾時，gateway log 同步出現 `[memory] qmd update failed` 錯誤，目前尚未有官方修復版本。

8. **社群模型票選排行**（截至 2026-04-23）：Kimi K2.5 居首，GLM 4.7 第二，Claude Opus 4.6 第三，反映出開源模型的強勁表現。

9. **GitHub 活躍度極高**：2026-04-23 單日新增 Issues 逾 10 筆（含 bug、enhancement、regression），PR 包含 OpenClaw Stabilization & Production Hardening（#70206）等重要合併請求，開發節奏維持每日多次提交。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **OAuth 假性 rate limit 問題**（持續回報中）  
  使用 claude OAuth（`sk-ant-oat01-` 前綴）的用戶普遍回報收到「API rate limit reached. Please try again later.」，但直接呼叫 Anthropic API 正常。根源在於 OpenClaw 錯誤地對整個 Anthropic provider 觸發 cooldown，連帶影響所有 Claude 模型（Opus / Sonnet / Haiku）。
  - 來源：[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828)
  - 來源：[AnswerOverflow 討論](https://www.answeroverflow.com/m/1478788645472436264)

- **Auth 格式容易搞錯**：OAuth token 必須使用 `{ "type": "token", ... }` 格式，若誤用 `"type": "api_key"` + `"key"` 欄位將導致 401 錯誤。

- **Cooldown 層級問題**：cooldown 套用在 provider 層（非 model 層），Claude Sonnet 遇到 429 時，Claude Opus 也會被凍結，即便兩者有獨立的 rate limit 配額。

### Kimi K2.6（Moonshot AI）

- **2026-04-20 發布**：1T 總參數、MoE 架構 32B active tokens，SWE-Bench Pro 58.6%，超越 GPT-5.4 的 57.7%，成為目前開源最強代碼模型。
- **OpenClaw 社群評價高**：在持續 24/7 agentic 工作流（多應用跨平台協作）場景中表現突出，Moonshot 內部以 K2.6-backed agent 自主運行 5 天執行監控與事件響應。
- **多輪工具呼叫 Bug 已修**：v2026.4.22 修復 Kimi 在 OpenClaw 中 2–3 輪後工具呼叫中斷的問題。
- 來源：[Trilogy AI Substack](https://trilogyai.substack.com/p/kimi-k26-is-the-open-model-release)
- 來源：[Kimi K2.6 Tech Blog](https://www.kimi.com/blog/kimi-k2-6)

### DeepSeek

- 對於結構性確定任務（批量資料處理、模式比對、簡單流程自動化）表現尚可。
- 在需要適應性推理或複雜多步驟任務管理的場景中，工具使用可靠性不穩定，**不建議作為主要 agent 驅動模型**。

### Gemini（Google）

- Gemini 3 Pro 以 100 萬 token 上下文視窗成為長文件處理首選。
- Gemini 免費方案（60 req/min、1000 req/day）成為 2026 「免費方案之王」，對預算有限的開發者具吸引力。
- 來源：[OpenClaw Playbook - Gemini 完整設定指南](https://www.openclawplaybook.ai/guides/how-to-use-openclaw-with-gemini/)

### OpenRouter

- 作為基礎設施層：單一 API 金鑰即可接入 Claude Opus 4.6、GPT-5.4、Grok、Kimi、Qwen 等全系模型，免去管理多個帳號和金鑰的麻煩，適合同時評測多模型的場景。
- 來源：[BetterClaw - 模型比較實際費用](https://www.betterclaw.io/blog/openclaw-model-comparison)

### 已知 Rate Limit 問題彙整

| 問題 | 狀態 | 連結 |
|------|------|------|
| 假性 rate limit（所有模型同時觸發 cooldown） | 開啟 | [#32828](https://github.com/openclaw/openclaw/issues/32828) |
| OAuth token auth 觸發假 cooldown | 開啟 | [#23996](https://github.com/openclaw/openclaw/issues/23996) |
| 429 無指數退避（exponential backoff） | 已關（not planned） | [#5159](https://github.com/openclaw/openclaw/issues/5159) |

---

## 3. 社群反應與討論亮點

- **Kimi K2.6 是 OpenClaw 用戶最期待的開源模型發布**：Trilogy AI 部落格標題直接以此為題，社群對 K2.6 在 agentic 工作流中的表現給予高度評價，認為其工具使用能力達到 Claude Opus 4.6 水準。

- **Active Memory 插件升溫**：自 2026.4.10 引入後持續成為熱門討論主題。「讓 AI 在回答前主動調取記憶」的設計理念受到社群認可，但 120 秒逾時 bug 讓部分用戶在升級至 4.15 後暫緩採用。

- **OpenClaw 社群 Discord 已達 173,236 人**：服務涵蓋 WhatsApp、Telegram、Discord、Slack 等 15+ 通訊平台，4 個管理團隊各司其職，並同時維護 r/OpenClaw 和 r/Clawdbot 子版。

- **「OpenClaw 更新後變廢了」引發討論**：GitHub Discussion #188842「Openclaw useless now after update」在社群引起共鳴，顯示部分用戶對近期更新的相容性有疑慮，具體問題集中在 Active Memory 逾時與 rate limit 誤報。

- **xAI/Grok 整合受到關注**：v2026.4.22 新增 grok-imagine-image 和 TTS 支援，是繼 Claude 和 OpenAI 後，xAI 獲得最完整 OpenClaw 多媒體整合的里程碑，社群對此表示期待。

- **模型費用實測文章增加**：多個第三方部落格（BetterClaw、PricePerToken、SimilarLabs）發布實際任務費用比較，成為社群決策參考依據。

---

## 4. 值得追蹤的後續議題

1. **Active Memory 120 秒逾時修復進度**（Issue #68825）  
   影響從 v2026.3.24+ 升級的用戶，需追蹤官方修復版本發布時間。

2. **Claude OAuth 假性 rate limit 根本解決**（Issue #32828）  
   目前僅能靠設定 fallback model 繞過，期待官方在 provider cooldown 邏輯層面修正。

3. **Kimi K2.6 在 OpenClaw 長時間 agentic 流程的穩定性**  
   K2.6 工具呼叫 bug 雖已修復，但在實際 24/7 多平台場景的長期表現仍需社群累積回報。

4. **v2026.4.20-beta.1 → v2026.4.22 的 beta 功能穩定化**  
   兩個 beta 版本在兩天內轉為正式版，需觀察是否有遺漏的 regression。

5. **OpenClaw Stabilization & Production Hardening（PR #70206）合併後影響**  
   此 PR 涉及生產環境穩定性重構，後續觀察對 agentic 工作流的實際影響。

6. **Tokenjuice 插件的社群採用與效益回報**  
   新 opt-in 插件承諾降低 Pi embedded 執行的 token 消耗，需累積實際數據驗證效益。

7. **DeepSeek v4 發布時機**  
   [AINews 報導](https://www.latent.space/p/ainews-moonshot-kimi-k26-the-worlds)指出 K2.6 在 DeepSeek v4 發布前搶先刷新排行，後者一旦發布將對 OpenClaw 模型選擇生態造成重大衝擊。

---

## 來源參考

- [GitHub Releases · openclaw/openclaw](https://github.com/openclaw/openclaw/releases)
- [OpenClaw v2026.4.22 on NewReleases.io](https://newreleases.io/project/github/openclaw/openclaw/release/v2026.4.22)
- [OpenClaw Release Notes - Releasebot](https://releasebot.io/updates/openclaw)
- [Best Models for OpenClaw (April 2026) - haimaker.ai](https://haimaker.ai/blog/best-models-for-clawdbot/)
- [Best Model for OpenClaw — Community Votes - PricePerToken](https://pricepertoken.com/leaderboards/openclaw)
- [OpenClaw Model Comparison - BetterClaw](https://www.betterclaw.io/blog/openclaw-model-comparison)
- [Kimi K2.6 Is the Open Model OpenClaw Users Were Waiting For - Trilogy AI](https://trilogyai.substack.com/p/kimi-k26-is-the-open-model-release)
- [Kimi K2.6 Tech Blog - Moonshot AI](https://www.kimi.com/blog/kimi-k2-6)
- [GitHub Issue #32828 - False API rate limit](https://github.com/openclaw/openclaw/issues/32828)
- [GitHub Issue #23996 - False provider cooldown on OAuth](https://github.com/openclaw/openclaw/issues/23996)
- [GitHub Issue #68825 - Active Memory timeout](https://github.com/openclaw/openclaw/issues/68825)
- [OpenClaw 2026.4.10: Active Memory Plugin - OpenClaw Playbook](https://www.openclawplaybook.ai/blog/openclaw-2026-4-10-release-codex-active-memory/)
- [OpenClaw Rate Limit Fix Guide - LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)
- [OpenRouter - Best AI Models for Coding](https://openrouter.ai/collections/programming)
- [Model Providers - OpenClaw Docs](https://docs.openclaw.ai/concepts/model-providers)

---

*報告生成時間：2026-04-24 | 資料來源：GitHub、社群論壇、技術部落格*
