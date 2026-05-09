# OpenClaw 每日情報 2026-05-09

> 搜尋範圍：過去 24 小時｜更新時間：2026-05-09

---

## 一、今日重點摘要

1. **2026.5.6 正式推出**：最新版本於約 15 小時前發布，核心修復 Codex OAuth 路由問題——還原了 2026.5.5 中誤將有效 `openai-codex/*` 路徑改寫為 `openai/*` 的 doctor --fix 邏輯，並改善 plugin/debug proxy 的 fetch 標頭處理，減少 OAuth 中斷與請求失敗情形。

2. **2026.5.5 頻道強化更新**：上一版本涵蓋 Feishu 話題填充、LINE DM 驗證、Telegram/Codex 草稿修復、Matrix 審核重試、Discord 心跳計時、Slack 重連診斷、iOS LAN 配對、插件診斷、Gateway 關閉清理、媒體去重、Control UI session/runtime 顯示等多項頻道層強化。

3. **Kimi K2.5 登頂社群評比**：截至 2026-05-08，社群投票將 Kimi K2.5 列為 OpenClaw 最佳搭配模型，緊隨其後的是 Claude Opus 4.6 與 GLM 4.7。Claude Opus 4.6 仍是官方推薦首選，因其在長上下文處理與提示注入防禦上表現最佳。

4. **Anthropic OAuth 訂閱禁令持續發酵**：自 4 月 4 日起，Claude Pro/Max 訂閱額度不再涵蓋 OpenClaw 等第三方工具，使用者須全面切換至 API Key 模式。此政策引發社群廣泛討論，甚至導致 OpenClaw 創始人的 Claude 帳號一度遭到停權。

5. **OpenClaw vs. Hermes 論戰持續升溫**：r/openclaw（現有 103,000 位成員）內 OpenClaw vs. Hermes 的比較討論成為當前自主 AI agent 社群最熱門話題，數十條比較串同時活躍中。

6. **Discord 整合崩潰問題仍在追蹤**：4 月底 2026.4.26 更新引發的 Discord gateway 崩潰、高 CPU 佔用、指令無回應問題，儘管後續版本已推出修復，部分使用者仍反映問題未完全解決。

7. **DeepSeek 伺服器穩定性疑慮**：DeepSeek V3.2 以低成本（$0.28/$0.42 per M tokens）受到預算有限的使用者青睞，但在尖峰時段頻繁出現 503 錯誤，首次 token 回應時間過長，於生產環境可靠度仍不足。

8. **ClawHub 安全警告**：安全審計發現 ClawHub 技能市集中約 2,857 個技能裡，有 341 個（約 12%）被標記為惡意，使用者在安裝第三方技能時需提高警覺。

9. **OpenClaw-RL 強化學習擴展亮相**：新擴展 OpenClaw-RL 正式發布，允許 agent 透過試錯與回饋機制學習最優行為，不再依賴靜態規則，為自主工作流引入強化學習能力。

---

## 二、LLM 串接實戰回報

### Claude（Anthropic）

