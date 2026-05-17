# OpenClaw 每日情報 2026-05-10

> 資料涵蓋範圍：過去 24 小時（2026-05-09 ～ 2026-05-10）

---

## 1. 今日重點摘要

1. **最新版本 v2026.5.7 發布**：修復原生指令權限執行（現強制 owner 驗證），全域記憶體開關改為需管理員權限，Telegram 支援存取群組白名單（`accessGroup:*`），Cron 任務修復多項持久化 bug（payload.model 儲存為 null/blank 時自動修復）。

2. **v2026.5.5 修復 Feishu 對話路由問題**（發布於 5/6）：修復 Feishu 原生話題起始執行緒 ID 水化問題，確保同一對話串流不被中斷；TUI 啟動優化，修復殭屍進程（orphaned openclaw-tui）問題；`doctor --fix` 可修復心跳歷史中毒的 main session。

3. **Anthropic OAuth 驗證政策變更持續衝擊用戶**：自 2026 年 4 月 4 日起，Claude Pro/Max 訂閱 OAuth Token 僅供官方工具，OpenClaw 等第三方工具須改用 API Key 付費。官方 X 帳號在 v2026.4.5 推文中直接表明「Anthropic cut us off. GPT-5.4 got better. We moved on.」

4. **Issue #23996：假性速率限制冷卻 bug — Critical 等級**：提供者冷卻原因被硬編碼為 `rate_limit`，無論實際錯誤原因（逾時、計費、Auth 失效等），所有代理回覆被完全封鎖且無法自動恢復，目前尚無官方熱修補。

5. **多模型路由（Multi-Model Routing）持續完善**：社群評測確立三強格局——Gemini 3.1 Pro（一般用途、最低成本）、Claude Opus 4.7（嚴格程式碼審查，SWE-bench Pro 64.3%）、GPT-5.5（自主代理，Terminal-Bench 2.0 領先）。

6. **Qwen3.6-plus 支援 PR 已提交**（5/9）：若合併成功，將成為 OpenClaw 首個正式支援的 Qwen3 系列模型，對亞太地區用戶具重大意義。

7. **xAI Grok 4.3 正式加入（v2026.5.2）**：繼 DeepSeek V4 Flash/Pro（v2026.4.24）後，Grok 4.3 成為 OpenClaw 最新支援的前沿模型，進一步豐富多模型選擇。

8. **v2026.4.26 大規模閘道崩潰事件餘波**：Discord 整合失效、閘道啟動崩潰循環問題仍有討論，部分長期用戶宣告轉向 Hermes Agent，OpenClaw 僅作備用。

9. **OAuth + 百萬 Token 上下文視窗不相容**：使用 `sk-ant-oat-*` OAuth Token 搭配 context-1m 模型收到 HTTP 401，需改用 200k 版本（Sonnet 4.6 / Opus 4.6）暫時迴避。

---

## 2. LLM 串接實戰回報

### Anthropic Claude

**OAuth 驗證失效與 API Key 強制轉換**

2026 年 1 月起 Anthropic 開始收緊 OAuth Token 使用，4 月 4 日後正式封鎖第三方工具使用 Pro/Max 訂閱 Token。目前唯一實際可行方案為 API Key 付費存取。

**踩坑 1：OAuth + 1M Context 視窗 → HTTP 401**

使用 `sk-ant-oat-*` Token 搭配 context-1m 模型時，Anthropic 拒絕 `context-1m-*` beta header，回傳 HTTP 401。
- 解法：改用 200k 上下文版本（claude-sonnet-4-6 或 claude-opus-4-6）

