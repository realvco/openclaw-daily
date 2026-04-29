# OpenClaw 每日情報 2026-04-29

> 資料涵蓋時段：2026-04-28 ～ 2026-04-29（過去 24 小時）
> 整理語言：繁體中文

---

## 1. 今日重點摘要

1. **OpenClaw v2026.4.26 正式發布**：本次更新帶來 Google Live 瀏覽器 Talk 支援（含受約束的短命 token 與 Gateway relay）、Cerebras 全新 bundled plugin（含 onboarding 流程與靜態 model catalog）、Control UI 強化，以及 gateway 啟動、memory、cron 等多處可靠性修正。

2. **v2026.4.26 重大 Breaking Change：`coding-agent` 預設使用 `openai/gpt-5.5`**：新版本靜默內建 `coding-agent` skill 與 `codex` provider，並以 `openai/gpt-5.5` 為預設 model。未設定 OpenAI API key 的用戶升級後立即報錯，社群湧現大量 issue 反映（Issue #73358、#73607）。建議三種修復路徑：(a) 以 OpenAI auth 是否存在作為 `coding-agent` 啟用門檻、(b) 將 Codex 改為路由選項之一而非預設、(c) 升級時要求明確 opt-in。

3. **Anthropic 禁止訂閱層 OAuth token 用於第三方工具**：自 2026-04-04 起，Claude Pro/Max 訂閱方案不再涵蓋 OpenClaw 等第三方工具的使用。用戶必須改用 API key 模式（計費：input $5 / output $25 per million tokens）。

4. **社群最愛 LLM 排行（2026-04-28 投票）**：依 pricepertoken.com 社群票選，當前最受 OpenClaw 開發者歡迎的 model 為 **Kimi K2.5**，其次為 **GLM 4.7**，第三為 **Claude Opus 4.6**。多數新手仍以 Claude Sonnet 4.6 或 GPT-4o 作為入門首選。

5. **安全漏洞 CVE-2026-25253 波及 135,000+ 暴露實例**：安全研究人員揭露此高嚴重性漏洞，同時發現 ClawHub 社群 skill 中約 12% 含惡意代碼，對自架 OpenClaw 用戶構成重大風險。

6. **Google provider 全域 cooldown bug（Issue #5744）**：已確認 bug：當單一 Google model 觸及 per-model TPM rate limit 時，整個 `google` provider 進入 cooldown，導致其他仍有配額的 Google model 無法使用，影響 Gemini 多模型並行場景。

7. **v2026.4.25 TTS 全面升級**：語音回覆新增 `/tts latest`、chat 作用域的 auto-TTS 控制、persona 設定，並新增支援 Azure Speech、Xiaomi、Local CLI、Inworld、Volcengine、ElevenLabs v3 等多個 provider。

8. **OpenTelemetry 監控覆蓋大幅擴展**：model 呼叫、token 用量、tool loop、harness run、exec process、outbound delivery、context assembly、memory pressure 等全數納入 OTel tracing，方便企業級觀測。

