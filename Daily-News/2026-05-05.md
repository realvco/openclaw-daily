# OpenClaw 每日情報 — 2026-05-05

> 資料搜集範圍：過去 24 小時（2026-05-04 ～ 2026-05-05）

---

## 1. 今日重點摘要

1. **v2026.5.3 正式發布，內建檔案傳輸插件上線**：新增 `file_fetch`、`dir_list`、`dir_fetch`、`file_write` 四項 agent 工具，支援配對節點間的二進位檔案操作，是自架伺服器與 homelab 用戶期待已久的重大功能。

2. **CLI 訊息子程序錯誤退出修復**：v2026.5.3 修復了插件 registry 載入失敗時，`openclaw-message` 子程序持續存活的問題；現在失敗時會以非零狀態乾淨退出，避免殭屍程序堆積。

3. **插件路徑安全檢查報告改善**：設定中存在但被路徑安全檢查攔截的插件，現在會正確顯示為「blocked」而非誤導性的「stale plugin not found」，排錯體驗明顯提升。

4. **macOS LaunchAgent 自動修復機制上線**：`gateway/update` 現在可在套件更新後偵測到「已安裝但未載入」的 macOS LaunchAgent，並自動重跑健康檢查，減少 macOS 使用者手動干預的頻率。

5. **Kimi K2.5 蟬聯社群投票第一名**：截至 2026-05-04，pricepertoken.com 社群排行榜 Kimi K2.5 仍居首位，其後依序為 GLM 4.7 與 Claude Opus 4.6；Kimi K2.6 在 SWE-Bench Pro 以 58.6% 超越 GPT-5.4（57.7%），但社群採用尚在觀望期。

6. **Anthropic 訂閱方案封鎖第三方工具滿一個月**：自 2026-04-04 正式執行以來，Claude Pro/Max 訂閱無法再用於 OpenClaw 的影響已全面發酵；使用者必須改用 API Key（pay-as-you-go），TechCrunch 等媒體持續跟進後續影響。

7. **OpenClaw 主倉庫突破 368K GitHub Stars**：主倉庫目前有 368,311 stars、75,853 forks，open issues 3,410 筆、open PR 3,428 筆，生態規模持續擴大。

8. **Grok 4.3 成為 xAI 預設模型，Grok 4.20 Beta 仍在測試**：v2026.5.2 已將 Grok 4.3 設為 bundled catalog 的 xAI 預設；Grok 4.20 Beta（2M context + 多代理推理）正式版尚未釋出，社群持續關注。

9. **Discord「Friends of the Crustacean」社群成員逼近 17 萬人**：Discord 社群規模持續成長，#models 與 #help 頻道為最活躍討論區，成為 LLM 選型與踩坑回報的第一手情報站。

---

## 2. LLM 串接實戰回報

### Claude（Anthropic）

