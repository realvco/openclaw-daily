# OpenClaw 每日情報 2026-04-30

> 資料涵蓋時段：2026-04-29 ～ 2026-04-30（過去 24 小時）
> 整理語言：繁體中文

---

## 1. 今日重點摘要

1. **v2026.4.26 閘道器（Gateway）崩潰問題大規模爆發**：升級至 v2026.4.26 的用戶在 Reddit（r/openclaw）與 Discord 社群集中反映 Gateway 崩潰或啟動循環失敗，並伴隨高 CPU 佔用與高延遲問題。官方確認 v2026.4.26 已針對孤立的 plugin 設定錯誤進行降級（degraded mode）處理，但此版本仍存在無法完全覆蓋的場景，修補工作持續進行中。（來源：[piunikaweb.com](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)、[GitHub Issue #71771](https://github.com/openclaw/openclaw/issues/71771)、[Issue #72042](https://github.com/openclaw/openclaw/issues/72042)）

2. **DeepSeek V4 Flash 正式成為 OpenClaw 新預設模型**：隨 v2026.4.26 正式上線，DeepSeek V4 Flash 取代原有預設，成為 OpenClaw onboarding 的首選模型；V4 Pro 則提供高效能複雜任務（程式碼生成、多步驟推理）選項。此舉引發社群對成本控制與穩定性的熱烈討論。（來源：[TechNode](https://technode.com/2026/04/27/deepseek-v4-becomes-default-model-for-openclaw/)、[CXO Digitalpulse](https://www.cxodigitalpulse.com/openclaw-integrates-deepseek-v4-models-to-enhance-ai-agent-performance/)）

3. **Cerebras 正式加入 Bundled Provider 陣列**：v2026.4.26 新增 Cerebras 作為內建 plugin，附帶 onboarding 流程、靜態 model catalog、完整文件與 manifest-owned endpoint metadata，提供超高速推理選項，吸引需要低延遲場景的開發者關注。

4. **Google Meet 整合以 Bundled Plugin 形式正式推出**：個人 Google auth、Chrome/Twilio 即時通話、配對節點 Chrome 支援、出席與工件匯出，以及針對已開啟 Meet 分頁的恢復工具，全數納入 v2026.4.26。

5. **`openclaw migrate` 遷移工具正式上線**：新指令支援 plan、dry-run、JSON 輸出、遷移前備份、onboarding 偵測，並內建 Hermes importer，可一鍵搬移設定、memory/plugin hints、model providers、MCP servers、skills 與憑證，大幅降低從 Hermes 等競品切換的門檻。

6. **GitHub Copilot Provider 新增 GPT-5.5 與 Claude Opus 4.7 1M（內部）**：Issue #72805 請求將 `gpt-5.5` 與 `claude-opus-4.7-1m-internal` 納入 GitHub Copilot provider 的預設 model 清單，社群反應熱烈，代表 1M context 場景需求明顯上升。（來源：[GitHub Issue #72805](https://github.com/openclaw/openclaw/issues/72805)）

7. **ClawSweeper Bot 自動分類超過 7,000 個 Issue/PR**：OpenClaw 維護團隊建立的 ClawSweeper bot 持續以滾動方式審查所有 open issue 與 PR，累計自動關閉 10,217 項，本週關閉率為 0.1%/輪次，有效緩解大型開源專案的維護壓力。（來源：[Apidog Blog](https://apidog.com/blog/clawsweeper-openclaw-github-triage-bot/)）

8. **社群出現 Hermes 切換潮**：因 v2026.4.26 穩定性問題，部分長期用戶公開表示轉向 Hermes Agent，並對 OpenClaw 更新品質提出批評。社群情緒較前日明顯趨向負面。

9. **OpenTelemetry 觀測覆蓋持續擴大**：model 呼叫、token 用量、tool loop、harness run、exec process、outbound delivery、context assembly、memory pressure 等關鍵路徑均已納入 OTel tracing，適合企業級部署監控需求。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **訂閱層 OAuth 持續失效**：Anthropic 自 2026-04-04 起禁止 Claude Pro/Max 訂閱方案的 OAuth token 用於第三方工具（包含 OpenClaw），用戶需改用 API key 模式（$5 input / $25 output per million tokens）。Discord 社群仍有新用戶因未注意此變更而卡關，建議在文件首頁加入顯眼警示。（來源：[answeroverflow.com](https://www.answeroverflow.com/m/1478788645472436264)、[Medium - Mohit Aggarwal](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)）
- **False Rate Limit 誤判 Bug（Issue #32828）**：即使 API 完全正常，部分用戶仍持續觸發「API rate limit reached」錯誤，根因為 gateway 的 `searchSkillsFromClawHub` 函式未正確帶入 ClawHub auth token（Issue #52949），導致持續收到 429 回應並觸發錯誤 cooldown。臨時解法：`openclaw models auth setup-token` 重新產生 gateway token。

### DeepSeek

- **V4 Flash 穩定性風險提醒**：社群建議不應將 DeepSeek V4 作為生產環境的唯一模型，API 穩定性不及 Gemini Flash 或 Claude Haiku，建議搭配穩定 provider 作為 fallback。（來源：[haimaker.ai - Best Models for OpenClaw](https://haimaker.ai/blog/best-models-for-clawdbot/)）
- **V4 thinking/replay 修復**：v2026.4.26 修正 DeepSeek thinking 模式在 follow-up tool-call 輪次的重播行為異常，升級後應重新測試相關 agent 工作流。

### Gemini（Google）

- **Per-Provider Cooldown Bug（Issue #5744）**：當單一 Google model 觸及 per-model TPM rate limit 時，整個 `google` provider 進入 cooldown，導致其他仍有配額的 Gemini model（如 Flash）無法使用。多模型並行或 fallback 配置的用戶需特別注意，官方修復尚未發布。

### GPT / OpenAI

- **v2026.4.26 Breaking Change：`coding-agent` 預設使用 GPT-5.5**：新版靜默安裝 `coding-agent` skill，以 `openai/gpt-5.5` 為預設，未配置 OpenAI API key 的用戶升級後即遇錯。社群 Issue #73358 已彙整三種緩解方案。（來源：[GitHub Issue #73358](https://github.com/openclaw/openclaw/issues/73358)）

### OpenRouter / 其他

- **OpenRouter 作為統一 Fallback 閘道**：多位用戶分享透過 OpenRouter 將 Gemini、Grok、Mistral 等模型統一接入 OpenClaw 的配置，可有效規避單一 provider 的 rate limit 問題，但需注意 OpenRouter 本身的請求轉發延遲。（來源：[haimaker.ai - Custom Provider Setup](https://haimaker.ai/blog/integrating-custom-llm-providers-with-clawdbot/)）

---

## 3. 社群反應與討論亮點

- **v2026.4.26 Gateway 崩潰引發大量投訴**：piunikaweb.com 整理的用戶反應顯示，此次更新是近期最大規模的穩定性倒退事件，問題涵蓋 Discord plugin 崩潰、npm 套件樹混合新舊版本、macOS 檔案鎖衝突等多個維度。（來源：[piunikaweb.com](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)）
- **Hermes 切換潮**：部分長期 OpenClaw 用戶在社群公開宣布切換至 Hermes，並稱其「更新更平穩、Bug 更少」，此議題在 r/openclaw 獲得高度關注。
- **DeepSeek V4 Flash 成為預設的爭議**：部分開發者認為以穩定性未充分驗證的 DeepSeek 作為 onboarding 預設不妥，建議至少提供 Gemini Flash 或 Claude Haiku 作為備選預設。
- **OpenClaw vs Claude Code 比較討論持續活躍**：[claudefa.st 的對比文章](https://claudefa.st/blog/tools/extensions/openclaw-vs-claude-code) 在社群流傳，引發「重度程式碼場景應用哪套工具更合適」的討論熱潮。
- **`openclaw migrate` 工具好評如潮**：多位從 Hermes 遷移的用戶稱讚新遷移工具操作流暢，但也有用戶反映 MCP server 設定的匯入準確度仍有改善空間。

---

## 4. 值得追蹤的後續議題

1. **v2026.4.26 Gateway 崩潰補丁進度**：官方是否會在近日發布 v2026.4.27 修補版本，以解決 macOS 檔案鎖、Discord plugin 崩潰及 npm 套件樹混合問題，需持續關注。（追蹤：[GitHub Issues](https://github.com/openclaw/openclaw/issues)）

2. **`coding-agent` / GPT-5.5 預設爭議的後續處理**：Issue #73358 的三項緩解方案（opt-in 機制、路由選項、OpenAI key 偵測）何者被採納，將影響非 OpenAI 用戶的升級體驗。

3. **Google Provider Cooldown Bug（Issue #5744）修復時程**：對 Gemini 多模型並行配置的用戶影響持續擴大，官方修復時程尚未公布。

4. **DeepSeek V4 作為預設的穩定性追蹤**：V4 Flash 成為 onboarding 預設後，長期 API 穩定性與 rate limit 行為是否達到生產環境標準，值得持續觀察。

5. **ClawHub 惡意 Skill 審查進展**：前日報告揭露約 12% 的 ClawHub 社群 skill 含惡意代碼，官方審查與清除進展需持續追蹤。

6. **Anthropic OAuth 政策對社群的長期衝擊**：訂閱層封鎖後，API key 模式的費用增幅是否影響 Claude 在 OpenClaw 用戶中的市占率，以及其他 LLM provider 是否因此受益。

---

*資料來源整理：*
- [OpenClaw 官方 GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw v2026.4.26 Release Notes](https://github.com/openclaw/openclaw/releases/tag/v2026.4.26)
- [piunikaweb - v2026.4.26 Gateway Crash Report](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)
- [TechNode - DeepSeek V4 Default Model](https://technode.com/2026/04/27/deepseek-v4-becomes-default-model-for-openclaw/)
- [haimaker.ai - Best Models for OpenClaw April 2026](https://haimaker.ai/blog/best-models-for-clawdbot/)
- [Medium - I Wired OpenClaw to Claude, ChatGPT & Telegram](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)
- [GitHub Issue #73358 - coding-agent gpt-5.5 breaking change](https://github.com/openclaw/openclaw/issues/73358)
- [GitHub Issue #72805 - GitHub Copilot provider model list](https://github.com/openclaw/openclaw/issues/72805)
- [Apidog - ClawSweeper Bot](https://apidog.com/blog/clawsweeper-openclaw-github-triage-bot/)
- [claudefa.st - OpenClaw vs Claude Code](https://claudefa.st/blog/tools/extensions/openclaw-vs-claude-code)
- [haimaker.ai - Custom LLM Provider Setup](https://haimaker.ai/blog/integrating-custom-llm-providers-with-clawdbot/)
