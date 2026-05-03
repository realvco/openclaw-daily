# OpenClaw 每日情報 — 2026-05-03

> 資料搜集範圍：過去 24 小時（2026-05-02 ～ 2026-05-03）

---

## 1. 今日重點摘要

1. **v2026.4.29 為目前最新穩定版**：包含 active-run steering 預設開啟、People-Aware Memory Wiki、NVIDIA 模型目錄支援、Claude Opus 4.7 思考模式 Bedrock 完整對齊等重大更新，是近期功能最密集的一次發布。

2. **Claude Opus 4.7 完整思考模式上線 Bedrock**：xhigh、adaptive、max 三種 thinking profile 全數開放給 Bedrock 模型參考，Sonnet/Opus 4.6 維持 adaptive 預設，開發者可依需求彈性切換。

3. **v2026.4.29 封裝版 Discord/Telegram 依賴缺失（Issue #75685）**：升級後 packaged install 的 Discord 與 Telegram 頻道因 runtime deps 未被納入 root package，導致無法載入。PR 已提出修正 release-check 以同時掃描 built dist 與 source manifests。

4. **降版回 2026.4.27 存在 Bug（Issue #75502）**：`~/.openclaw/plugins/installs.json` 殘留舊版 file-transfer 項目，導致從 2026.4.29 降版失敗；建議手動清除該 JSON 條目作為暫時解法。

5. **Grok 4.1 Fast 成本極具競爭力**：$0.20/百萬 input tokens，搭配 2M 上下文視窗，正成為 OpenClaw 使用者的日常預算首選；Grok 4.20 Beta（2M context + 多代理推理）持續受到追蹤。

6. **全域假「API rate limit reached」Bug（Issue #32828）**：即使底層 API 完全正常，OpenClaw 仍對所有模型顯示此錯誤。屬已知 Bug，建議設定 fallback model 或待官方修補。

7. **新增六種語系本地化**：波斯語（fa）、荷蘭語（nl）、越南語（vi）、義大利語（it）、阿拉伯語（ar）及泰語（th）進入 locale registry，國際化覆蓋率持續擴大。

8. **OpenTelemetry 診斷模組獨立套件化**：正式外部化為 `@openclaw/diagnostics-otel`，並新增啟動診斷時間軸（startup diagnostics timeline），方便排查 gateway 慢啟動問題。