來源：
- [Fix Every OpenClaw Error: API Key, Rate Limits, Gateway Token](https://blog.laozhang.ai/en/posts/openclaw-api-key-error)
- [OpenClaw + Claude Code Costs 2026: API Key vs Pro $20 vs Max $200](https://www.shareuhack.com/en/posts/openclaw-claude-code-oauth-cost)
- [Claude Code OAuth Token Expiry: Fixes & Alternatives](https://daveswift.com/claude-oauth-update/)

---

### 假性速率限制冷卻 Bug（Issue #23996）— Critical

**現象**：所有提供者同時進入冷卻，代理回覆完全中斷，無法自動恢復。  
**根因**：提供者冷卻原因硬編碼為 `rate_limit`，無論實際錯誤為超時、計費問題、Auth 失效，均觸發相同冷卻邏輯。  
**狀態**：截至 5/10，尚無官方熱修補，可手動重啟 gateway 暫時恢復。

來源：[Bug #23996 · openclaw/openclaw (GitHub)](https://github.com/openclaw/openclaw/issues/23996)

---

### 多模型效能比較（社群實測）

| 模型 | 推薦用途 | 重要備注 |
|------|----------|----------|
| **Gemini 3.1 Pro** | 一般用途首選 | 大上下文程式碼審查、成本最低、有免費開發層、原生多模態 |
| **Claude Opus 4.7** | 嚴格程式碼審查 | SWE-bench Pro 64.3%，指令遵循最強，自我檢查行為明顯 |
| **GPT-5.5** | 自主代理任務 | Terminal-Bench 2.0 領先，注意 128K Token 上限 |
| **DeepSeek V4 Flash/Pro** | 中低成本高效 | v2026.4.24 加入，適合預算敏感部署 |
| **xAI Grok 4.3** | 新加入，待評測 | v2026.5.2 加入，社群實測資料尚少 |
| **Qwen3.6-plus** | 亞太地區 | PR 待合併，正式支援後值得測試 |

來源：
- [Best LLM for OpenClaw: Gemini 3.1 Pro vs GPT-5.5 vs Claude Opus 4.7 (2026) — DEV Community](https://dev.to/matthewrevell/best-llm-for-openclaw-gemini-31-pro-vs-gpt-55-vs-claude-opus-47-2026-3na4)
- [Best Models for OpenClaw (April 2026): Tested & Ranked — haimaker.ai](https://haimaker.ai/blog/best-models-for-clawdbot/)
- [Best Model for OpenClaw — LLM Rankings & Community Votes (2026) — pricepertoken.com](https://pricepertoken.com/leaderboards/openclaw)
- [OpenClaw LLM Setup: Current Provider, Model, Auth, and Fallback Guide — LaoZhang AI Blog](https://blog.laozhang.ai/en/posts/openclaw-llm-setup)

---

### Token 費用控制實戰

OpenClaw 智慧模型管理器（Smart Model Manager）可依任務類型自動切換最低成本模型，社群案例顯示可降低最高 80% Token 費用。實作方式：在 `model-manager` 外掛設定任務類型規則，輕量任務路由至 Gemini Flash 或 DeepSeek Flash，複雜任務路由至 Claude Opus 或 GPT-5.5。

來源：[How to Run OpenClaw 24/7 Without Breaking the Bank: Eliminate Rate Limits + Cut Costs by 80%](https://perelweb.be/blog/openclaw-token-management-smart-model-manager/)

---

## 3. 社群反應與討論亮點

### 官方 X.com 動態

- **OpenClaw 官方 (@openclaw)** — v2026.5.4：  
  「cleaner plugin installs、faster Gateway startup、better doctor/repair hints、Windows + Discord fixes」— 強調「boring got fast」，訊號顯示重心已轉向穩定性打磨。

- **OpenClaw 官方** — v2026.5.2：  
  「Less drama. More uptime.」——首次加入 xAI Grok 4.3，plugin 穩定性大修，Discord/Slack/Telegram/WhatsApp 四平台修復。

- **OpenClaw 官方** — v2026.4.5（歷史脈絡）：  
  「Anthropic cut us off. GPT-5.4 got better. We moved on.」——官方公開表態 Anthropic OAuth 封鎖事件，語氣強硬，社群引發大量討論。

- **Luke The Dev (@iamlukethedev)**：  
  v2026.4.23 詳細分析推文，重點介紹 gpt-image-2 支援與 GPT-5.5 加入。

來源：
- [OpenClaw v2026.5.4 官方推文](https://x.com/openclaw/status/2051582130417721696)
- [OpenClaw v2026.5.2 官方推文](https://x.com/openclaw/status/2050735037230801042)
- [OpenClaw v2026.4.5 官方推文](https://x.com/openclaw/status/2040998570317197607)

### Discord 社群（Friends of the Crustacean 🦞🤝）

- **成員數**：176,015 人（官方 Discord，邀請碼：`discord.com/invite/clawd`）
- **熱點討論**：v2026.4.26 更新後閘道崩潰問題，部分用戶在社群中公告「暫時轉用 Hermes Agent」
- **Claude OAuth 問題**：多名用戶反映搭配 Claude OAuth 後持續收到「API rate limit reached」，追查後確認為 Issue #23996 假性冷卻 bug，而非真實超限

來源：
- [OpenClaw 2026.4.26 update triggering gateway crashes — Piunikaweb](https://piunikaweb.com/2026/04/29/openclaw-update-triggering-gateway-crashes-issues/)
- [Openclaw with claude oauth rate limit issue — answeroverflow.com](https://www.answeroverflow.com/m/1478788645472436264)

### GitHub 社群（截至 2026-05-09）

- **Issues 總數**：約 3,900 個；**PRs 總數**：約 3,800 個
- **5/9 單日活躍 PR**：Qwen3.6-plus 支援、Codex response tool replay 修復、自動模型回退 pin、Markdown blockquote 樣式修正、model auth health 狀態修復
- **用戶討論串**：`openclaw useless now after update` (#188842) 反映大型更新後的挫敗感，但後續多數問題可透過 `openclaw doctor --fix` 修復

來源：
- [Issues · openclaw/openclaw (GitHub)](https://github.com/openclaw/openclaw/issues)
- [Pull requests · openclaw/openclaw (GitHub)](https://github.com/openclaw/openclaw/pulls)
- [Openclaw useless now after update · community Discussion #188842](https://github.com/orgs/community/discussions/188842)

---

## 4. 值得追蹤的後續議題

1. **Issue #23996 假性冷卻 bug 修復進度**  
   Critical 等級，影響所有使用提供者路由的實例，是否有官方熱修補（hotfix）為近期最重要追蹤項目。

2. **Qwen3.6-plus PR 合併狀態**  
   若順利合併，將開啟 OpenClaw 對 Qwen3 系列的正式支援，對中文語境任務和亞太部署意義重大。

3. **Anthropic OAuth 封鎖後的用戶遷移趨勢**  
   部分用戶已轉向 Gemini 或 GPT 作為主力 LLM，OpenClaw + Claude 組合的使用黏性和市佔率走向值得持續觀察。

4. **v2026.4.26 穩定性危機的信心修復**  
   多個 5.x 版本連續強調「Less drama. More uptime.」，需持續追蹤社群對穩定性的評價是否回升，以及 Hermes Agent 的競爭態勢。

5. **Gemini 3.1 Pro 作為默認模型的生態影響**  
   社群評測多將 Gemini 3.1 Pro 列為首選，若 Composio Gemini MCP 整合普及，可能進一步鞏固 Gemini 在 OpenClaw 生態中的地位。

6. **xAI Grok 4.3 實戰評測累積**  
   v2026.5.2 剛加入，目前社群實測資料有限，需追蹤穩定性、速率限制、成本表現等具體回報。

7. **多模型路由（Multi-Model Routing）商業化動向**  
   官方和社群部落格積極推廣，需關注是否出現付費層或用量限制，以及自動路由決策邏輯是否開源。

---

*報告生成時間：2026-05-10*  
*資料來源：GitHub Releases、X.com 官方動態、DEV Community、Discord 社群、個人技術部落格（LaoZhang AI Blog、Perelweb、Haimaker.ai、PricePerToken 等）*
