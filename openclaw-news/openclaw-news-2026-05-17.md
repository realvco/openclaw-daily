# OpenClaw 每日情報 2026-05-17

> 搜尋時間範圍：過去 24 小時（2026-05-16 ～ 2026-05-17）

---

## 1. 今日重點摘要

1. **創辦人 30 天燒掉 130 萬美元 OpenAI 費用（今日爆出）**：OpenClaw 創辦人 Peter Steinberger 公開 API 帳單截圖，30 天內在 OpenAI 花費 1,305,088.81 美元，涵蓋 6,030 億 tokens、760 萬次 API 請求，全由約 100 個 Codex 實例、三人團隊產生。費用由其現任雇主 OpenAI 承擔，引發社群對 AI 成本結構的激烈討論。
   - 來源：[Tom's Hardware](https://www.tomshardware.com/tech-industry/artificial-intelligence/openclaw-creator-burns-through-1-3-million-in-openai-api-tokens-in-a-single-month)

2. **v2026.5.16-beta.3 釋出**：新增 xAI Grok OAuth（SuperGrok 訂閱者免設 API Key）、`openclaw cron run --wait` 旗標（支援自訂 timeout 與輪詢間隔），並修復 Grok Responses API 的 `Invalid reasoning effort` 錯誤。
   - 來源：[GitHub Releases](https://github.com/openclaw/openclaw/releases)

3. **v2026.5.14-beta.2 新增原生 Codex App-Server 支援**：OpenAI agent turns 可透過 OpenClaw 直接呼叫 Codex，提供更乾淨的工具搜尋、訂閱認證與獨立 per-agent 狀態隔離。
   - 來源：[GitHub – openclaw/openclaw](https://github.com/openclaw/openclaw)

4. **社群「OpenClaw 已死」論戰持續發酵**：Medium 文章《OpenClaw is Dead》主指 Anthropic 取消訂閱 OAuth、創辦人出走 OpenAI、512 個安全漏洞（含 8 個 critical）三大死因，Forrester 以《OpenClaw Is Dead — Long Live OpenClaw》回應，承認動能減弱但認為開源社群仍具韌性。

5. **April 2026 提供商清單（Provider Manifest）確立六大可熱替換 LLM**：GPT-5.5、Claude Opus 4.7、Gemini 3.1 Pro、DeepSeek、Ollama、Gemma 4 現在全部支援執行時切換，無需重建工作流；社群投票最受歡迎模型截至 5/16 依序為 Kimi K2.5、Claude Opus 4.6、GLM 4.7。
   - 來源：[MindStudio Blog](https://www.mindstudio.ai/blog/openclaw-april-2026-model-agnostic-provider-manifest)、[PricePerToken](https://pricepertoken.com/leaderboards/openclaw)

6. **OpenRouter 回應快取上線（v2026.5.4）**：新增 `X-OpenRouter-Cache` header 支援，opt-in 啟用可降低重複 prompt 呼叫成本。
   - 來源：[Blink Blog](https://blink.new/blog/openclaw-whats-new-may-2026)

7. **Discord「Friends of the Crustacean」突破 10 萬成員**：自 2026 年 1 月成立，六週內達成，增速超越 Midjourney Discord 紀錄，但社群氣氛因穩定性事故與 Anthropic 禁令有所降溫。
   - 來源：[openclaw.report](https://openclaw.report/news/openclaw-discord-100k-members)

8. **Anthropic 訂閱 OAuth 禁令（2026-04-04）影響持續**：Claude Pro/Max 訂閱者無法再透過 OAuth 在 OpenClaw 使用 Claude，必須切換 API Key 或購買 Extra Usage；假「rate limit」bug（Issue #32828）仍未關閉，影響多個模型提供商。
   - 來源：[LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)、[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828)

9. **v2026.4.26 更新引發 Discord/Gateway 崩潰事件尚未完全平復**：部分用戶在此事故後公開宣告轉向 Hermes Agent，稱其「更新更順、Bug 更少」。
   - 來源：[Piunikaweb](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **訂閱 OAuth 終止（2026-04-04）**：Anthropic 明確禁止 Claude Pro/Max 訂閱 OAuth 在第三方工具中使用。用戶出現 `API rate limit reached`、`LLM request rejected – You're out of extra usage` 兩類錯誤。
  - 解法：改用 API Key 模式（`openclaw doctor --fix` 可自動修正），或購買 Extra Usage。
  - 來源：[LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)、[Apiyi Blog](https://help.apiyi.com/en/openclaw-claude-llm-request-rejected-extra-usage-fix-en.html)

- **OAuth Bearer Header 衝突（v2026.3.28+）**：v2026.3.28 起 OpenClaw 改用 `Authorization: Bearer` 傳送 Anthropic OAuth token，導致 HTTP 401「OAuth authentication is currently not supported」。回退至 v2026.3.24 可繞過，官方後續版本已修正。
  - 來源：[GitHub Issue #60279](https://github.com/openclaw/openclaw/issues/60279)

- **Claude Opus 4.7**：SWE-bench Pro 64.3%，為程式碼審查首選，但 Gateway 重啟時整個 LLM run 被強制重試，高成本模型費用會瞬間暴增（Issue #17589 追蹤中）。
  - 來源：[DEV.to – Best LLM for OpenClaw 2026](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)、[GitHub Issue #17589](https://github.com/openclaw/openclaw/issues/17589)

### GPT（OpenAI / Codex）

- **GPT-5.5 agentic benchmark 表現最佳**，但 context 上限 128K tokens，超出後任務必須拆分。
- **Codex App-Server 原生整合（v2026.5.14-beta.2）**：每個 agent 狀態必須隔離，否則出現工具搜尋衝突。
- **創辦人帳單曝光**：100 個 Codex agent 30 天燒 130 萬美元，也間接揭示大規模部署下的 OpenAI 成本結構。
  - 來源：[Tom's Hardware](https://www.tomshardware.com/tech-industry/artificial-intelligence/openclaw-creator-burns-through-1-3-million-in-openai-api-tokens-in-a-single-month)

### Gemini（Google）

- **社群推薦為多數場景首選**：Gemini 3.1 Pro 支援 1M tokens context、成本最低、免費開發層，適合大型 PR diff 一次性攝入。
- **社群 Issue #21897**：提案將預設模型從 Claude Opus 4.6 遷移至 Gemini 3.1 Pro，討論進行中。
  - 來源：[DEV.to – Using Gemini with OpenClaw](https://dev.to/matthewrevell/using-gemini-with-openclaw-setup-guide-real-use-cases-2i48)、[MindStudio Blog](https://www.mindstudio.ai/blog/openclaw-april-2026-model-agnostic-provider-manifest)

### xAI Grok

- **Grok 4.3 成為 xAI 後端預設**：v2026.5.2 起整合，v2026.5.16-beta.3 加入 SuperGrok OAuth，省去 API Key 設定。
- **`reasoning_effort` 參數衝突修復**：OpenAI 風格 reasoning effort 傳入 Grok Responses API 導致 `Invalid reasoning effort` 錯誤，已修復。
  - 來源：[BuildFastWithAI](https://www.buildfastwithai.com/blogs/openclaw-2026-5-2-release)

### OpenRouter

- **回應快取（v2026.5.4）**：`X-OpenRouter-Cache` header opt-in 快取，可顯著降低重複 prompt 費用；同步擴充 app-attribution 類別傳送。
  - 來源：[Blink Blog](https://blink.new/blog/openclaw-whats-new-may-2026)

### 429 Rate Limit 通用踩坑彙整

| 來源 | 症狀 | 處理方式 |
|---|---|---|
| 上游模型配額 | 所有模型皆失敗 | 等待 `retry-after` 秒數 |
| ClawHub Skill 下載限制 | 安裝外掛時失敗 | 稍後重試或切換映像 |
| Gateway cooldown | 重啟後恢復 | `openclaw doctor --fix` |
| Fallback 設定不足 | 特定模型失敗 | 設定備援模型 |
| Agent session 並發過高 | 費用激增 | 限制 session 並發數 |

來源：[OpenClaw Rate Limit Guide](https://www.getopenclaw.ai/help/rate-limits-quota-management)、[LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)

---

## 3. 社群反應與討論亮點

- **$130 萬帳單震撼全場**：Steinberger 的帳單截圖在 X.com、Reddit 與 Hacker News 引發廣泛討論，部分開發者將其視為「AI agent 成本失控」的警示；也有聲音強調這是三人團隊、100 個並行 agent 的極端邊界情境，並非一般用戶場景。
  - 來源：[Tom's Hardware](https://www.tomshardware.com/tech-industry/artificial-intelligence/openclaw-creator-burns-through-1-3-million-in-openai-api-tokens-in-a-single-month)

- **「OpenClaw is Dead」論戰**：Mehul Gupta（Medium）與 Forrester 的互相辯論引起關注，核心分歧在於「社群動能是否能彌補商業模式脆弱性」；Reddit 討論串中鼓吹「換用 Claude Code」的帖子熱度明顯上升。
  - 來源：[Medium](https://medium.com/data-science-in-your-pocket/openclaw-is-dead-6f6e3cab731f)、[Forrester](https://www.forrester.com/blogs/openclaw-is-dead-long-live-openclaw/)

- **Anthropic OAuth 禁令爭議延燒**：社群對 Anthropic 決策仍存在分歧，有開發者認為此舉傷害低成本試驗用戶，也有部分人（如 Ben Newton）認為訂閱制長期補貼重度 OpenClaw 用戶財務上不可持續。
  - 來源：[Ben Newton Blog](https://benenewton.com/blog/everyone-mad-anthropic-killed-openclaw-im-not)

- **動態模型路由獲正面評價**：Provider Manifest 功能（同一工作流可動態切換模型）被視為 OpenClaw 最具差異化優勢，社群中有多個「本地 Gemma 4 分類 → GPT-5.5 實作 → Claude 審查」工作流分享。
  - 來源：[MindStudio Blog](https://www.mindstudio.ai/blog/openclaw-april-2026-model-agnostic-provider-manifest)

- **Hermes Agent 分流加速**：v2026.4.26 崩潰事件後，可觀察到部分 OpenClaw 用戶公開宣告轉向 Hermes Agent，稱其維護更穩、Bug 更少，兩個專案的競爭態勢進一步白熱化。
  - 來源：[Piunikaweb](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)、[Kilo.ai – OpenClaw vs Hermes 2026](https://kilo.ai/articles/openclaw-vs-hermes-what-reddit-says)

---

## 4. 值得追蹤的後續議題

1. **Issue #32828（假 rate limit 錯誤）**：假 `API rate limit reached` 問題影響多個模型提供商，官方尚未關閉，需持續觀察修復進度。
   - 追蹤：[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828)

2. **v2026.5.16 正式版時程**：SuperGrok OAuth、cron wait 控制目前在 beta.3，正式版預計近期釋出，測試進展值得盯緊。
   - 追蹤：[GitHub Releases](https://github.com/openclaw/openclaw/releases)

3. **Issue #21897（預設模型遷移）**：從 Claude Opus 4.6 遷移至 Gemini 3.1 Pro 的提案可能影響大量用戶預設體驗，最終決議值得關注。
   - 追蹤：[GitHub Issue #21897](https://github.com/openclaw/openclaw/issues/21897)

4. **Issue #17589（Gateway 重啟費用爆炸）**：高成本模型用戶受影響最重，官方尚未提供完整修復，需追蹤。
   - 追蹤：[GitHub Issue #17589](https://github.com/openclaw/openclaw/issues/17589)

5. **Codex App-Server 整合成熟度**：v2026.5.14-beta.2 為早期測試版，per-agent 狀態隔離機制的可靠性與多 agent 並發穩定性仍有待觀察。

6. **OpenClaw 與 Hermes Agent 競爭態勢**：部分用戶已因穩定性問題轉向 Hermes，需追蹤 OpenClaw 在穩定性上能否回防，防止進一步流失。
   - 參考：[Kilo.ai 分析](https://kilo.ai/articles/openclaw-vs-hermes-what-reddit-says)

---

*報告產生時間：2026-05-17 | 資料來源：GitHub、Tom's Hardware、DEV.to、Medium、Forrester、Piunikaweb、MindStudio、PricePerToken、LaoZhang AI Blog 等公開平台*
