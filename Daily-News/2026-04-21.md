# OpenClaw 每日情報 2026-04-21

> 資料來源：GitHub、X.com、Discord、Reddit、社群部落格｜涵蓋過去 24 小時動態

---

## 1. 今日重點摘要

1. **v2026.4.15 正式版釋出（4/20）**：加入 Google Gemini 文字轉語音（TTS）支援、將 Anthropic 預設模型升級至 Claude Opus 4.7、新增 Model Auth 狀態卡，可即時顯示 OAuth token 健康狀態與 provider rate-limit 壓力，是本月份功能最完整的版本之一。
2. **v2026.4.16-beta.1 當日釋出（4/21）**：修補工具名稱注入（tool name injection）漏洞，攻擊者可透過 client 端定義偽造 tool 名稱，此為安全性緊急修補，強烈建議升級。
3. **Kimi K2.5 連續稱霸社群最佳模型投票**：截至 4/21，Kimi K2.5 在 PricePerToken 社群投票排行榜排名第一，GLM 4.7 第二，Claude Opus 4.6 第三；社群標語 "Kimi 2.5 is the best for OpenClaw" 在 r/openclaw 廣泛流傳。
4. **Anthropic 訂閱封鎖 OpenClaw 的爭議持續發酵**：自 4/4 正式切斷第三方工具訂閱配額後，OpenClaw 創辦人 Peter Steinberger 的帳號於 4/10 被 Anthropic 短暫封禁，引發輿論強烈反應後數小時內恢復。社群持續熱議 Anthropic 的付費政策走向。
5. **Active Memory 外掛進入主線**：v2026.4.10 加入的 Active Memory 外掛（可在主回覆前自動喚起記憶子 agent）在社群獲得廣泛好評，Discord #features 頻道討論量大幅上升。
6. **GitHub Issue #32828：各模型回傳假 rate limit 錯誤**：多名用戶回報即使 API 完全正常，OpenClaw 仍顯示「API rate limit reached」，已確認為 OpenClaw 本身的 bug 而非 provider 端問題，尚未正式修復。
7. **Google Gemini TTS 正式整合**：v2026.4.15 新增 Gemini TTS 支援，包含語音選擇、WAV 音訊輸出與 PCM 電話語音輸出，是繼 macOS 本地 MLX 語音後的又一重大語音里程碑。
8. **安全性：本月累計修補 13 個 CVE**：包含 CVSS 8.7 的特權提升（CVE-2026-35639）與 CVSS 8.4 的任意程式碼執行（CVE-2026-35641），已於 4/9–4/10 釋出的安全性更新中修補。
9. **Discord 社群突破 17 萬人**：「Friends of the Crustacean 🦞🤝」Discord 伺服器成員數達 172,648，顯示 OpenClaw 使用者基數持續快速成長。

---

## 2. LLM 串接實戰回報

### Anthropic（Claude 系列）

