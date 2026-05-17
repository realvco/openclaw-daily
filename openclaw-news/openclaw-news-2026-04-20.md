# OpenClaw 每日情報 2026-04-20

> 資料來源：GitHub、Discord、Reddit、社群部落格｜涵蓋過去 24 小時動態

---

## 1. 今日重點摘要

1. **v2026.4.19-beta.2 釋出**：修正 agent 用量回報錯誤、巢狀 session 隔離問題，以及舊版安裝升級失敗的 bug，是本週第二個 beta 版。
2. **PR #68863 修正 token 重複計算**：貢獻者 `tianhaocui` 提交 PR，修復 tool-use 迴圈中 prompt token 被雙重計算的問題，影響費用顯示精度。
3. **PR #68859 新增 `OPENCLAW_SESSION_KEY` 環境變數**：允許子行程繼承 session key，改善多行程部署場景。
4. **Kimi K2.5 登上社群投票第一名**：截至 4/19，Kimi K2.5 超越 Claude Opus 4.6 成為社群最推薦的 OpenClaw 模型，GLM 4.7 排名第二。
5. **Anthropic OAuth 訂閱配額已正式切斷（自 4/4 起）**：第三方工具（包含 OpenClaw）無法再使用訂閱方案的 quota，必須改用 API Key（`sk-ant-api03-*`）。
6. **CVE-2026-25253 高危漏洞（CVSS 8.8）**：遠端程式碼執行漏洞，攻擊者可透過惡意連結竊取 auth token，建議立即更新至最新版。
7. **Issue #52949：Gateway 未傳遞 ClawHub auth token**：造成持續性 429 rate limit 錯誤且 ClawControl 的「Available Skills」顯示空白，已有多人確認複現。
8. **OpenClaw vs Hermes 討論持續延燒**：r/openclaw（103,000 成員）出現大量比較文章，成為當前自主 AI agent 社群最熱議的話題。
9. **Mission Control v3.0 UI 整合 PR 合入中**：PR #68861 正在審查，將帶來管理介面的重大視覺更新。

---

## 2. LLM 串接實戰回報

### Anthropic（Claude 系列）