9. **OpenClaw GitHub Stars 突破 250,829**：截至 2026 年 3 月，OpenClaw 已成為 GitHub 上非聚合器軟體類最高星數專案，社群活躍度持續攀升，目前有 3,345 個 open PR。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）
- **訂閱層封鎖**：Anthropic 官方宣布禁止 Claude Pro/Max 訂閱 OAuth token 用於 OpenClaw，僅允許 API key 方式存取。未切換的用戶會在 OpenClaw 內遭遇「LLM request rejected」或 auth 失效錯誤。
  - 來源：[Fix OpenClaw Error LLM request rejected](https://help.apiyi.com/en/openclaw-claude-llm-request-rejected-extra-usage-fix-en.html)
- **False rate limit cooldown（Issue #23996）**：provider cooldown reason 硬編碼為 `"rate_limit"`，導致 timeout、billing error、auth error 都被誤判為 rate limit，觸發無限 cooldown，需手動重啟 agent。
  - 來源：[GitHub Issue #23996](https://github.com/openclaw/openclaw/issues/23996)

### OpenAI（GPT-5.5 / Codex）
- **v2026.4.26 silent default 破壞升級**：`coding-agent` skill 偷渡 `openai/gpt-5.5` 為預設，未設定 OpenAI key 的用戶升級後立刻遭遇 `"No API key found for provider 'openai'"` 錯誤。
  - 來源：[GitHub Issue #73358](https://github.com/openclaw/openclaw/issues/73358)
- **GPT-5.5 thinking level 回退（Issue #71269）**：v2026.4.23 起，`openai-codex/gpt-5.5` 和 `gpt-5.4-mini` 不再廣告 `xhigh` thinking level，與 v2026.4.22 及官方文件相悖（regression）。
  - 來源：[GitHub Issue #71269](https://github.com/openclaw/openclaw/issues/71269)

### Google（Gemini）
- **Per-model rate limit 誤殺整個 provider**：某 Gemini model 達到 TPM 上限後，整個 `google` provider 被封鎖，即便其他模型仍有配額也無法切換，影響多 Gemini model 並行部署。
  - 來源：[GitHub Issue #5744](https://github.com/openclaw/openclaw/issues/5744)

### DeepSeek
- **費用最佳化實戰**：有用戶將 OpenClaw 後端從 Claude 切換至 DeepSeek V3.2，在 20 天內將 API 費用從 $420 降至 $168（節省 60%）。適合郵件分類、日曆管理等非 critical 場景。
  - 來源：[Best free AI models for OpenClaw - LumaDock](https://lumadock.com/tutorials/free-ai-models-openclaw)

### OpenRouter
- **統一入口存取 300+ 模型**：OpenClaw 原生整合 OpenRouter，只需設定 API key，以 `openrouter/<author>/<slug>` 格式引用 model，無需額外設定 `models.providers`。支援熱重載，可在不重啟 agent 的情況下即時切換 Claude → GPT → Gemini → DeepSeek。
  - 來源：[OpenRouter Integration Guide](https://openrouter.ai/docs/guides/guides/openclaw-integration)

### Kimi（Moonshot）與 GLM
- **社群票選雙冠**：Kimi K2.5 與 GLM 4.7 在 2026-04-28 的 pricepertoken.com 社群票選中分列一、二，顯示中系 LLM 在 OpenClaw 社群的接受度快速提升。
  - 來源：[Best Model for OpenClaw — LLM Rankings (2026)](https://pricepertoken.com/leaderboards/openclaw)

---

## 3. 社群反應與討論亮點

### Discord「Friends of the Crustacean」（174,596 成員）
- 三大反覆出現的批評：**安全模型不成熟**（不適合個人以外的使用場景）、**版本升級路徑偶爾無預警破壞 skill**、**Windows 支援明顯落後**。
- 來源：[Discord - OpenClaw](https://docs.openclaw.ai/channels/discord)

### Reddit（r/MachineLearning、r/selfhosted）
- 開發者熱衷分享個人化的 `SOUL.md` 設定（人格/行為配置檔），方式類似上一個世代分享 dotfiles，顯示社群高度認同對 agent 個性的客製化。

### Hacker News
- 討論集中在**實作經驗分享**而非理論探討，表明大量用戶已在生產環境中實際運行 OpenClaw。

### GitHub 社群討論
- **Issue #73358** 引發大量用戶抱怨 v2026.4.26 的 silent breaking change，要求官方在 release notes 明確標示 breaking changes，並提供自動 migration 機制。
- **「Openclaw useless now after update」**（Discussion #188842）：部分長期用戶在 Anthropic 封鎖訂閱層後表示工具失去原有吸引力，正在評估切換至其他 provider。
  - 來源：[GitHub Discussions #188842](https://github.com/orgs/community/discussions/188842)

---

## 4. 值得追蹤的後續議題

1. **v2026.4.26 `coding-agent` breaking change 官方修復進度**：Issue #73358 / #73607 尚在討論中，需觀察官方是否在下一個 patch 中提供自動遷移或明確 opt-in 機制。

2. **CVE-2026-25253 修補狀況**：安全漏洞影響 135,000+ 暴露實例，需追蹤官方安全公告與 patch 釋出時程，以及 ClawHub 惡意 skill 的掃除進度（ClawSweeper bot 的處理效率）。

3. **Anthropic API 費用政策走向**：訂閱層封鎖後，Claude 使用成本大幅上升，需關注 Anthropic 是否推出針對第三方工具的企業方案，或 OpenClaw 官方是否協商取得特殊授權。

4. **Google provider cooldown bug 修復（Issue #5744）**：影響 Gemini 多模型並行場景，若官方未在近期版本修復，建議改用 OpenRouter 作為 Google model 的統一入口以繞過此問題。

5. **GPT-5.5 `xhigh` thinking level regression（Issue #71269）**：影響需要深度推理的 coding agent 工作流，需追蹤 OpenAI 或 OpenClaw 端的修復進展。

6. **Kimi K2.5 / GLM 4.7 深度評測**：中系 LLM 在社群票選中異軍突起，值得深入測試其在 OpenClaw agent 工作流中的實際效能（特別是 tool use、multi-step 推理與中文語境）。

7. **OpenTelemetry 整合的企業落地案例**：v2026.4.25 的 OTel 擴展為企業部署帶來可觀測性，值得追蹤社群是否產出可參考的 Grafana / Datadog dashboard 範本。

---

## 來源索引

- [OpenClaw 官方 GitHub Releases](https://github.com/openclaw/openclaw/releases)
- [Release v2026.4.26](https://github.com/openclaw/openclaw/releases/tag/v2026.4.26)
- [GitHub Issue #73358 — coding-agent silent default](https://github.com/openclaw/openclaw/issues/73358)
- [GitHub Issue #73607 — agents.defaults.params stripped](https://github.com/openclaw/openclaw/issues/73607)
- [GitHub Issue #71269 — gpt-5.5 xhigh regression](https://github.com/openclaw/openclaw/issues/71269)
- [GitHub Issue #5744 — Google provider cooldown bug](https://github.com/openclaw/openclaw/issues/5744)
- [GitHub Issue #23996 — False provider cooldown](https://github.com/openclaw/openclaw/issues/23996)
- [GitHub Discussions #188842 — Openclaw useless after update](https://github.com/orgs/community/discussions/188842)
- [Best Model for OpenClaw — LLM Rankings (pricepertoken.com)](https://pricepertoken.com/leaderboards/openclaw)
- [OpenRouter Integration Guide](https://openrouter.ai/docs/guides/guides/openclaw-integration)
- [Fix OpenClaw rate limit exceeded (429)](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)
- [Fix OpenClaw LLM request rejected](https://help.apiyi.com/en/openclaw-claude-llm-request-rejected-extra-usage-fix-en.html)
- [OpenClaw Discord](https://docs.openclaw.ai/channels/discord)
- [LumaDock — Free AI models for OpenClaw](https://lumadock.com/tutorials/free-ai-models-openclaw)
- [Medium — I Wired OpenClaw to Claude, ChatGPT & Telegram](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)
- [OpenClaw 2026.4.1 Deep Dive (VibeSparking)](https://www.vibesparking.com/en/blog/tools/openclaw/2026-04-02-openclaw-2026-4-1-stable-analysis/)
- [Releasebot — OpenClaw April 2026 Release Notes](https://releasebot.io/updates/openclaw)
