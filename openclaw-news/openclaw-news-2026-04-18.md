# OpenClaw 每日情報 2026-04-18

> 資料涵蓋範圍：2026-04-17 至 2026-04-18（過去 24 小時）

---

## 一、今日重點摘要

1. **v2026.4.14 為最新穩定版本**：著重 GPT-5 家族的顯式輪次（explicit turn）改進、頻道供應商問題修復，及底層核心程式碼效能重構。v2026.4.15-beta.1 已開放搶先測試。

2. **Anthropic 訂閱 OAuth 已正式斷線**：自 2026 年 4 月 4 日起，Claude Pro/Max 訂閱者無法再以 OAuth token 授權 OpenClaw 使用，部分使用者費用暴增高達 50 倍，須改用 pay-as-you-go API Key。

3. **OpenClaw 創辦人帳號遭 Anthropic 暫停（4/10）**：Peter Steinberger 被 Anthropic 以「可疑活動」為由暫時封鎖帳號，引發社群廣泛討論，對 Anthropic 開放性的質疑聲浪上升。

4. **社群最佳模型評選：Kimi K2.5 奪冠**：截至 2026-04-17，依社群投票 Kimi K2.5 排名第一，GLM 4.7 第二，Claude Opus 4.6 第三；多數使用者轉向中國模型以降低成本。

5. **Active Memory Plugin 正式推出**：在主回覆前執行專屬記憶子代理，自動提取偏好、上下文及過去細節，支援 message/recent/full 三種 context 模式，可用 `/verbose` 即時檢查。

6. **Model Auth 狀態卡新增**：一目了然顯示 OAuth token 健康狀態與供應商速率限制壓力，token 即將過期時主動顯示警示。

7. **ClawHub auth token 未傳遞 Bug（Issue #52949）**：Gateway 呼叫時未傳遞 auth token，導致所有 ClawHub 呼叫持續觸發 429，Available Skills 清單顯示為空。

8. **Amazon Bedrock PR 修復合入**：社群貢獻者提交 PR #67311 修復 Bedrock mantle 穩定性問題，改善 Bedrock 模型在 OpenClaw agent 工作流中的表現。

9. **本地 MLX 語音供應商（實驗性）**：Talk Mode 新增本地 MLX 語音合成選項，支援供應商顯式選擇與本地語音播放，可完全離線運作。

---

## 二、LLM 串接實戰回報

### 2.1 Anthropic Claude 訂閱政策大轉變