- **訂閱封鎖後的 Auth 設定變化**：自 2026-04-04 起，所有透過 OAuth token（Claude Pro/Max 訂閱）存取 OpenClaw 的方式全數失效，必須改用 `sk-ant-api03-` 格式的 Anthropic API Key，OpenClaw 設定路徑為 `Settings → Model Providers → Anthropic → API Key`。
- **Claude Sonnet 4.6（$3/$15 per MTok）**為目前 OpenClaw 推薦的 Claude 預設模型，智能與成本平衡最佳；Opus 4.7 定價調降後，重度用戶已開始升級為日常主力。
- **假 rate limit 錯誤（Issue #32828）持續未修**：底層 API 完全正常的情況下，OpenClaw 仍會對所有 provider（含 Claude）顯示 rate limit 警告，官方尚未給出修補時程；建議設定 fallback model（例如 Haiku 4.5）作為緩解措施。
- **`tool_use.input` 欄位缺失**：出現 `LLM request rejected: messages.content.tool_use.input field required` 時，表示 session history 損毀，執行 `/new` 開啟新 session 可解決。
- 來源：[Anthropic 封鎖 OpenClaw 第三方工具報導（TechCrunch）](https://techcrunch.com/2026/04/04/anthropic-says-claude-code-subscribers-will-need-to-pay-extra-for-openclaw-support/)、[API Key 設定指南](https://www.shareuhack.com/en/posts/openclaw-setup-tutorial-2026)、[Issue #32828](https://github.com/openclaw/openclaw/issues/13615)

### Kimi（Moonshot）

- **K2.5 社群票選第一，K2.6 benchmark 更強但採用尚觀望**：Kimi K2.6 在 SWE-Bench Pro 達 58.6%，超越 GPT-5.4（57.7%），適合多步驟長鏈任務與中英雙語 agent 場景；K2.5 仍因長期穩定性獲社群信賴，兩者間的切換觀察持續進行中。
- 來源：[pricepertoken.com OpenClaw 排行](https://pricepertoken.com/leaderboards/openclaw)、[Best Models for OpenClaw April 2026](https://haimaker.ai/blog/best-models-for-clawdbot/)

### GPT（OpenAI / ChatGPT Pro）

- **Codex + /goal 指令**：v2026.5.2 引入的 `/goal` 指令需搭配 ChatGPT Pro 訂閱及 Codex plugin，可描述高層目標讓 OpenClaw 跨多步驟自主執行，是目前最長鏈 agentic 能力之一。
- **Rate Limit 處理**：OpenClaw 內建的指數退避（exponential backoff）有已知 bug，建議在 OpenAI provider 設定中加入 `fallback_model` 作為緩衝。
- 來源：[OpenClaw 2026.5.2 Release](https://www.buildfastwithai.com/blogs/openclaw-2026-5-2-release)、[Rate Limit 修復指南](https://evolink.ai/blog/openclaw-fix-429-rate-limit-errors)

### Google Gemini

- **Gemini 2.5 Pro token 消耗異常高**：社群回報 Gemini 2.5 Pro 在數十次 API 呼叫內即消耗 190 萬+ 輸入 token，主因是龐大的 `config.schema`（396KB+ JSON）被寫入 session `.jsonl` 後，後續每則訊息都拖著完整 blob 前進，造成指數型 token 成長。
- **單一模型 rate limit 拖累整個 Google Provider（Issue #5744）**：Gemini 某模型觸發 rate limit 後，OpenClaw 將整個 Google provider 置入 cooldown，其他仍有配額的 Gemini 模型也一同受影響，問題仍未修復。
- **免費額度提醒**：Gemini 2.5 Flash 仍提供業界最慷慨的免費 tier 之一，適合低流量、高延遲容忍的 cron 任務。
- 來源：[OpenClaw Token 使用文件](https://docs.openclaw.ai/reference/token-use)、[Issue #5744 相關討論](https://www.aifreeapi.com/en/posts/openclaw-api-key-error-troubleshooting-guide)

### DeepSeek

- **DeepSeek V3（V4 Flash 等系列）**：成本最低的雲端高品質選項，輸入費用約 $0.20/M，約為 Kimi K2 的 2.5 倍便宜；重度 coding 場景推薦 DeepSeek V4 Pro，短問答推薦 V4 Flash。
- **透過 OpenRouter 路由建議**：單一 API key 覆蓋 300+ 模型，可在多家 provider 間自動輪替，有效規避單一 provider rate limit。
- 來源：[Free LLM APIs for OpenClaw 2026](https://clawhosters.com/blog/posts/kostenlose-llm-api-openclaw)、[Best AI Models for OpenClaw 2026](https://launchmyopenclaw.com/openclaw-best-models/)

### Mistral

- **Mistral Large 2512 為預設推薦**：除非工作流以 agent loop 與程式碼生成為主，否則 Mistral Large 2512 是最穩健的選擇；重度 coding 場景可改用 Devstral 2512。
- 來源：[Best Mistral Models for OpenClaw 2026](https://haimaker.ai/blog/best-mistral-models-for-openclaw/)

### OpenRouter

- 原生支援 `openrouter/<author>/<slug>` 格式，無需額外設定 `models.providers`，是跨 provider fallback 的最低阻力方案；目前可存取 200+ 至 300+ 模型。
- 來源：[Best AI Models for Hermes Agent 2026](https://www.remoteopenclaw.com/blog/best-models-for-hermes-agent)

---

## 3. 社群反應與討論亮點

- **Medium 實測文持續擴散**：Mohit Aggarwal 發表的「I Wired OpenClaw to Claude, ChatGPT & Telegram — Here's What Actually Works」詳述三家 LLM 串接的實際踩坑紀錄，含 auth 設定細節與 rate limit 應對策略，在社群廣泛轉發，是近期最受引用的入門實測文。
  - 來源：[Medium 文章](https://medium.com/@mohit15856/i-wired-openclaw-to-claude-chatgpt-telegram-heres-what-actually-works-8278906edccc)

- **OpenClaw vs Hermes Agent 論戰佔據 r/openclaw**：r/openclaw（10.3 萬成員）中多條比較討論串持續熱門，聚焦於兩者在 agent 自主度、LLM 相容性、插件生態與部署難度上的差異，是社群近期最活躍的話題。
  - 來源：[OpenClaw vs Hermes: What Reddit Actually Says (2026)](https://kilo.ai/articles/openclaw-vs-hermes-what-reddit-says)

- **「是否應延遲更新？」保守策略討論升溫**：延續上週 v2026.4.24～.26 連續三版重大問題的風波，社群持續討論「等一至兩個版本再升」的保守更新策略，部分重度用戶甚至設定了自動 pin 至指定版本的 Docker compose 設定。
  - 來源：[Openclaw useless after update discussion](https://github.com/orgs/community/discussions/188842)

- **Discord「Friends of the Crustacean」逼近 17 萬人**：社群持續成長，成為 LLM 選型情報與踩坑分享的核心平台；X.com 官方社群亦有 3.34 萬成員，是快速配置分享的主要場域。
  - 來源：[OpenClaw X Community](https://x.com/i/communities/2013441068562325602)

- **GitHub ClawSweeper 每週自動分類 Issues/PRs**：官方維護的 ClawSweeper 工具每週自動掃描所有 issues 與 PR，建議可關閉項目及理由，協助維護者管理現有 3,410 筆 open issues，被社群稱為「最有用的維護輔助工具之一」。
  - 來源：[GitHub openclaw/clawsweeper](https://github.com/openclaw/clawsweeper)

---

## 4. 值得追蹤的後續議題

1. **Issue #32828 假 Rate Limit Bug 修補時程**：影響所有 LLM provider 的全域性 bug，官方至今未給出預計修補版本，持續是 open issue 回報量最高的問題，建議每日追蹤。

2. **Issue #5744 Google Provider Cooldown 範圍過大**：Gemini 使用者的核心痛點，per-model limit 應只影響該模型而非整個 provider，修復後將大幅改善 Gemini 工作流穩定性。

3. **Discord Bot token 自動重設問題（Issue #58173）**：大量重連沒有 backoff 導致 Discord bot token 被自動重設，影響 Discord 為主要介面的使用者，需持續關注修補進度。

4. **Kimi K2.6 社群採用速度**：K2.6 在 benchmark 上已超越 GPT-5.4，若接下來一週社群投票數上升，可能取代 K2.5 成為推薦的新預設模型。

5. **Grok 4.20 Beta 正式版釋出時程**：多代理推理功能若轉正式，可能改變 OpenClaw 多代理工作流的首選 xAI 模型組合。

6. **Anthropic API 費用衝擊的中期評估**：訂閱封鎖滿一個月，各方用量與費用數據逐漸浮現，社群對「月費 vs. pay-as-you-go」的成本比較討論值得持續追蹤。

7. **v2026.5.x 插件 npm-first 遷移後的相容性測試**：外部插件安裝流程的重大重構，現有第三方插件的相容性問題預期將陸續在社群浮現。

---

*報告生成時間：2026-05-05 | 資料來源：GitHub Issues/PRs、Releasebot、newreleases.io、pricepertoken.com、Medium、TechCrunch、TNW、haimaker.ai、evolink.ai、Discord 社群、r/openclaw、X.com*
