# OpenClaw 每日情報 — 2026-04-10

> 資料來源涵蓋過去 24 小時（2026-04-09 ～ 2026-04-10）

---

## 一、今日重點摘要

1. **v2026.4.9 正式釋出（4/9）**：本次版本由 21 位貢獻者共同完成，重點為 Memory/Dreams 強化、Control UI 日記視圖、安全性修補，以及 Plugin SDK 的 `providerAuthAliases` 支援。
2. **REM 記憶回填機制上線**：新增 `rem-harness --path` 歷史回填通道，可將舊日記（diary）重新播放進 Dreams 及持久記憶，並提供時間軸導覽與 commit/reset 流程。
3. **安全性強化**：瀏覽器 SSRF 隔離在互動導覽後重新執行安全檢查；封鎖來自不信任 workspace `.env` 的 runtime-control 環境變數；Remote node 事件在入隊前先進行消毒，防止可信內容注入。
4. **Anthropic 封鎖 Claude 訂閱用於 OpenClaw（4/4 起生效）**：Claude Pro/Max 訂閱者已無法透過訂閱額度使用 OpenClaw，必須另行購買用量套餐或切換其他 LLM。Anthropic 提供一次性等額訂閱費信用點，兌換期限至 4/17。
5. **社群強烈反彈**：r/ClaudeAI 及 r/AI_Agents 上出現大量討論，OpenClaw 創辦人 Peter Steinberger 公開批評 Anthropic「複製熱門功能進封閉生態後鎖死開源整合」。Hacker News 亦出現熱門討論串。
6. **Plugin SDK 分離輕量路徑**：`openclaw/plugin-sdk/command-status` 拆分為獨立子路徑，避免 auth-only 插件引入多餘的 status/context warmup 依賴。
7. **iOS CalVer 版號鎖定**：iOS 版本號改採明確的 `apps/ios/version.json` CalVer 管理，TestFlight 迭代使用短版號，直到 maintainer 正式晉升 gateway 版本。
8. **4 月活躍 Issues**：4/9 單日開出 #63937、#63936、#63935、#63933、#63931 等多個 Issue，另有 #63922 被標記為 bug + regression。
9. **Gemini 成為免費層首選**：社群評比中，Google Gemini 以每分鐘 60 次請求、每日 1,000 次免費額度，成為 2026 年常駐 AI agent 最具性價比的選擇。

---

## 二、LLM 串接實戰回報

### Claude（Anthropic）