- **OAuth 訂閱切斷事件**：自 2026-04-04 起，Anthropic 正式停止向第三方工具（包含 OpenClaw）提供訂閱 quota。使用 `sk-ant-oat-*` 的使用者將遭遇 401/429 錯誤。**建議全面改用 API Key**。
  - 來源：[OpenClaw + Claude Code Costs 2026](https://www.shareuhack.com/en/posts/openclaw-claude-code-oauth-cost)
  - 來源：[Fix Every OpenClaw Error - LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)

- **OAuth + context-1m-\* header 衝突**：使用 OAuth token 時，OpenClaw 會跳過 `context-1m-*` beta header，因為 Anthropic 對該組合回傳 HTTP 401。API Key 用戶不受影響。
  - 來源：[OpenClaw API Rate Limit Guide](https://www.getopenclaw.ai/help/rate-limits-quota-management)

- **創辦人帳號被暫停事件**：4/10，OpenClaw 創辦人 Peter Steinberger 個人帳號即使已改用 API Key 仍被暫時封禁，數小時後恢復，引發社群對 Anthropic 政策透明度的質疑。
  - 來源：[answeroverflow - Claude OAuth rate limit thread](https://www.answeroverflow.com/m/1478788645472436264)

### Kimi K2.5 / OpenRouter

- **Kimi K2.5 tool-call 格式相容性**：透過 OpenRouter 接入時需使用 `openroutercustom` Provider 並設定 `"api": "openai-completions"`，否則 tool-calling 參數格式不相容。
  - 來源：[Connect Kimi, GLM-5, MiniMax via OpenRouter - WordUPR Blog](https://blog.wordupr.com/post/openclaw-connect-kimi-glm-5-minimax-via-openrouter/)

- **Kimi K2.5 費用控制**：社群反映 Kimi K2.5 在複雜推理任務上表現優異，成本低於 Claude Opus；搭配 OpenRouter 可享有動態路由與 fallback 機制。
  - 來源：[Best Model for OpenClaw - Price Per Token](https://pricepertoken.com/leaderboards/openclaw)

### DeepSeek R1

- **Sub-agent 低成本方案**：DeepSeek R1 每百萬 token 約 $2.74，為 Claude Opus 的 1/10，在長鏈推理 sub-agent 任務中獲得正面評價，但複雜 tool-use 場景穩定性略遜。
  - 來源：[Stop overpaying for OpenClaw - VelvetShark](https://velvetshark.com/openclaw-multi-model-routing)

### Google Gemini

- **高量 cron job 推薦**：Gemini Flash 比 Claude Sonnet 便宜顯著，適合高頻排程任務；Gemini 3 Pro 因超長 context window 被推薦用於長文件處理場景。
  - 來源：[How to Use OpenClaw with Google Gemini](https://www.openclawplaybook.ai/guides/how-to-use-openclaw-with-gemini/)

---

## 3. 社群反應與討論亮點

- **Discord 社群規模**：OpenClaw 官方 Discord（Friends of the Crustacean）已達 **171,975 成員**，社群持續快速成長。
- **r/openclaw 最熱話題**：OpenClaw vs Hermes Agent 的對比討論佔據 subreddit 首頁，聚焦於自主性、工具整合能力與穩定性的比較。
  - 來源：[OpenClaw vs Hermes Agent - Kilo AI](https://kilo.ai/articles/openclaw-vs-hermes-what-reddit-says)
- **「更新後無法使用」抱怨帖**：GitHub Discussions #188842 出現多位用戶反映某次更新後 OpenClaw 功能失常，維護者已介入回應。
  - 來源：[Openclaw useless now after update - community discussion](https://github.com/orgs/community/discussions/188842)
- **多模型路由策略分享**：社群中出現多篇「如何用 OpenRouter 結合多個 LLM 降低 80% 費用」的教學，引發廣泛討論。
  - 來源：[How to Run OpenClaw 24/7 - perelweb.be](https://perelweb.be/blog/openclaw-token-management-smart-model-manager/)

---

## 4. 值得追蹤的後續議題

| 議題 | 說明 | 狀態 |
|------|------|------|
| **Issue #52949** | Gateway 未傳遞 ClawHub auth token 導致持續 429 | 🔴 Open，待修復 |
| **CVE-2026-25253** | CVSS 8.8 遠端程式碼執行漏洞，token 竊取風險 | 🔴 需立即更新 |
| **PR #68861** | Mission Control v3.0 UI 整合，管理介面大改版 | 🟡 審查中 |
| **PR #68863** | Tool-use 迴圈 token 雙重計算修正 | 🟡 審查中 |
| **Anthropic OAuth 政策** | 第三方工具訂閱配額切斷後的長期影響 | 🟡 持續觀察 |
| **Kimi K2.5 穩定性** | 新晉社群推薦第一名，長期工作流穩定性待驗證 | 🟡 持續追蹤 |

---

*報告生成時間：2026-04-20｜資料涵蓋範圍：過去 24 小時*

**主要來源：**
- [Releases · openclaw/openclaw](https://github.com/openclaw/openclaw/releases)
- [openclaw/openclaw v2026.4.19-beta.2](https://newreleases.io/project/github/openclaw/openclaw/release/v2026.4.19-beta.2)
- [Model Providers - OpenClaw Docs](https://docs.openclaw.ai/concepts/model-providers)
- [Best Model for OpenClaw - Price Per Token](https://pricepertoken.com/leaderboards/openclaw)
- [Fix Every OpenClaw Error - LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)
- [OpenClaw API Rate Limit Guide](https://www.getopenclaw.ai/help/rate-limits-quota-management)
- [OpenClaw Release Notes - Releasebot](https://releasebot.io/updates/openclaw)
- [OpenClaw vs Hermes - Kilo AI](https://kilo.ai/articles/openclaw-vs-hermes-what-reddit-says)
- [GitHub Issue #52949](https://github.com/openclaw/openclaw/issues/52949)
