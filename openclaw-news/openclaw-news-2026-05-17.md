# OpenClaw 每日情報 2026-05-17

> 搜尋時間範圍：過去 24 小時（2026-05-16 ～ 2026-05-17）

---

## 1. 今日重點摘要

1. **v2026.5.16-beta.3 釋出**：新增 xAI Grok OAuth 登入，支援 SuperGrok 訂閱者直接使用 `xai/*` 系列模型，無需手動設定 `XAI_API_KEY`；同步新增 `openclaw cron run --wait` 旗標，支援自訂 timeout 與輪詢間隔。

2. **Grok 4.3 成為 xAI 後端預設模型**：xAI Grok 4.3 已正式整合進模型目錄，現有 xAI 設定更新後即自動切換，無需額外配置；先前因 OpenAI 樣式 reasoning effort 參數傳入 Grok Responses API 導致 `Invalid reasoning effort` 錯誤的問題已修復。

3. **GitHub 開放議題持續高位**：主倉庫目前有 3,600+ 個 Issues 與 3,500+ 個 PRs，5 月 16 日單日新增多條含 `bug`/`regression` 標籤的議題（#82628、#82627、#82623、#82621 等），顯示版本迭代速度仍快但穩定性壓力不輕。

4. **Anthropic 訂閱 OAuth 禁令持續生效**：自 2026-04-04 起 Anthropic 禁止第三方工具使用 Claude Pro/Max 的訂閱 OAuth 授權，Claude 用戶若要繼續在 OpenClaw 中使用 Claude 模型，必須切換至 API Key 模式或購買 Extra Usage，社群反彈仍未平息。

5. **假「API rate limit reached」Bug 受關注**：Issue #32828 回報即使上游 API 完全正常，OpenClaw 仍顯示 `API rate limit reached` 錯誤，影響多個模型提供商，目前尚未關閉。

6. **OpenRouter 回應快取上線（v2026.5.4）**：新增透過 `X-OpenRouter-Cache` header 的 opt-in 回應快取機制，並擴充傳送至 OpenRouter 的 app-attribution 類別，有助於降低重複呼叫成本。

7. **檔案傳輸外掛正式加入（v2026.5.3）**：新外掛提供四個 Agent 工具：`file_fetch`、`dir_list`、`dir_fetch`、`file_write`，支援節點間二進位檔案操作，單趟最大傳輸上限為 16 MB。

8. **Discord 社群突破 10 萬成員**：OpenClaw 官方 Discord「Friends of the Crustacean」自 2026 年 1 月成立，約六週內達成 10 萬成員里程碑，增長速度超越 Midjourney Discord 當年紀錄。