- **訂閱封鎖衝擊**：4/4 起 Claude Pro（$20）和 Max（$200）訂閱額度不再涵蓋 OpenClaw 使用量。受影響用戶最高面臨 50 倍的費用增長。
  - 過渡方案：購買 Anthropic Usage Bundle（目前有折扣）或切換至其他 LLM。
  - 信用點兌換截止：**4/17**。
  - 來源：[TechCrunch](https://techcrunch.com/2026/04/04/anthropic-says-claude-code-subscribers-will-need-to-pay-extra-for-openclaw-support/) ｜ [TechRadar](https://www.techradar.com/pro/bad-news-claude-users-anthropic-says-youll-need-to-pay-to-use-openclaw-now) ｜ [InfoWorld](https://www.infoworld.com/article/4154435/anthropic-cuts-openclaw-access-from-claude-subscriptions-offers-credits-to-ease-transition.html)

- **已知 Bug**：虛假 provider cooldown（[Issue #23996](https://github.com/openclaw/openclaw/issues/23996)）——OAuth token auth 觸發 hardcoded `rate_limit` 原因，導致無限冷卻，完全阻斷所有 agent 回覆，無自動恢復機制。

### Gemini（Google）

- 免費開發者額度（60 rpm / 1,000 req/day）使其成為「always-on agent」最佳免費選擇。
- 社群建議搭配 Claude Haiku 或 GPT-5 Mini 作為穩定備援。
- 來源：[haimaker.ai](https://haimaker.ai/blog/best-models-for-clawdbot/) ｜ [similarlabs.com](https://similarlabs.com/blog/best-models-for-openclaw)

### DeepSeek

- **V3.2 實戰評比**：在社群 LLM 排行榜（成功率、速度、費用綜合）中位列第 15。SWE-bench 得分約 60%（Claude Opus 89%），但每 token 成本約為 Claude Opus 的 1/90。
- **API 可靠性問題**：尖峰時段頻繁出現 503 錯誤、time-to-first-token 偏長。建議搭配 Gemini Flash、Claude Haiku 或 GPT-5 Mini 作穩定性保底。
- 來源：[haimaker.ai DeepSeek 評比](https://haimaker.ai/blog/best-deepseek-models-for-openclaw/)

### OpenRouter

- 單一端點整合 290+ 模型，有效規避單一廠商鎖定風險。
- 社群專案 [ClawRouter](https://github.com/BlockRunAI/ClawRouter)（BlockRunAI）提供 agent-native LLM 路由，支援 41+ 模型，延遲 < 1ms，並支援 Base 及 Solana 的 USDC 支付（x402 協議）。
- [Issue #63416](https://github.com/openclaw/openclaw/pulls) 修復了 OpenRouter catalog 模型的 provider-qualified reference 保留問題。
- 來源：[skywork.ai OpenRouter 設定指南](https://skywork.ai/skypage/en/openclaw-openrouter-setup/2037020692520374272)

### Mistral

- 123B agentic coding 模型支援多檔案協調、框架感知及錯誤回復，適合複雜代理工作流。
- 來源：[dev.to LLM Router 比較](https://dev.to/sophiaashi/6-best-llm-routers-for-openclaw-in-2026-17oa)

### 通用踩坑紀錄

| 問題類型 | 說明 | 參考 Issue |
|---|---|---|
| 假性 rate limit 冷卻 | 未實際觸發 API 限制卻雙 provider 同時進入冷卻 | [#32828](https://github.com/openclaw/openclaw/issues/32828) |
| WebSocket 重連遺失 auth token | Control UI webchat 重連後觸發 rate limiter | [#28997](https://github.com/openclaw/openclaw/issues/28997) |
| 速率限制多維觸發 | 429 同時適用 RPM、輸入 token/min、輸出 token/min，任一觸發即中斷 | [#48603](https://github.com/openclaw/openclaw/issues/48603) |
| Gateway token 失效 | 解法：`openclaw models auth setup-token` 重新生成 | [LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error) |

---

## 三、社群反應與討論亮點

### Hacker News

- 討論串「Tell HN: Anthropic no longer allowing Claude Code subscriptions to use OpenClaw」引發大量回覆，社群對 Anthropic 的商業策略持批評態度。
  - 來源：[Hacker News #47633396](https://news.ycombinator.com/item?id=47633396)

### Reddit

- **r/ClaudeAI**：數百則留言，主要情緒為不滿——用戶認為此舉破壞了既有的開放生態承諾。
- **r/AI_Agents**：討論此變化是否代表 AI agent 框架將被主要 LLM 廠商逐步封閉的產業趨勢。

### OpenClaw 創辦人公開批評

> Peter Steinberger（OpenClaw 創辦人）：「Anthropic 把熱門功能複製進自家封閉 harness，然後鎖死開源整合。我和 Dave Morin 都試著跟 Anthropic 溝通，最多只爭取到延後一週。」

- 來源：[XDA Developers](https://www.xda-developers.com/claude-subscribers-just-lost-access-to-openclaw-and-other-third-party-toolsunless-they-pay-more/) ｜ [RoboRhythms](https://www.roborhythms.com/anthropic-blocks-openclaw-claude-subscription-2026/)

### Discord（官方）

- Discord 社群規模：168,508 成員。
- 版本 2026.4.7-1 發布後的討論集中在新的影音生成功能、Discord 綁定優化（跨 DM、元件互動、原生指令的身份統一）。
- 來源：[Discord 官方文件](https://docs.openclaw.ai/channels/discord) ｜ [dev.to 設定指南](https://dev.to/bdougieyo/setting-up-openclaw-on-exedev-with-discord-27n)

---

## 四、值得追蹤的後續議題

1. **Claude 信用點兌換期限（4/17）**：尚未兌換的用戶只剩一週，需決定是否繼續使用 Claude API 或遷移至其他 LLM。屆時用戶遷移潮的規模值得觀察。

2. **OpenClaw 與 Claude Code 的競合關係**：Anthropic 持續強化 Claude Code CLI 的 agent 功能，與 OpenClaw 的功能重疊日益明顯。社群正在關注二者的邊界如何演化。
   - 參考：[Blink Blog 比較文](https://blink.new/blog/openclaw-vs-claude-code-comparison-2026)

3. **DeepSeek API 穩定性改善進度**：目前 503 錯誤影響生產環境可用性，是否有官方 SLA 或改善公告值得持續追蹤。

4. **Plugin SDK `providerAuthAliases` 生態適配**：新的 provider 認證別名機制若被廣泛採用，可能大幅簡化多 provider 混用的設定，值得關注生態系的適配速度。

5. **REM Dreams 記憶回填的實際效果評估**：v2026.4.9 的記憶系統大幅重構，社群尚未有足夠時間驗證歷史日記回放的品質與可靠性。

6. **iOS 版本正式上架進度**：CalVer 管理機制到位後，正式版何時從 TestFlight 晉升至 App Store 值得追蹤。

7. **OpenRouter v/s ClawRouter 路由生態競爭**：第三方 agent-native router（如 ClawRouter）與 OpenRouter 的整合方案在成本與延遲上的競爭，可能影響未來 OpenClaw 的預設推薦 provider。

---

*報告生成時間：2026-04-10 | 資料截止：2026-04-10 UTC 00:00*

*主要資料來源：[GitHub openclaw/openclaw](https://github.com/openclaw/openclaw) ｜ [Hacker News](https://news.ycombinator.com/item?id=47633396) ｜ [TechCrunch](https://techcrunch.com/2026/04/04/anthropic-says-claude-code-subscribers-will-need-to-pay-extra-for-openclaw-support/) ｜ [newreleases.io v2026.4.9](https://newreleases.io/project/github/openclaw/openclaw/release/v2026.4.9)*