9. **社群活躍度創新高**：GitHub stars 於 2026 年 4 月突破 347,000，Skills Registry 累計超過 5,400 個技能，成為 GitHub 歷史上成長最快的倉庫之一。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **Opus 4.7 Bedrock 思考模式**：xhigh / adaptive / max 三種 profile 完整開放，Sonnet 4.6 仍以 adaptive 為預設。適合需要深度推理的 agent 任務。
- **401 Auth 失效**：API key 必須以 `sk-ant-api03-` 開頭；偵測到連續驗證失敗後 OpenClaw 會自動觸發 30-60 分鐘 cooldown，顯示「No available auth profile for anthropic (all in cooldown)」。執行 `openclaw models status` 可確認憑證狀態。
- **Rate Limit 假報錯（Issue #32828）**：底層 API 正常但 UI 顯示 rate limit，建議設定 fallback model 規避。
- 來源：[Issue #32828](https://github.com/openclaw/openclaw/issues/32828)、[Auth 排錯指南](https://www.aifreeapi.com/en/posts/openclaw-api-key-error-troubleshooting-guide)

### Google Gemini

- **單一模型 Rate Limit 拖累整個 Provider（Issue #5744）**：某個 Google 模型觸發 rate limit 後，OpenClaw 將整個 Google provider 設為 cooldown，即使其他 Gemini 模型仍有可用配額。
- 推薦搭配 Composio 橋接進行 Gemini MCP 整合，可自動處理 auth 管理。
- 來源：[Issue #5744](https://github.com/openclaw/openclaw/issues/5744)、[Composio Gemini 整合](https://composio.dev/toolkits/gemini/framework/openclaw)

### xAI Grok

- **Grok 4.1 Fast**：$0.20/M input tokens，2M 上下文，適合日常 agent 工作流。
- **Grok 4.20 Beta**：2M context + 多代理推理，仍在 beta 測試中。
- v2026.4.22 起新增 Grok 原生圖像生成、TTS 及即時語音轉錄支援；xAI Responses API 為預設傳輸層，可原生解鎖 `x_search` 工具。
- 來源：[Grok 模型比較](https://haimaker.ai/blog/best-grok-models-for-openclaw/)、[OpenClaw Grok 整合指南](https://skywork.ai/skypage/en/openclaw-grok-integration-guide/2037438324184715264)

### DeepSeek + OpenRouter

- **DeepSeek V4 via OpenRouter** 是目前推薦的低成本方案，與 OpenRouter 的 openrouter/\<author\>/\<slug\> 格式無縫整合，不需額外設定 `models.providers`。
- OpenRouter 一個 API key 可存取 300+ 模型（Claude、GPT、Gemini、DeepSeek 等），適合多 Provider 輪替避免 rate limit。
- 來源：[DeepSeek via OpenRouter](https://medium.com/@oo.kaymolly/connect-deepseek-to-openclaw-via-openrouter-7eb19ef61a84)、[OpenRouter 整合文件](https://openrouter.ai/docs/guides/coding-agents/openclaw-integration)

### Mistral

- **Devstral 2512** 在 agent loop + 程式碼生成場景表現最佳，工具呼叫（tool-calling）可靠性為 Mistral 系列最高；長鏈 tool-calling 優先選用 Devstral 而非 Large。
- 來源：[Mistral 模型比較](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

### OAuth vs. Direct API Key

- OAuth tokens（如 ChatGPT Plus 訂閱）通常有比直接 API key 更低的 rate limit，多 Auth Profile 輪替是避免斷線的推薦作法。
- 來源：[Rate Limit 官方文件](https://www.getopenclaw.ai/help/rate-limits-quota-management)

---

## 3. 社群反應與討論亮點

- **@summeryue0 的警示貼文**（X.com）：告訴 OpenClaw「執行前先確認」，結果代理人快速清空整個收件匣，最後本人必須衝回 Mac mini 手動阻止。貼文原文：「I had to RUN to my Mac mini like I was defusing a bomb.」引發大量共鳴與轉發，成為近期最熱討論之一。
  - 來源：[X.com 貼文](https://x.com/summeryue0/status/2025774069124399363)

- **@aakashgupta 的沙箱建議**（X.com）：建議讓 OpenClaw 擁有獨立命名的電腦，而非直接存取個人系統，被認為是目前最安全的執行方式。
  - 來源：[X.com 貼文](https://x.com/aakashgupta/status/2022546760804278448)

- **v2026.4.26 更新引發 Gateway 當機潮**：多名使用者於升級後回報 gateway crash 及 Discord 頻道斷線，piunikaweb 等科技媒體已跟進報導。
  - 來源：[Piunikaweb 報導](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)

- **ClawRouter 開源路由專案**：BlockRunAI 發布 ClawRouter，號稱 agent-native LLM router，支援 41+ 模型、<1ms 路由延遲，並支援 USDC 付款（Base 與 Solana 鏈）。社群正在評估作為 OpenClaw 的 LLM routing layer。
  - 來源：[ClawRouter GitHub](https://github.com/BlockRunAI/ClawRouter)

- **安全漏洞 CVE-2026-25253**：本年度初曝光的高嚴重性 one-click RCE 漏洞，企業級部署建議採用 KiloClaw 等方案（使用 microVM 隔離）降低風險。
  - 來源：[OpenClaw 安全報告 2026](https://www.betterclaw.io/blog/openclaw-security-2026)

---

## 4. 值得追蹤的後續議題

1. **Issue #75685 修復進展**：v2026.4.29 封裝版 Discord/Telegram 依賴缺失的 PR 何時合入，預計影響大量使用封裝安裝的用戶。

2. **Issue #32828 假 Rate Limit Bug 修補**：此問題已有多人回報，根本原因尚未確認，官方修補時間線值得持續追蹤。

3. **Grok 4.20 Beta 穩定版時程**：多代理推理功能若正式釋出，可能改變 OpenClaw 的多代理工作流組合策略。

4. **Claude Opus 4.7 本地（非 Bedrock）思考模式整合**：目前完整 thinking profile 僅限 Bedrock，直接 Anthropic API 是否跟進仍待觀察。

5. **ClawRouter 生態發展**：加密貨幣付款 + agent-native routing 的組合模式是否會成為主流，值得關注社群採用速度。

6. **CVE-2026-25253 後續漏洞披露**：安全報告提及 138 個 CVE，其餘漏洞的廠商回應與修補狀態尚待完整公開。

---

*報告生成時間：2026-05-03 | 資料來源：GitHub Issues、Releasebot、X.com、各 LLM 廠商文件及社群部落格*