**來源：**
- [TechCrunch — Anthropic says Claude Code subscribers will need to pay extra for OpenClaw usage](https://techcrunch.com/2026/04/04/anthropic-says-claude-code-subscribers-will-need-to-pay-extra-for-openclaw-support/)
- [The Register — Anthropic closes door on subscription use of OpenClaw](https://www.theregister.com/2026/04/06/anthropic_closes_door_on_subscription/)
- [VentureBeat — Anthropic cuts off the ability to use Claude subscriptions with OpenClaw](https://venturebeat.com/technology/anthropic-cuts-off-the-ability-to-use-claude-subscriptions-with-openclaw-and)
- [Hacker News #47633396](https://news.ycombinator.com/item?id=47633396)
- [費用試算指南（shareuhack.com）](https://www.shareuhack.com/en/posts/openclaw-claude-code-oauth-cost)

2026 年 4 月 4 日，Anthropic 正式切斷 Claude Pro（$20/月）與 Claude Max（$200/月）訂閱者透過 OAuth token 使用 OpenClaw 的能力。官方理由為第三方工具繞過 Prompt Cache 最佳化，對運算資源造成「不成比例的壓力」。

**實際影響：**
- 使用者費用可能暴增高達 **50 倍**
- 必須改用 pay-as-you-go API Key 模式
- Peter Steinberger 與投資人 Dave Morin 嘗試談判緩衝期，最終僅爭取到多一週的延緩執行

**創辦人帳號遭封：**
- [TechCrunch（4/10）](https://techcrunch.com/2026/04/10/anthropic-temporarily-banned-openclaws-creator-from-accessing-claude/) 報導 Steinberger 帳號遭 Anthropic 暫時封鎖，進一步加劇社群對 Anthropic 開放性的疑慮

---

### 2.2 Claude OAuth Token 持續 429 問題

**來源：**
- [Discord 社群討論（AnswerOverflow）](https://www.answeroverflow.com/m/1478788645472436264)
- [GitHub Issue #52949](https://github.com/openclaw/openclaw/issues/52949)

多名使用者反映搭配 Claude OAuth 時，即使 API 本身完全正常，仍持續收到「API rate limit reached」錯誤。根本原因為 Gateway 的 `searchSkillsFromClawHub` 函式未正確傳遞 `~/.config/clawhub/config.json` 中的 ClawHub auth token，導致每次呼叫都因未授權觸發 429。

**臨時解法：**
```bash
openclaw doctor --fix
```
可自動修復大多數設定問題。

---

### 2.3 虛假 Provider Cooldown 問題

**來源：**
- [GitHub Issue #23996](https://github.com/openclaw/openclaw/issues/23996)
- [GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828)
- [LaoZhang AI Blog — 完整 Rate Limit 除錯指南](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)

Provider cooldown 的觸發原因被硬編碼為 `rate_limit`，無論實際失敗原因是 timeout、billing 異常、auth 錯誤或其他情況，一律觸發無限冷卻。此 bug 導致使用者誤以為達到速率限制，實際上可能只是其他錯誤被誤判，影響所有模型供應商。

---

### 2.4 社群模型效能排名（截至 2026-04-17）

**來源：**
- [Price Per Token — Best Model for OpenClaw](https://pricepertoken.com/leaderboards/openclaw)
- [Novita AI — Best AI Models for OpenClaw Agents](https://blogs.novita.ai/best-ai-models-for-openclaw/)
- [Apiyi Blog — 性價比模型比較：DeepSeek V3.2 vs MiniMax M2.5 vs GLM-5](https://help.apiyi.com/en/openclaw-cost-effective-models-deepseek-minimax-glm5-en.html)

| 排名 | 模型 | 核心優勢 | 輸入定價（/M） | 輸出定價（/M） |
|------|------|---------|--------------|--------------|
| 🥇 1 | **Kimi K2.5** | 工具呼叫準確率高、性價比佳 | $0.45 | $0.45 |
| 🥈 2 | **GLM 4.7 / GLM-5** | SWE-bench 第一（67.80%）、程式碼能力強 | $0.80 | $2.56 |
| 🥉 3 | **Claude Opus 4.6** | 穩定可靠、工具使用一致性高 | API 計費 | API 計費 |
| — | **DeepSeek V3.2** | 最便宜，但整體排名已下滑 | $0.27 | $0.41 |

**Medium 實測文章：** [《I Wired OpenClaw to Claude, ChatGPT & Telegram — Here's What Actually Works》](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc) 記錄了多模型串接 Telegram 的完整實戰心得，涵蓋各模型在 agent 工作流中的穩定性比較。

---

### 2.5 Amazon Bedrock 修復

**來源：** [GitHub PR #67382](https://github.com/openclaw/openclaw/pull/67382)

社群貢獻者針對 Amazon Bedrock mantle 問題提交 PR，改善 Bedrock 模型在 OpenClaw 中的穩定性。此 PR 為近期 GitHub 活動中受關注度較高的社群貢獻。

---

## 三、社群反應與討論亮點

### Anthropic vs OpenClaw 爭議持續延燒

- **Hacker News（#47633396）**：討論串湧現大量開發者不滿，認為 Anthropic 此舉為「逐利行為」，且缺乏事先充分溝通。部分社群成員開始探索轉向 OpenAI、Gemini、Kimi 等替代 provider。

- **Fusion94 Blog — [《Anthropic Just Closed the Subscription Loophole - And Why It Might Backfire》](https://fusion94.org/blog/2026-04-04-anthropic-kills-claude-subscriptions-for-third-party-tools/)**：深度分析 Anthropic 此舉可能對自身生態的反效果，認為此舉可能加速使用者轉向競爭對手模型。

- **XDA Developers — [《Claude subscribers just lost access to OpenClaw unless they pay more》](https://www.xda-developers.com/claude-subscribers-just-lost-access-to-openclaw-and-other-third-party-toolsunless-they-pay-more/)**：以消費者角度分析影響，吸引大量非技術用戶關注。

### Discord「Friends of the Crustacean 🦞🤝」動態

- 伺服器已達 **171,104 成員**，為主要用戶互助基地（[邀請連結](https://discord.com/invite/clawd)）
- `#models` 頻道持續有 Kimi K2.5 vs GLM-5 的熱烈討論，多數使用者轉向中國模型以降低成本
- `#help` 頻道中 Helper Team 積極協助大量 OAuth → API Key 遷移問題
- `#users-helping-users` 頻道出現多份自製遷移指南

### OpenClaw Newsletter（2026-04-09）

- **來源：** [Buttondown — OpenClaw Newsletter 2026-04-09](https://buttondown.com/openclaw-newsletter/archive/openclaw-newsletter-2026-04-09/)
- 本期 Newsletter 詳細回顧了 Anthropic 政策變更的時間軸，並提供 API Key 申請與設定的逐步指引。

---

## 四、值得追蹤的後續議題

1. **v2026.4.15 正式版發布時程**：Beta 版現已開放，預計近日釋出，需關注正式版功能清單與穩定性表現。

2. **Issue #52949（ClawHub token 未傳遞）修復進度**：此 Bug 影響範圍廣，是否納入 v2026.4.15 正式修復值得持續關注。

3. **Anthropic 政策後續**：Anthropic 是否會推出官方 OpenClaw 整合方案，或社群是否形成穩定的「非 Anthropic」模型生態，將決定 OpenClaw 未來走向。

4. **OpenClaw Foundation 治理機制**：Peter Steinberger 離開後，非營利基金會的決策結構與社群代表性機制尚未完全確立。

5. **Active Memory Plugin 長期穩定性**：新 Plugin 在長時間、多輪對話中的記憶準確性與效能影響，需更多社群實測數據。

6. **中國模型（Kimi / GLM）長期競爭力**：隨 GLM-5 在 SWE-bench 奪冠，以及 Kimi K2.5 在 tool calling 的優異表現，後續 benchmark 更新與實際 agent 使用數據值得持續追蹤。

---

*報告生成時間：2026-04-18 | 資料來源：GitHub、TechCrunch、The Register、VentureBeat、Hacker News、Discord、Medium、Price Per Token、Releasebot*
