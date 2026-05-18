# OpenClaw 每日情報 — 2026-05-18

> 搜尋範圍：過去 24 小時（2026-05-17 ～ 2026-05-18 UTC）
> 資料來源：GitHub、X.com、Reddit、技術部落格

---

## 一、今日重點摘要

1. **最新 beta 版本更新**：主倉庫於 2026-05-18 更新，最新版本為 `v2026.5.16-beta.5`（2026-05-17 發布）。本次 beta 著重在穩定性、認證流程與排程機制，包含 xAI Grok OAuth 登入、更智慧的 cron 任務排隊阻斷機制、更好的 session 與 config 管理，以及多媒體與 provider 可靠性修復。

2. **xAI Grok OAuth 整合上線**：SuperGrok 訂閱用戶現可透過 OAuth 登入使用 `xai/*` 模型與 xAI 工具 provider，無需手動設定 `XAI_API_KEY`。舊版本中 OpenClaw 會誤送 OpenAI-style reasoning effort 參數至 Grok，導致 `grok-4.3` 出現 `Invalid reasoning effort` 錯誤，本次修復。

3. **GitHub 新增 P1 高優先 issue（2026-05-17）**：
   - Issue #83290（作者 giodl73-repo）：ClawSweeper 自動標記為 P1、高可信度重現。
   - Issue #83291（同作者）：ClawSweeper 找到清晰的實作修復方向，P1 狀態，待 maintainer 處理。
   - Issue #83302（作者 M00zyx）：標記需 maintainer 審查，待議中。

4. **Anthropic OAuth token 政策影響持續**：自 2026-04-04 起，Anthropic 禁止第三方工具（包含 OpenClaw）使用訂閱制 OAuth token，必須改用「Extra Usage（隨用隨付）」或獨立 API Key。社群仍有持續討論如何配置對應的認證方式。

5. **Gemini 圖片識別 JSON Schema 問題**：在 OpenAI 相容模式下使用 Gemini 進行圖片識別時，會出現 `Unknown name 'patternProperties'` 錯誤。需切換至 Gemini native 格式配置，或升級至 `2026.2.21` 以上版本以支援 Gemini 3.1 model refs。

6. **預設模型遷移討論（Issue #21897）**：社群正在討論是否將預設模型從 `Claude Opus 4.6` 切換為 `Gemini 3.1 Pro`，PR 仍在審查中，尚未合併。

7. **Discord 社群破 10 萬人 · 立場分歧加劇**：「Friends of the Crustacean」Discord 伺服器突破 100,000 人，但社群態度兩極：部分原本力推 OpenClaw 的開發者開始公開轉向競品；Medium 上有文章〈OpenClaw is Dead〉引發廣泛討論，官方目前未回應。

8. **「Task Brain」架構已穩定**：2026.3.31 beta 首次登場的 Task Brain 控制面板（以 SQLite 為後端帳本，統一四種執行實體）在近期版本中趨於穩定，社群回報正式環境可用性提升。

9. **本地化安裝流程擴充**：onboarding 安裝流程新增繁體中文、簡體中文、英文本地化，降低非英語用戶初始設定門檻。

---

## 二、LLM 串接實戰回報

### Claude (Anthropic)

