# OpenClaw 每日情報 2026-05-02

> 資料蒐集範圍：2026-05-01 ～ 2026-05-02（過去 24 小時）

---

## 1. 今日重點摘要

1. **v2026.4.27 穩定性跟進**：繼 4.26 大規模崩潰事件後，4.27 著重修復 Gateway 啟動循環與 Discord 頻道斷線問題；Codex Computer Use 狀態/安裝指令及 Marketplace 探索功能正式上線，fail-closed MCP 安全檢查亦同步強化。
2. **Google Meet 插件正式捆綁**：v2026.4.27 隨附 Google Meet 參與者插件，支援個人 Google OAuth 認證、明確指定會議 URL 加入、Chrome 與 Twilio 即時傳輸，並可在語音通話中即時呼叫 Full Agent 協助。
3. **NVIDIA 與 Bedrock Opus 4.7 整合**：NVIDIA 模型目錄及上線流程已納入供應商支援；AWS Bedrock Opus 4.7 新增 xhigh / adaptive / max 三種思考模式 Profile，與本地 Opus 4.7 完全對等。
4. **假性 Rate Limit 錯誤持續困擾用戶**：GitHub Issue #32828 確認即使所有底層 API 均正常，OpenClaw 仍可能對所有模型顯示「API rate limit reached」；Issue #48603 追蹤同一問題並請求官方功能修復，Claude OAuth 搭配 OpenClaw 的 rate limit 循環錯誤亦持續有用戶回報。
5. **ClawSweeper 議題管理數據披露**：OpenClaw 自研的 ClawSweeper 維護機器人目前掃描 7,000+ 筆 Issue 與 PR，過去一週審查 3,478 筆，最終關閉率僅 0.1%（約 4 筆），顯示議題仍以保守方式管理。
6. **社群 LLM 票選排名維持不變**：截至 5 月 1 日，社群票選最佳模型仍為 Kimi K2.5（第 1）、GLM 4.7（第 2）、Claude Opus 4.6（第 3）；DeepSeek V4 Flash 成為 4 月起的預設模型，具備 100 萬 token 上下文窗口。
7. **安全警示仍在發酵**：CVE-2026-25253（CVSS 8.8）WebSocket Origin 標頭繞過 RCE 漏洞尚未獲廣泛修補；ClawHub 技能市集約 341 個（12%）技能被確認含惡意程式，官方建議謹慎安裝第三方技能。
8. **創辦人加入 OpenAI 後續**：Peter Steinberger（OpenClaw 原作者，開發時名為 Clawdbot）2 月宣佈加入 OpenAI 後，非營利基金會的籌備進度受社群持續關注，部分用戶擔憂長期維護方向。
9. **用戶出走至 Hermes Agent**：因 4.26 更新穩定性問題，部分長期用戶轉而使用 Hermes Agent，稱其更新流程更順暢、Bug 更少；OpenClaw 缺乏可靠的穩定版本頻道成為核心抱怨點。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **Claude OAuth + Rate Limit 循環**：多名用戶在 OpenClaw Discord 回報，使用 Claude OAuth 認證方式時，頻繁觸發「API rate limit reached. Please try again later.」，即便 API 本身完全正常；暫解方法為改用 API Key 方式，或執行 `openclaw models auth setup-token` 重新建立憑證。
  - 來源：[AnswerOverflow – Claude OAuth rate limit](https://www.answeroverflow.com/m/1478788645472436264)
- **Bedrock Opus 4.7 思考模式 (Thinking)**：Bedrock 上的 Opus 4.7 現已支援 xhigh、adaptive、max 三種思考模式 Profile，與直接呼叫 Anthropic API 功能對等，適合需要深度推理的 Agent 任務。
  - 來源：[OpenClaw Release Notes – Releasebot](https://releasebot.io/updates/openclaw)
- **Sonnet 4.5 仍是 CP 值首選**：社群普遍認為 Claude Sonnet 4.5 在郵件自動化、日曆管理、網頁瀏覽等標準任務中，以遠低於 Opus 的費用達到足夠效能。
  - 來源：[Best AI Model for OpenClaw 2026 – GetAIPerks](https://www.getaiperks.com/en/blogs/18-best-ai-model-openclaw)

### GPT / OpenAI

- **GPT-4.1 定位於 Tier 2 均衡層**：社群測試建議將 GPT-4.1（$2–8/M tokens）用於標準生成與程式碼任務；OpenAI 的 token bucket rate limiting 系統與 Anthropic 相同，等待 60 秒通常可恢復。
  - 來源：[Fix OpenClaw Rate Limit 429 – LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)

### DeepSeek

- **V4 Flash 成預設模型**：DeepSeek V4 Flash 自 4 月起成為 OpenClaw 預設模型，提供 1M token 上下文窗口及出色的工具呼叫能力，價格為所有主流模型中最低。OpenClaw 官方文件亦已更新 DeepSeek 整合教學。
  - 來源：[DeepSeek – OpenClaw Docs](https://docs.openclaw.ai/providers/deepseek)
- **多模型路由策略下的角色**：60–70% 的低成本任務（分類、摘要）建議路由至 DeepSeek V3 或 Llama 3.3 70B（$0.27–1.10/M tokens），可使整體成本較全程使用 Claude/GPT-4.1 降低 4–8 倍。
  - 來源：[Best AI Models for OpenClaw 2026 – GetClawHosting](https://getclawhosting.com/blog/guides/best-ai-models-for-openclaw-2026)

### Gemini

- **Gemini 2.5 Pro 定位於 Tier 3 旗艦層**：針對複雜多工具 Agent 任務，Gemini 2.5 Pro 與 Claude Sonnet 4.6 同列 Tier 3（$3–15/M tokens），適合需要長上下文推理的場景。
  - 來源：[Best AI Models for OpenClaw 2026 – GetClawHosting](https://getclawhosting.com/blog/guides/best-ai-models-for-openclaw-2026)

### Mistral

- **Mistral Large 均衡層替代方案**：Mistral Large 被列為 GPT-4.1 的同層替代選項，適合標準生成任務；社群另有 Devstral 與 Ministral 的比較文章，聚焦於 OpenClaw 程式碼任務場景。
  - 來源：[Best Mistral Models for OpenClaw – haimaker.ai](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

### OpenRouter

- **OpenRouter 整合方式確認**：OpenRouter 已獲官方文件正式支援，設定方式為填入 API Key 並以 `openrouter/<author>/<slug>` 格式引用模型；社群 Top AI Models on OpenRouter 排行榜亦持續追蹤 OpenClaw 使用者的模型偏好。
  - 來源：[OpenRouter – OpenClaw Integration Docs](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)

---

## 3. 社群反應與討論亮點

- **穩定性信任危機**：4.26 更新造成的大規模崩潰仍是社群討論主軸，用戶在 Reddit（r/openclawsetup）及官方 Discord 中強烈呼籲官方建立「穩定版本頻道」，避免重蹈覆轍。部分用戶直言「4.26 之後就再也不信任自動更新」。
  - 來源：[OpenClaw 2026.4.26 update triggering gateway crashes – Piunika Web](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)
- **Hermes Agent 競爭壓力**：有長期用戶在 Discord 發文表示已將 Hermes Agent 設為主力，保留 OpenClaw 作為備援，主因是 Hermes 更新更穩定、Bug 較少。這是社群首次出現較大規模的替代方案討論。
- **ClawHub 安全性爭論**：12% 惡意技能的數字引發社群對技能市集審核機制的廣泛批評，有用戶發起討論要求強制程式碼簽名及沙盒執行環境。
- **假性 Rate Limit Bug 怒吼**：Issue #32828 下方已累積大量用戶回報，描述為「花 1 小時以為是 API 帳號問題，結果是 OpenClaw 的 Bug」，情緒激動程度為近期 Issue 中最高之一。
  - 來源：[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828)
- **創辦人動向與專案前途**：Steinberger 加入 OpenAI 的消息持續發酵，Discord 中有用戶討論「若 OpenAI 介入，OpenClaw 是否會商業化或閉源」，官方尚未就此正式回應。
- **多模型路由成為進階用法標配**：越來越多的進階用戶分享其「三層路由」配置，將 DeepSeek/Llama 處理雜務、Claude/Gemini 處理複雜任務，成本控制效果顯著，相關教學帖在社群廣泛傳播。

---

## 4. 值得追蹤的後續議題

1. **v2026.4.27 正式穩定性評估**：4.26 崩潰事件後，4.27 的修復品質將決定社群信心能否回升；建議追蹤 GitHub Issues 是否新增與 4.27 相關的崩潰或迴歸 Bug。
2. **假性 Rate Limit Bug 修復進度**（Issue #32828、#48603）：此 Bug 影響所有 Claude OAuth 用戶，若官方長期不修復將加速用戶改用 API Key 或其他 Provider。
3. **ClawHub 安全審查機制改革**：12% 惡意技能的揭露後，官方是否會推出技能簽名、沙盒或白名單機制，值得持續觀察。
4. **非營利基金會籌備進度**：Steinberger 加入 OpenAI 後宣佈的基金會至今無具體時間表，若基金會正式成立，將對專案治理模式產生深遠影響。
5. **DeepSeek V4 Flash 長期穩定性**：作為新預設模型，V4 Flash 在 OpenClaw Agent 工作流中的 rate limit、工具呼叫準確率及長上下文穩定性仍需社群持續驗證。
6. **Hermes Agent 競爭態勢**：用戶出走速度若加快，可能促使 OpenClaw 加速穩定版本頻道的推出；值得追蹤 Hermes Agent 的 GitHub Star 成長曲線與功能對比。
7. **NVIDIA 與 Bedrock Opus 4.7 Thinking 實測**：新整合的供應商與思考模式 Profile 尚缺乏社群實測數據，後續的 benchmark 與踩坑回報將是重要參考。

---

*報告產生時間：2026-05-02 | 資料來源涵蓋 GitHub、Reddit、Discord、Medium、官方文件及社群部落格*

### 主要來源

- [OpenClaw Releases – GitHub](https://github.com/openclaw/openclaw/releases)
- [OpenClaw Release Notes – Releasebot](https://releasebot.io/updates/openclaw)
- [OpenClaw 2026.4.26 Gateway Crashes – Piunika Web](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)
- [GitHub Issue #32828 – False Rate Limit](https://github.com/openclaw/openclaw/issues/32828)
- [GitHub Issue #48603 – Rate Limit Feature](https://github.com/openclaw/openclaw/issues/48603)
- [ClawSweeper – OpenClaw GitHub Triage Bot – Apidog](https://apidog.com/blog/clawsweeper-openclaw-github-triage-bot/)
- [I Wired OpenClaw to Claude, ChatGPT & Telegram – Medium](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)
- [Best AI Model for OpenClaw 2026 – GetAIPerks](https://www.getaiperks.com/en/blogs/18-best-ai-model-openclaw)
- [Best AI Models for OpenClaw 2026 – GetClawHosting](https://getclawhosting.com/blog/guides/best-ai-models-for-openclaw-2026)
- [Best Mistral Models for OpenClaw – haimaker.ai](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)
- [OpenClaw DeepSeek Docs](https://docs.openclaw.ai/providers/deepseek)
- [OpenRouter – OpenClaw Integration](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)
- [Fix OpenClaw API Key & Rate Limit Errors – LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)
- [OpenClaw vs Claude Code 2026 – ClaudeFast](https://claudefa.st/blog/tools/extensions/openclaw-vs-claude-code)
- [OpenClaw Wikipedia](https://en.wikipedia.org/wiki/OpenClaw)