9. **「OpenClaw 已死」論戰再起**：Medium、Forrester 等平台出現多篇「OpenClaw is Dead」文章，核心矛頭指向 Anthropic 取消訂閱優惠、創辦人 Peter Steinberger 出走 OpenAI（2 月），以及年初曝光的 512 個安全漏洞（含 8 個 critical）。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **訂閱 OAuth 終止（2026-04-04）**：Anthropic 明確禁止 Claude Pro/Max 訂閱在 OpenClaw 等第三方工具中使用 OAuth Token。大量用戶反映突然出現 `API rate limit reached` 與 `LLM request rejected – You're out of extra usage` 錯誤。
  - 解決方式：改用 API Key 模式（`openclaw doctor --fix` 可自動修正大部分設定），或購買 Extra Usage。
  - 來源：[Fix Every OpenClaw Error – LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)、[Apiyi.com Blog](https://help.apiyi.com/en/openclaw-claude-llm-request-rejected-extra-usage-fix-en.html)
- **Context 上限**：Claude Opus 4.7 標準 API 支援 200K tokens（1M tokens Beta 可透過 platform.claude.ai 申請），OpenClaw 預設保留最後 100 則對話訊息，可透過 `maxHistoryMessages` 調整。
  - 來源：[OpenClaw FAQ](https://docs.openclaw.ai/help/faq)
- **Gateway 重啟觸發重試成本爆炸**：Issue #17589 記錄 Gateway 重啟或 `AbortError` 導致整個 LLM run 被強制重試，使用 Claude Opus 4.7 的用戶費用暴增，目前仍在追蹤中。
  - 來源：[GitHub Issue #17589](https://github.com/openclaw/openclaw/issues/17589)

### GPT（OpenAI / Codex）

- **Codex App-Server 原生支援**：v2026 系列新增 Codex app-server 的 OpenAI agent turns 支援，ChatGPT 訂閱者可授權 OpenClaw Agent 直接使用 Codex 做為後端，但每個 agent 的 state 必須隔離，否則會出現工具搜尋衝突。
  - 來源：[GitHub – openclaw/openclaw](https://github.com/openclaw/openclaw)
- **GPT-5.5 限制**：標準 API context 上限為 128K tokens（1M tokens 通道目前不對一般用戶開放），適合任務規模不超過 128K 的自主 Agent 工作流。
  - 來源：[DEV.to – Best LLM for OpenClaw 2026](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### Gemini（Google）

- **推薦為大多數 OpenClaw 部署的預設模型**：Gemini 3.1 Pro 支援 1M tokens、成本最低、免費開發層，且原生多模態，被多篇評測列為一般工作流的首選。
  - 來源：[DEV.to – Best LLM for OpenClaw 2026](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)、[MindStudio Blog](https://www.mindstudio.ai/blog/openclaw-april-2026-model-agnostic-provider-manifest)
- **GitHub Issue #21897**：有用戶提案將預設模型從 Claude Opus 4.6 遷移至 Gemini 3.1 Pro，討論中。
  - 來源：[GitHub Issue #21897](https://github.com/openclaw/openclaw/issues/21897)

### xAI Grok

- **Grok 4.3 成為預設**：v2026.5.2 起整合 Grok 4.3，現有 xAI 設定自動切換無需額外設定。
- **OAuth 問題修復**：先前版本誤將 OpenAI 風格的 `reasoning_effort` 參數傳給 Grok Responses API，導致 `Invalid reasoning effort` 錯誤，已於最新版修復。
- **SuperGrok OAuth 登入**：v2026.5.16-beta.3 新增 SuperGrok 訂閱者的 OAuth 登入流程，省去 API Key 設定步驟。
  - 來源：[OpenClaw 2026.5.2 Release – BuildFastWithAI](https://www.buildfastwithai.com/blogs/openclaw-2026-5-2-release)、[AI News Detail](https://blockchain.news/ainews/xai-grok-4-3-powers-openclaw-update)

### OpenRouter

- **回應快取支援（v2026.5.4）**：新增 `X-OpenRouter-Cache` header 支援，opt-in 啟用可大幅降低相同 prompt 的重複呼叫費用。
  - 來源：[Blink Blog – What's New in OpenClaw May 2026](https://blink.new/blog/openclaw-whats-new-may-2026)

### 429 Rate Limit 通用踩坑彙整

OpenClaw 的 429 錯誤可能來自五個不同來源，相同的錯誤訊息掩蓋了不同根因：

| 來源 | 症狀 | 處理方式 |
|---|---|---|
| 上游模型提供商配額 | 所有模型皆失敗 | 等待 `retry-after` 秒數 |
| ClawHub Skill 下載限制 | 安裝外掛時失敗 | 稍後重試或切換映像 |
| Gateway cooldown 狀態 | 重啟後恢復 | `openclaw doctor --fix` |
| Fallback 覆蓋不足 | 特定模型失敗 | 設定備援模型 |
| Agent session 過度呼叫 | 費用激增 | 限制 session 並發數 |

來源：[OpenClaw Rate Limit Guide](https://www.getopenclaw.ai/help/rate-limits-quota-management)、[LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-rate-limit-exceeded-429)

---

## 3. 社群反應與討論亮點

- **「OpenClaw is Dead」論戰**：Medium 上 Mehul Gupta 撰文《OpenClaw is Dead》，指出三大死因：Anthropic 切斷訂閱優惠、創辦人出走 OpenAI、安全性危機。Forrester 同步發文以「OpenClaw Is Dead — Long Live OpenClaw」回應，認為開源社群仍有韌性，但承認整體動能已明顯減弱。
  - 來源：[Medium](https://medium.com/data-science-in-your-pocket/openclaw-is-dead-6f6e3cab731f)、[Forrester](https://www.forrester.com/blogs/openclaw-is-dead-long-live-openclaw/)

- **v2026.4.26 更新災難**：上月底更新後大量 Discord 與 Gateway 用戶遭遇崩潰，部分老用戶公開表示轉投 Hermes Agent，稱其「更新更順、Bug 更少」。
  - 來源：[Piunikaweb](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)

- **Anthropic 禁令爭議**：多位開發者在社群中表達不滿，認為 Anthropic 的決定打擊了以低成本試驗 OpenClaw 的入門用戶；亦有部分聲音（如 Ben Newton 的部落格）認為此舉合理，因為 Anthropic 長期補貼重度 OpenClaw 用戶在財務上不可持續。
  - 來源：[Ben Newton Blog](https://benenewton.com/blog/everyone-mad-anthropic-killed-openclaw-im-not)

- **安全漏洞仍受關注**：Kaspersky 與 The Register 持續追蹤 OpenClaw 生態系的安全問題，社群貢獻的 Skills 插件仍被視為潛在攻擊面。
  - 來源：[Kaspersky Blog](https://www.kaspersky.com/blog/openclaw-vulnerabilities-exposed/55263/)、[The Register](https://www.theregister.com/2026/02/02/openclaw_security_issues/)

- **動態模型路由受期待**：MindStudio 介紹的 Provider Manifest 功能（可在同一工作流中動態切換不同模型，如本地 Gemma 4 做初步分類、GPT-5.5 做實作、Claude 做最終審查）在社群引發正面討論，被視為 OpenClaw 最具差異化的能力之一。
  - 來源：[MindStudio Blog](https://www.mindstudio.ai/blog/openclaw-april-2026-model-agnostic-provider-manifest)

---

## 4. 值得追蹤的後續議題

1. **Issue #32828（假 rate limit 錯誤）**：假 `API rate limit reached` 問題影響範圍廣，尚未關閉，需持續觀察官方修復進度。
   - 追蹤連結：[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828)

2. **Issue #21897（預設模型遷移討論）**：是否將預設模型從 Claude Opus 4.6 切換至 Gemini 3.1 Pro 的提案，可能影響現有用戶的預設行為，需留意最終決議。
   - 追蹤連結：[GitHub Issue #21897](https://github.com/openclaw/openclaw/issues/21897)

3. **v2026.5.16 正式版時程**：目前仍在 beta.3 階段，包含 SuperGrok OAuth 與 cron wait 控制，正式版預計近期釋出。
   - 追蹤連結：[GitHub Releases](https://github.com/openclaw/openclaw/releases)

4. **Issue #17589（Gateway 重啟成本爆炸）**：對使用高成本模型（如 Claude Opus 4.7）的用戶影響重大，官方尚未提供完整修復。
   - 追蹤連結：[GitHub Issue #17589](https://github.com/openclaw/openclaw/issues/17589)

5. **安全外掛生態系整治進度**：Kaspersky 與 The Register 持續追蹤，官方 Plugin 審查機制是否強化值得關注。

6. **Hermes Agent 競爭態勢**：部分 OpenClaw 用戶在 v2026.4.26 事故後轉向 Hermes Agent，需追蹤 OpenClaw 能否在穩定性上作出反擊，防止用戶進一步流失。

---

*報告產生時間：2026-05-17 | 資料來源：GitHub、DEV.to、Medium、Reddit、Discord、Forrester、Kaspersky、The Register 等公開平台*