- **OAuth 禁令影響**：4 月 4 日起，Anthropic 正式禁止訂閱 OAuth token 用於 OpenClaw，凡仍使用 OAuth 模式者將全面失效。建議執行 `openclaw models auth setup-token` 並切換至 API Key 模式。若 doctor --fix 已在 2026.5.5 版本中將模型路徑改成 `openai/*`，需手動執行 `openclaw models set openai-codex/gpt-5.5 && openclaw config validate` 還原。
  - 來源：[Anthropic 封鎖第三方工具說明 | The Register](https://www.theregister.com/2026/04/06/anthropic_closes_door_on_subscription/)
  - 來源：[OpenClaw Ban 完整指南 | Kersai](https://kersai.com/anthropic-killed-third-party-claude-access-heres-every-workaround-that-still-works/)

- **Claude Opus 4.6 成本爆炸問題**：GitHub Issue #17589 記錄了 Gateway 重啟/AbortError 觸發完整 LLM-run 重試，導致 Opus 費用異常飆升，並伴隨誤報「API rate limit reached」警告。
  - 來源：[Issue #17589 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/17589)

- **誤報 rate limit 問題**：Issue #32828 反映即使 API 完全正常，仍在所有模型上出現「API rate limit reached」誤報。
  - 來源：[Issue #32828 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/32828)

### OpenAI（GPT / Codex）

- **Cloudflare 403 封鎖**：從 2026.4.12 升級至 2026.4.14 後，openai-codex provider 在每次請求時均遭 Cloudflare 403 拒絕（Issue #66633）。2026.5.6 已針對此問題進行修復。
  - 來源：[Issue #66633 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/66633)

- **GPT-5.5 OAuth 整合**：使用者成功透過 Telegram bot 串接 Claude 與 ChatGPT，並進一步觸發 Claude Code 自動部署 Vercel 網站，整個流程可完全從 Telegram 訊息驅動。
  - 來源：[Medium — Mohit Aggarwal](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)

### DeepSeek

- **尖峰時段 503 問題**：V3.2 在社群排名中位列第 15，但在高流量時段伺服器穩定性不佳，503 錯誤及首 token 延遲問題頻繁，不建議作為生產環境主力模型。
  - 來源：[Best DeepSeek Models for OpenClaw | haimaker.ai](https://haimaker.ai/blog/best-deepseek-models-for-openclaw/)

### Mistral

- **Devstral 工具呼叫最穩定**：Mistral 系列中 Devstral 在多步 OpenClaw agent 工作流中工具呼叫可靠性最高，同時 Mistral 整體具備歐洲資料駐留及多語言支援優勢。
  - 來源：[Best Mistral Models for OpenClaw | haimaker.ai](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

### Gemini（Google）

- **長文件處理首選**：Gemini 3 Pro 為長文件處理推薦首選；Gemini 2.5 Flash 提供最慷慨的免費額度及快速回應，同時支援多模態輸入，適合入門使用者。
  - 來源：[Best LLM for OpenClaw 2026 | DEV Community](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### Grok（xAI）

- **社群評價偏負面**：多位使用者表示根據社群評測直接跳過 Grok，實際整合體驗及穩定性口碑均不佳。
  - 來源：[Best Model for OpenClaw | Price Per Token](https://pricepertoken.com/leaderboards/openclaw)

### OpenRouter

- 作為 OpenAI 相容端點的中介支援多家模型，適合希望統一管理多 provider API 的使用者，詳情可參考官方文件。
  - 來源：[OpenRouter Models](https://openrouter.ai/models)

---

## 三、社群反應與討論亮點

- **r/openclaw（103K 成員）最熱話題**：OpenClaw vs. Hermes 的框架選擇討論持續延燒，大量比較串聚焦在 agent 自主性、插件生態、穩定性三個維度，暫無明確社群共識。

- **Anthropic 禁令輿論效應**：Medium 文章《The OpenClaw Ban That Exposed Anthropic's Real Problem》（作者 Suleiman Tawil）引發大量轉發，核心論點為 Anthropic 以訂閱價格補貼 API 等級工作負載，商業模型難以為繼。Hacker News 同步討論串亦相當熱絡。
  - 來源：[Medium — Suleiman Tawil](https://medium.com/@stawils/the-openclaw-ban-that-exposed-anthropics-real-problem-fe8f10aa0e80)
  - 來源：[Hacker News 討論串](https://news.ycombinator.com/item?id=47633396)

- **2026.4.26 更新崩潰後遺症**：Reddit 貼文確認大量使用者在更新後遭遇 Discord/Telegram/其他聊天整合全面失效，高 CPU、高延遲問題雖在後續版本部分修復，但社群情緒仍偏向不信任快速升級。

- **ClawHub 惡意技能事件**：安全審計發現約 12% 技能為惡意，社群建議在安裝任何非官方認證技能前，先透過 ClawHub 審計報告核實。

- **OpenClaw-RL 正式亮相**：強化學習擴展引發技術社群興趣，多位開發者在 Discord 詢問如何與現有 agent workflow 整合，官方尚未發布詳細文件。

---

## 四、值得追蹤的後續議題

1. **2026.5.6 Codex OAuth 修復驗證**：OpenAI Codex OAuth 路徑問題是否在 2026.5.6 中完整解決，需持續關注後續 issue 回報，特別是 Cloudflare 403 問題是否徹底消除。

2. **Anthropic API Key 費用影響評估**：大量使用者從訂閱 OAuth 轉至 Pay-as-you-go API，實際費用衝擊與使用者留存率值得追蹤，後續可能影響 OpenClaw 的 Claude 使用佔比。

3. **OpenClaw vs. Hermes 競爭格局**：隨著 Hermes 功能持續迭代，兩框架在 agent 自主性與生態豐富度上的競爭態勢，是下一季度社群最關鍵的觀察點。

4. **ClawHub 安全機制改進**：341 個惡意技能事件是否推動官方建立更嚴格的技能審核流程，相關政策更新值得關注。

5. **OpenClaw-RL 文件與整合指南**：強化學習擴展目前文件空白，預期近期將發布 Getting Started 指南，可作為 agent workflow 優化的重要工具。

6. **Discord 整合穩定性**：2026.4.26 引發的 gateway 崩潰問題雖有修復，但使用者反映未完全解決，後續版本中 Discord heartbeat 機制的進一步強化值得持續追蹤。

---

*報告生成時間：2026-05-09 | 資料來源：GitHub Releases、Reddit r/openclaw、Medium、DEV Community、Hacker News、The Register、TechCrunch、各技術部落格*
