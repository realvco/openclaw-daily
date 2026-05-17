# OpenClaw 每日情報 2026-05-11

> 資料涵蓋範圍：過去 24 小時（2026-05-10 ～ 2026-05-11）

---

## 1. 今日重點摘要

1. **v2026.5.7-beta.1 釋出**：最新 beta 版帶來 chat 指令 `/think default`、`/fast default` 清除 session 覆寫設定、Codex 審核流程改善、Dreaming 記憶體預設改為獨立檔案、Google Meet 與即時語音整合持續強化。
2. **GitHub Issues 爆量（May 10–11）**：單日新增超過 12 個 Issues，集中在 LM Studio Provider 環境變數展開失敗（#80495）、per-model `api` override 路徑未解析（#80487）、Realtime voice provider openai 未登錄啟動錯誤（#80483），以及 `openclaw doctor` 每次更新覆寫自訂 systemd unit（#80462）。
3. **OpenAI Responses API 跨 provider failover 故障**：PR #80452 記錄了跨 provider 切換時 400 錯誤「'message' missing required 'reasoning' item」；相關的 #80451 回報 failover ladder 在同一 billing account 下重試多個 Gemini 模型，account-level quota 耗盡後仍持續重試。
4. **模型路由衝突**：Issue #79325 確認當 opencode-go/deepseek-v4-pro 與 openrouter/deepseek/deepseek-v4-pro 同時存在於 catalog 時，model picker 會錯誤解析至 OpenRouter 版本；DeepSeek 預設 4096 token context window 問題依然是新手常踩的坑。
5. **`openclaw doctor --fix` 漏洞**：#80490 指出 doctor --fix 不會自動 build 尚未編譯的 plugins，反而觸發 validator 錯誤，誤導使用者以為是設定問題。
6. **社群規模突破**：「Friends of the Crustacean」官方 Discord 伺服器成員已突破 176,000，成為 OpenClaw 生態系最大即時支援平台。
7. **SWAT 多 agent 框架走紅**：來自 GitHub Discussion #41068 的純 Markdown 驅動 squad 協作系統（MANIFEST.md + OPERATION.md + INTEL.md）在社群中廣泛傳播，不需自訂程式碼即可實現 multi-agent 任務分工。
8. **Cron 訊息不可見問題**：#80468 回報 cron 排程送出的訊息在 channel session context 中無法顯示，影響依賴排程自動化的工作流。
9. **Plugin SDK 安全性強化**：before_tool_call 與 trusted policy hooks 現在可取得 host-derived tool target paths，讓 workflow plugins 能在不重新解析 tool envelopes 的前提下驗證已知檔案目標。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **OAuth 觸發假 rate limit**：多名使用者在 Discord 回報，使用 Claude OAuth 登入後，OpenClaw 顯示「API rate limit reached. Please try again later.」，但直接呼叫 Anthropic API 完全正常。已知 Issue #32828、#23996。根本原因為 hardcoded `rate_limit` reason 在 OAuth token 驗證時觸發無限 cooldown。
  - 來源：[AnswerOverflow - Claude OAuth rate limit](https://www.answeroverflow.com/m/1478788645472436264)
  - 來源：[Bug #23996 - False provider cooldown](https://github.com/openclaw/openclaw/issues/23996)
- **Claude Opus 4.7 表現最穩定**：在長 context 任務、prompt-injection 防護、多步驟 tool use 三個維度均優於 GPT-5.5，SWE-bench Pro 達 64.3%。
  - 來源：[DEV Community - Best LLM for OpenClaw 2026](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)

### GPT（OpenAI）

- **跨 provider failover 格式錯誤**：切換到非 OpenAI provider 後再 failover 回 OpenAI Responses API，會因為 message 缺少必要 `reasoning` 欄位而收到 HTTP 400。
  - 來源：[Issue #80452](https://github.com/openclaw/openclaw/issues)
- **GPT-5.5 agent 場景優勢**：在 context ≤ 128K tokens 的自主 agent 場景中，GPT-5.5 在 Terminal-Bench 2.0 排名居首。
  - 來源：[PricePerToken - Best Model for OpenClaw](https://pricepertoken.com/leaderboards/openclaw)

### Gemini（Google）

- **同帳號 quota failover 問題**：#80451 指出 failover ladder 在 Gemini 3 系列多個模型間重試時，若 quota 為帳號層級（account-scoped），所有模型同時被限速，failover 無法真正緩解問題。
  - 來源：[Issue #80451](https://github.com/openclaw/openclaw/issues)
- **tool-call thought-signature replay**：v2026.5.7 修復了 Gemini 3 tool-call thought-signature 重播問題，加入 fallback signatures 確保 transcript 完整性。
  - 來源：[newreleases.io - v2026.5.7](https://newreleases.io/project/github/openclaw/openclaw/release/v2026.5.7)

### DeepSeek

- **預設 context window 太小**：OpenClaw 對 custom provider 預設設定 4096 token limit，DeepSeek 需手動修改 Web UI Raw JSON 設定將 `contextWindow` 改為 131072。
  - 來源：[LaoZhang AI Blog - OpenClaw LLM Setup](https://blog.laozhang.ai/en/posts/openclaw-llm-setup)
- **OpenRouter 路由衝突**：若同時設定 opencode-go 和 openrouter 兩條 DeepSeek V4 Pro 路由，model picker 會隨機選到 OpenRouter 版本。
  - 來源：[Issue #79325](https://github.com/openclaw/openclaw/issues/79325)

### OpenRouter

- **內建支援，設定簡單**：只需設定 API key，以 `openrouter/<author>/<slug>` 格式指定模型即可，不需額外設定 `models.providers`。
  - 來源：[OpenRouter 官方文件 - OpenClaw Integration](https://openrouter.ai/docs/guides/guides/openclaw-integration)

### 通用 Rate Limit 防範建議

- 設定 fallback model，避免單一 provider 被限速時工作流中斷。
- 執行 `openclaw doctor --fix` 可自動修復大部分 401/429/gateway token 錯誤。
- Anthropic Tier 1 需帳號有至少 $5 消費紀錄（2026 年 2 月起生效）。
  - 來源：[LaoZhang AI Blog - Fix Every OpenClaw Error](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)

---

## 3. 社群反應與討論亮點

- **Discord 問題排查最即時**：2026.4.26 更新後觸發 gateway crash 及 Discord/Telegram 無回應問題，社群在 Discord「Friends of the Crustacean」伺服器中迅速彙整 workaround，顯示 Discord 已成為第一線技術支援場所。
  - 來源：[PiunikaWeb - 2026.4.26 update crashes](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)

- **SWAT multi-agent 框架熱議**：GitHub Discussion #41068 的純 Markdown squad 協調方案（無需程式碼）正在社群中廣泛擴散，被視為 OpenClaw 多 agent 模式的「最低成本起點」。

- **NVIDIA Blog 背書 OpenClaw**：NVIDIA Nemotron Labs 發表文章探討 OpenClaw Agents 對企業的意義，顯示 OpenClaw 已引起大廠關注。
  - 來源：[NVIDIA Blog - What OpenClaw Agents Mean for Every Organization](https://blogs.nvidia.com/blog/what-openclaw-agents-mean-for-every-organization/)

- **Realtime Voice 成下一個熱點**：社群討論 OpenClaw 的 Google Meet 整合與即時語音功能，認為「下一代 agent 不只是回訊息，而是能加入會議、聆聽、發言、輸出成果物」。

- **`openclaw doctor` 信任問題**：#80462 與 #80490 的出現讓社群開始質疑 `doctor --fix` 的可靠性——連自動修復工具本身都有 bug，讓不少人改採手動驗證設定。

- **OpenClaw 作為 autonomous coding agent 的使用案例增加**：Blink Blog 發文介紹「讓 OpenClaw 在你睡覺時修 bug 並開 PR」，引發廣泛轉發與討論。
  - 來源：[Blink Blog - OpenClaw as a Coding Agent](https://blink.new/blog/openclaw-autonomous-coding-agent)

---

## 4. 值得追蹤的後續議題

1. **#80487 per-model `api` override 路徑問題**：影響所有使用自訂 API endpoint 的用戶，預計下一個 patch 版本（v2026.5.8 或 beta.2）優先修復。

2. **Gemini account-scoped failover 設計缺陷（#80451）**：現有 failover ladder 設計對 account-level quota 束手無策，期待 OpenClaw 引入 provider-level quota awareness。

3. **OpenAI Responses API `reasoning` 欄位相容性（#80452）**：跨 provider failover 的格式驗證問題，需要雙邊（OpenClaw + OpenAI）的修復。

4. **`openclaw doctor` 可靠性提升**：#80490 暴露 doctor 工具對未編譯 plugin 的處理缺陷，社群期待更嚴謹的診斷流程。

5. **Realtime Voice 與 Google Meet 整合的穩定性**：#80483 的 voice provider 載入順序 bug 顯示這塊功能尚未成熟，值得持續關注每日 beta 版進展。

6. **LM Studio 本地部署支援（#80495）**：環境變數展開與 API endpoint 不匹配問題，對使用本地 LLM 的用戶影響較大，預期會有更多相關 issue 跟進。

7. **多 agent SWAT 框架的官方文件化**：社群驅動的方法論持續在 GitHub Discussion 中演進，觀察 OpenClaw 團隊是否會官方採納或提供標準化支援。

---

*報告產製時間：2026-05-11 | 資料來源：GitHub Issues/Releases、Discord 社群、搜尋引擎即時結果*