- **OAuth token 失效**：Anthropic 於 4 月起封鎖 Claude Pro 訂閱 OAuth token 給第三方工具使用，已改用 Extra Usage 方案的用戶需在 OpenClaw 設定中切換至獨立 API Key 模式。若出現 `LLM request rejected: You're out of extra usage`，需至 Anthropic 帳號手動加購額度。
  - 來源：[Apiyi.com Blog — OpenClaw Claude LLM request rejected fix](https://help.apiyi.com/en/openclaw-claude-llm-request-rejected-extra-usage-fix-en.html)

- **Rate Limit（429）處理**：遇到 429 錯誤時建議先執行 `openclaw doctor --fix`，自動修復大多數配置問題；如為 gateway token 不符則執行 `openclaw models auth setup-token` 重新生成。
  - 來源：[LaoZhang AI Blog — OpenClaw API Key Error](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)

### Gemini (Google)

- **JSON Schema 相容問題**：在 OpenAI 相容模式下呼叫 Gemini 圖片識別，因 `patternProperties` 欄位不被接受導致失敗。解法：改用 Gemini native 格式，或升級至 `2026.2.21+`。
  - 來源：[Apiyi.com Blog — Gemini image recognition fix](https://help.apiyi.com/en/openclaw-gemini-image-recognition-fix-openai-compatible-mode-guide-en.html)

- **模型名稱格式**：若 Gemini 模型選擇被拒絕，確認 OpenClaw 版本 ≥ `2026.2.21`，並使用正確 model ref 格式（如 `gemini-3.1-pro`）。

### xAI Grok

- **`grok-4.3` reasoning effort 參數錯誤**：已在最新 beta 修復，SuperGrok 用戶升級後應可正常使用；若仍有問題，確認使用 OAuth 認證而非 API Key 路徑。
  - 來源：[X.com @openclaw — 2026.5.4 release](https://x.com/openclaw/status/2051582130417721696)

---

## 三、社群反應與討論亮點

- **a16z 報告引用**：a16z 的報告指出 OpenClaw 在 GitHub Stars 上超越 Linux 與 React，一度成為史上最高星數倉庫，但近期成長明顯趨緩，社群關注度正在分流至其他 agent 框架。
  - 來源：[a16z on X](https://x.com/a16z/status/2031521410896793764)

- **「OpenClaw is Dead」論戰**：Medium 上一篇批評 OpenClaw 可持續性的文章引發廣泛討論，核心質疑包含架構複雜性提升而文件未跟上、gateway 穩定性、以及社群維護能量是否足夠。
  - 來源：[Medium — OpenClaw is Dead](https://medium.com/data-science-in-your-pocket/openclaw-is-dead-6f6e3cab731f)

- **No-Crypto Discord 規則爭議**：Discord 伺服器的絕對禁幣政策（包含任何加密貨幣討論）在社群引發討論，部分用戶認為過於嚴格，官方立場為：此規則緣自早期 token 詐騙與騷擾事件，屬必要防護。
  - 來源：[Bitcoin.com News — Openclaw's No-Crypto Discord Rule](https://news.bitcoin.com/openclaws-no-crypto-discord-rule-sparks-debate-across-tech-community/)

- **WSL2 升級注意事項**：X 上有用戶詳細分享在 WSL2 高度定製環境下升級 OpenClaw 的步驟（乾跑 → 影子驗證 → 正式升級 → 補丁 → healthcheck），值得 Windows 用戶參考。
  - 來源：[Larus Canus on X](https://x.com/MrLarus/status/2043322911008714919)

---

## 四、值得追蹤的後續議題

| 議題 | 狀態 | 追蹤建議 |
|------|------|----------|
| Issue #83290 / #83291 / #83302 | 開放中（P1）| 關注 ClawSweeper 自動修復進度與 maintainer 回應 |
| Issue #21897（預設模型遷移）| PR 審查中 | 若合併，Claude Opus 4.6 將不再是預設；關注 Gemini 3.1 Pro 相容性 |
| Anthropic OAuth token 政策後續 | 持續影響 | 確認 Extra Usage 用量與費用管理，考慮備用 API Key 策略 |
| Grok-4.3 reasoning effort 修復 | beta 已修，正式版待確認 | 追蹤下一個正式 release 節點 |
| "OpenClaw is Dead" 社群情緒 | 討論活躍 | 觀察官方是否回應；留意是否影響維護者資源投入 |
| Rate limit 統一視圖（Issue #24865）| 提案中 | 若實作，將可在單一介面查看所有 auth profile 的用量狀態 |