- **訂閱配額正式切斷（自 4/4 起）**：Anthropic 停止向第三方工具（包含 OpenClaw）提供訂閱 quota。使用 `sk-ant-oat-*`（OAuth token）的用戶將收到 401/429 錯誤，**必須改用 API Key（`sk-ant-api03-*`）**。
  - 來源：[Anthropic cuts off OpenClaw third-party use – VentureBeat](https://venturebeat.com/technology/anthropic-cuts-off-the-ability-to-use-claude-subscriptions-with-openclaw-and)
  - 來源：[Anthropic closes door on subscription use of OpenClaw – The Register](https://www.theregister.com/2026/04/06/anthropic_closes_door_on_subscription/)

- **Claude Opus 4.7 成為新預設**：v2026.4.15 將 Anthropic 所有預設選項、opus 別名及 CLI 預設全面切換至 Claude Opus 4.7，用戶升級後若有 prompt 格式依賴需重新測試。
  - 來源：[OpenClaw 2026.4.15 Release – OpenClaw Playbook Blog](https://www.openclawplaybook.ai/blog/openclaw-2026-4-15-release-google-tts-model-auth-trust/)

- **OpenClaw 創辦人遭 Anthropic 短暫封禁（4/10）**：Peter Steinberger 在 X 發文後帳號被暫停，Anthropic 稱因「可疑活動」，數小時後在社群壓力下恢復。事件背景與 Steinberger 於 2/14 宣布加入 OpenAI 相關，引發開源社群對廠商生態依賴的廣泛討論。
  - 來源：[TechCrunch – Anthropic temporarily banned OpenClaw's creator](https://techcrunch.com/2026/04/10/anthropic-temporarily-banned-openclaws-creator-from-accessing-claude/)
  - 來源：[Medium – The OpenClaw Ban That Exposed Anthropic's Real Problem](https://medium.com/@stawils/the-openclaw-ban-that-exposed-anthropics-real-problem-fe8f10aa0e80)

### OpenAI（GPT 系列）

- **GPT-5.4-pro 前向相容支援**：v2026.4.14 加入 GPT-5.4-pro 的 Codex 定價與限額查詢，上游目錄尚未更新前即可正常顯示狀態與計費資訊。
  - 來源：[OpenClaw 2026.4.14 Release Analysis – Blockchain News](https://blockchain.news/ainews/openclaw-2026-4-14-release-smarter-gpt-5-4-routing-chrome-cdp-upgrades-and-messaging-fixes-reliability-analysis)

- **Codex transport 自我修復機制**：v2026.4.15 新增 Codex 傳輸層自動重連，解決 Codex 工作流中偶發的連線中斷問題。

### Google（Gemini 系列）

- **Gemini TTS 正式上線**：v2026.4.15 整合 Gemini 文字轉語音，支援多種語音、WAV 及 PCM 輸出格式，設定文件同步更新。
  - 來源：[Release openclaw 2026.4.15 – GitHub](https://github.com/openclaw/openclaw/releases/tag/v2026.4.15)

- **Gemini 2.5 Pro/Flash 穩定支援**：社群反饋 Gemini 2.5 Pro 在複雜 multi-step tool-calling 中表現穩定，與 Claude 相當，價格更具競爭力。
  - 來源：[Best Models for OpenClaw April 2026 – haimaker.ai](https://haimaker.ai/blog/best-models-for-clawdbot/)

### Moonshot（Kimi K2.5）

- **社群票選最佳模型**：Kimi K2.5 在 PricePerToken 社群投票中排名第一，以免費方案即可使用頂級推理能力，成為低預算用戶首選。
  - 來源：[Best Model for OpenClaw – PricePerToken](https://pricepertoken.com/leaderboards/openclaw)
  - 來源：[Kimi K2.5 Free on OpenClaw – Vertu](https://vertu.com/ai-tools/openclaw-drops-bombshell-kimi-k2-5-becomes-first-free-premium-model/)

- **Kimi + OpenClaw 教學熱潮**：多篇教學文「Kimi 2.5 is Better Than Claude 4.5 for Free?」在社群廣泛分享，API 設定採 OpenAI 相容格式，設定難度低。

### DeepSeek

- **V3.2 廣受推薦，但複雜 tool chain 仍不穩**：DeepSeek V3.2（128K context，$0.28/M input / $0.40/M output）性價比高，但在多步驟 tool-calling 中偶發格式錯誤或遺漏參數，建議搭配 Claude 或 Gemini 作為主力。
  - 來源：[Best DeepSeek Models for OpenClaw – haimaker.ai](https://haimaker.ai/blog/best-deepseek-models-for-openclaw/)
  - 來源：[OpenClaw Best Model Selection Guide – LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-best-model-selection-guide)

### Rate Limit / Auth 通用踩坑

- **假 rate limit 錯誤（Issue #32828）**：OpenClaw 對所有模型回報「API rate limit reached」，但 API 本身正常運作。問題源於 OpenClaw 內部狀態判斷錯誤，非 provider 端問題。暫時解法：重啟 OpenClaw 或清除 session。
  - 來源：[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828)

- **Model Auth 狀態卡（v2026.4.15 新功能）**：新增 OAuth token 到期提醒與 rate-limit 壓力指示，改善用戶對認證狀態的可見性，降低因 token 過期導致的靜默失敗。
  - 來源：[OpenClaw 2026.4.15 Release – OpenClaw Playbook Blog](https://www.openclawplaybook.ai/blog/openclaw-2026-4-15-release-google-tts-model-auth-trust/)

---

## 3. 社群反應與討論亮點

### Discord（172,648 成員）
- **#models 頻道熱議**：Kimi K2.5 vs Claude Opus 4.7 成為本週最熱話題，多數用戶建議以 Kimi K2.5 作為日常任務主力、Claude 處理需要高安全性的情境。
- **Active Memory 外掛使用心得**：多名進階用戶分享 Active Memory 的 `/verbose` 指令用法，並反饋可設定 `message/recent/full` 三種 context 模式，靈活性高。
- **Gemini TTS 首批試用回報**：v2026.4.15 釋出後數小時內，Discord 語音相關頻道已出現多則測試心得，整體評價正面，但部分用戶反映 PCM 電話音質仍有待優化。

### Reddit（r/openclaw，103,000+ 成員）
- **「Kimi 2.5 is the best for OpenClaw」成為社群共識**：多篇高讚貼文比較各模型，Kimi K2.5 憑藉免費方案與穩定工具呼叫能力獲得廣泛認可。
  - 來源：[Kimi K2.5 Free on OpenClaw – Vertu](https://vertu.com/ai-tools/openclaw-drops-bombshell-kimi-k2-5-becomes-first-free-premium-model/)
- **Anthropic 訂閱封鎖引發生態依賴討論**：討論聚焦「不應過度依賴單一 LLM provider」，呼籲 OpenClaw 強化 multi-provider fallback 機制。

### X.com（前 Twitter）
- **Ryan Carson 展示 OpenClaw 自我更新**（[@ryancarson](https://x.com/ryancarson/status/2019059775573540981)）：分享 OpenClaw 自行安裝新版本（從 2026.2.1 升至 2026.2.2-3）的截圖，引發大量轉推，凸顯 OpenClaw 代理自主性的特色。
- **Peter Steinberger 封禁事件持續發酵**：X 上對 Anthropic 的批評聲浪未減，多個科技帳號分析 Anthropic 政策轉向對開源生態的衝擊。
  - 來源：[@openclaw 官方 X 帳號](https://x.com/openclaw)

### Hacker News
- **「Tell HN: Anthropic no longer allowing Claude Code subscriptions to use OpenClaw」**：貼文引發熱烈討論，社群對 Anthropic 的商業模式轉向看法分歧，部分人認為此舉合理，另一部分則認為傷害了開源生態。
  - 來源：[Hacker News #47633396](https://news.ycombinator.com/item?id=47633396)

---

## 4. 值得追蹤的後續議題

1. **v2026.4.16 正式版釋出時程**：beta.1 已於 4/21 釋出，工具名稱注入漏洞的正式修補版本預計近日跟進，建議持續關注 GitHub Releases。
   - 追蹤：[Releases · openclaw/openclaw](https://github.com/openclaw/openclaw/releases)

2. **Anthropic 訂閱政策長期走向**：Anthropic Pro/Max 訂閱用戶是否會有其他官方方案整合 OpenClaw，或需完全轉向 API Key 計費？Peter Steinberger 加入 OpenAI 後的 OpenClaw 開發方向值得密切追蹤。
   - 追蹤：[TechCrunch – Anthropic 封禁事件](https://techcrunch.com/2026/04/10/anthropic-temporarily-banned-openclaws-creator-from-accessing-claude/)

3. **Issue #32828 假 rate limit 錯誤修復**：此問題影響多個 provider 的正常使用，官方尚未提供 ETA，社群期待熱修補（hotfix）儘早釋出。
   - 追蹤：[GitHub Issue #32828](https://github.com/openclaw/openclaw/issues/32828)

4. **Kimi K2.5 在複雜 agent 工作流的穩定性驗證**：Kimi K2.5 雖然在社群投票中領先，但長上下文（>100K tokens）與高頻 tool-calling 場景的穩定性資料仍不完整，值得持續收集社群實測數據。
   - 追蹤：[PricePerToken OpenClaw Leaderboard](https://pricepertoken.com/leaderboards/openclaw)

5. **Active Memory 外掛成熟度追蹤**：此外掛仍屬可選（opt-in），官方尚在收集實際使用回饋。`full context` 模式對 token 消耗的影響、與不同 LLM provider 的相容性仍需社群進一步驗證。

6. **安全性修補後的漏洞披露細節**：CVE-2026-35639（CVSS 8.7）與 CVE-2026-35641（CVSS 8.4）的技術細節尚未完整公開，預計於修補後的適當時機披露，建議訂閱 GitHub Security Advisories。
   - 追蹤：[OpenClaw Security Patch – Blink Blog](https://blink.new/blog/openclaw-april-2026-new-cves-security-patch-guide)

---

*報告產生時間：2026-04-21 | 資料涵蓋範圍：過去 24 小時*
