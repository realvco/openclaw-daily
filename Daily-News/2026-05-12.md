# OpenClaw 每日情報 2026-05-12

> 搜尋區間：2026-05-11 至 2026-05-12｜以下資訊來自 GitHub、DEV Community、Medium、社群討論及官方公告

---

## 今日重點摘要

1. **v2026.5.9-beta.1 發布**：2026 年 5 月 9 日釋出，帶來重大語音功能升級，包括 Discord 雙向即時語音（bidi realtime）、STT/TTS/agent-proxy 三模式切換，以及 agent-proxy 設為預設語音模式。
2. **Node.js 最低版本提升至 22.16+**：以支援 `node:sqlite` statement metadata API，官方同步建議升級至 Node 24；舊版環境需評估升級成本。
3. **Slack 隱式對話路由改善**：reply threading 啟用時，Slack 頻道頂層對話自動轉至 thread-scoped session，確保後續討論保持同一 OpenClaw session。
4. **Codex 整合版本釘定**：managed Codex harness 釘定至 `@openai/codex@0.129.0`，舊 `fast` 設定升級後仍可延用，避免相容性中斷。
5. **假冒 Rate Limit 錯誤 Bug 持續受關注**：Issue #32828 與 #52949 指出即使 API 正常運作仍觸發 rate limit 錯誤，ClawHub auth token 未正確傳遞問題尚待修復。
6. **GitHub 活躍開發**：主倉庫達 76,694 stars、3,630 個 open issues，5 月 11 日新增多個 bug report 及 PR（含 trajectory 功能 PR #80683）。
7. **多模型路由成主流**：April 2026 推出 provider manifest，可在不重建的情況下即時切換 LLM，支援 GPT-5.5、Claude、Gemini、DeepSeek 等六家 provider。
8. **社群人心動盪**：v2026.4.26 更新造成 gateway 崩潰，部分長期用戶轉向 Hermes Agent，官方部落格坦承「度過了艱難的一週」。
9. **安全管控強化**：Active Memory tool allowlist 支援自訂記憶插件白名單；全域記憶開關現在需要 admin scope；內聯 skill dispatch 加入 before-tool 授權鉤子。

---

## LLM 串接實戰回報

### Claude Opus 4.7（Anthropic）

- **SWE-bench Pro 表現領先**：64.3%，優於 GPT-5.5（58.6%）與 DeepSeek V4-Pro（55.4%）
- **適用場景**：嚴格程式碼審查、複雜多步驟 tool-calling（穩定性最高）
- **費用**：$25/M tokens（輸出），屬高端定位
- **踩坑紀錄**：OAuth token 格式需使用 `"type": "token"` + `"provider": "anthropic"`，若誤用 `"type": "api_key"` + `"key"` 欄位，錯誤會被誤判為 rate limit，難以排查
- 來源：[Best LLM for OpenClaw: Gemini 3.1 Pro vs GPT-5.5 vs Claude Opus 4.7 (2026)](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)、[OpenClaw Best Model Selection Guide](https://blog.laozhang.ai/en/posts/openclaw-best-model-selection-guide)

### Gemini 3.1 Pro（Google）

- **推薦為大多數 OpenClaw 部署的預設模型**：大型 context 程式碼審查效能優異，成本最低，提供免費開發者層
- **GPQA Diamond** 表現：94.3%（與 Claude Opus 4.7 並列最高）
- **Native multimodal** 支援，適合含圖像的工作流
- 來源：[Best Models for OpenClaw (April 2026)](https://haimaker.ai/blog/best-models-for-clawdbot/)、[How to Use OpenClaw with Google Gemini](https://www.openclawplaybook.ai/guides/how-to-use-openclaw-with-gemini/)

### GPT-5.5（OpenAI / Codex）

- **Terminal-Bench 2.0 自主任務表現最佳**：82.7%，適合 context 在 128K tokens 以內的自主 agent 任務
- **費用**：$30/M tokens（輸出），與 Claude Opus 4.7 相近
- **注意**：v2026.5.9-beta.1 中 Codex 釘定至 `@openai/codex@0.129.0`，升級前請確認版本相容性
- 來源：[OpenClaw Multi-Model Routing: Claude + Gemini + GPT (2026)](https://computertech.co/openclaw-multi-model-routing-guide-2026/)

### DeepSeek V4-Pro

- **成本最低**：$3.48/M tokens（輸出），比 Claude Opus 4.7 便宜約 7 倍
- **複雜多步驟 tool-calling 穩定性不足**：社群回報在 OpenClaw agent 工作流中準確率明顯低於 Claude，建議作為輔助模型而非主要模型
- **GPQA Diamond**：90.1%
- 來源：[OpenClaw Best Model Selection Guide](https://blog.laozhang.ai/en/posts/openclaw-best-model-selection-guide)、[DeepSeek V4 vs Claude Opus 4.7 vs GPT-5.5](https://lushbinary.com/blog/deepseek-v4-vs-claude-opus-4-7-vs-gpt-5-5-comparison/)

### 已知 Auth / Rate Limit 踩坑彙整

| 問題 | Issue | 症狀 | 快速排查 |
|------|-------|------|----------|
| 假冒 rate limit | [#32828](https://github.com/openclaw/openclaw/issues/32828) | 所有模型同時顯示「API rate limit reached」，實際 API 正常 | 降版或等待修復；`openclaw doctor --fix` |
| ClawHub auth token 未傳遞 | [#52949](https://github.com/openclaw/openclaw/issues/52949) | 持續 429 + 「Available Skills」清單為空 | 手動重設 ClawHub token；`openclaw doctor --fix` |
| OAuth token 格式錯誤 | [社群回報](https://www.answeroverflow.com/m/1478788645472436264) | Claude OAuth 持續 rate limit 警告 | 確認格式為 `"type": "token"` 非 `"type": "api_key"` |

---

## 社群反應與討論亮點

### Discord「Friends of the Crustacean」🦞（176,142 成員）

- v2026.4.26 更新後大量用戶回報 gateway 崩潰，截止 4 月 29 日問題擴大，引發社群強烈不滿（[報導來源](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)）
- 部分長期用戶公開宣布轉向 Hermes Agent，理由是更新更穩定、bug 更少
- Claude OAuth auth 踩坑是近期最熱門的討論主題

### X.com 亮點

- **Scott Belsky**（[@scottbelsky](https://x.com/scottbelsky/status/2017436989604192505)）：「good breakdown of the architecture behind OpenClaw/Clawdbot, and also makes it perfectly clear that our operating systems are overdue for reimagination. the big OS companies need to lean in hard here…this is the future」—直指傳統 OS 廠商需要在此領域加大力道

### 官方部落格

- OpenClaw 官方發文〈[OpenClaw Had a Rough Week](https://openclaw.ai/blog/openclaw-rough-week)〉坦承：自 2026.4.24 開始核心模組拆分動作導致問題累積，4 月 29 日集中爆發。目前策略是持續收縮核心，將可選功能移至 ClawHub。

### DEV Community / Medium

- **架構解析熱文**：Task Brain 統一任務管理層被比喻為「AI agent 版的 Kubernetes scheduler」，SQLite-backed ledger 設計引發廣泛討論（[OpenClaw and the Architecture Nobody Noticed](https://dev.to/david_aronchick_ea415de50/openclaw-and-the-architecture-nobody-noticed-2kbk)）
- **Gemini 整合教學廣受歡迎**：[Using Gemini with OpenClaw: Setup Guide + Real Use Cases](https://dev.to/matthewrevell/using-gemini-with-openclaw-setup-guide-real-use-cases-2i48)

---

## 值得追蹤的後續議題

1. **v2026.5.9 穩定版上線時程**：beta.1 已推出，Discord 語音、Slack thread routing、Telegram re-probe 等功能仍在驗證中
2. **Issue #52949（ClawHub auth token bug）修復進度**：直接影響所有使用 ClawHub skill 的用戶，為高優先度缺陷
3. **Issue #32828（假冒 rate limit）**：已歷經多個版本未解決，社群壓力持續升高，需關注是否列入 hotfix 計畫
4. **PR #80683（Trajectory 功能）**：若合入主線，將為 agent 工作流帶來新的任務軌跡追蹤能力
5. **Node 22.16+ / Node 24 遷移衝擊評估**：使用舊版 Node.js 的自架部署環境需盡早規劃升級
6. **DeepSeek V4 多步驟 tool-calling 穩定性改善**：若 DeepSeek 修復此問題，其極具競爭力的價格將更有吸引力
7. **用戶流失趨勢觀察**：Hermes Agent 吸引的是短期情緒宣洩還是長期用戶？需追蹤後續 OpenClaw 社群規模變化

---

*報告生成時間：2026-05-12 | 資料來源涵蓋 GitHub、DEV Community、Medium、X.com、官方部落格及社群論壇*
