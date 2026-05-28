# Lessons Learned(經驗知識庫)

> **目的**:把每次 retro、incident、專案結束的學習累積成組織記憶。
> **負責人**:Engineering Manager / Tech Lead
> **核心原則**:**寫下來才會傳承**。記憶會走,人會換,文件留下來。
> **形式**:單一文件持續累加,或 wiki / Notion 資料庫。

---

## 使用方式

- 每次 retro / postmortem 結束,把重要 lesson 摘要進來
- 新人加入時,這份文件是必讀
- 開新專案前,翻一下相關 lesson 避免重蹈覆轍
- 每季 review 一次,過時的標記或移除

---

## 索引

- [技術](#技術)
- [流程](#流程)
- [協作](#協作)
- [上線與維運](#上線與維運)
- [反模式 / 不要做的事](#反模式)

---

## 技術

### L-T-001:資料庫變更要 Expand-Contract
**情境**:2025-Q3 改了 orders.status 欄位型別,新舊程式碼不相容,造成 30 分鐘 downtime。
**學到**:破壞性 schema 變更要分多階段:加新 → 雙寫 → 切讀 → 移除舊。永遠不要一步到位。
**對應**:已寫進 coding-standards.md 與 data-schema.md。
**Tags**:`#database` `#migration`

### L-T-002:外部 API 一定要有 timeout 與 circuit breaker
**情境**:金流商連續 timeout 把我們的 thread pool 吃光,連帶不相關功能也掛。
**學到**:任何同步呼叫外部都要設 timeout(通常 < 3s),關鍵路徑要有 circuit breaker。
**對應**:edge-cases.md 第 7 節。
**Tags**:`#integration` `#resilience`

### L-T-003:N+1 query 在小資料下看不出來
**情境**:dev 環境 10 筆資料 100ms,prod 10 萬筆變 30s。
**學到**:dev 環境要有「接近 prod 規模」的測試資料子集。
**Tags**:`#performance` `#testing`

---

## 流程

### L-P-001:大 PR 是雙輸
**情境**:一個 1500 行的 PR 卡了一週才被 review,期間 4 次 conflict。
**學到**:
- 作者:寫的時候就在心裡拆 PR,500 行是上限
- Reviewer:大 PR 直接退回,不要硬看
**對應**:review-checklist.md。
**Tags**:`#review` `#workflow`

### L-P-002:估算用人天比 story point 直觀(對小團隊)
**情境**:5 人團隊試了 story point 半年,大家還是私下換算成天,反而增加溝通成本。
**學到**:工具要看團隊規模選。小團隊直接人天可能更實用,大團隊跨組才需要相對單位。
**Tags**:`#planning` `#estimation`

### L-P-003:Retro 沒 action 等於沒開
**情境**:連續 3 次 retro 都討論「code review 太慢」,但都沒人改。
**學到**:每個 action 必須有負責人 + 期限。下次 retro 第一件事 review 上次 action。
**對應**:retrospective.md。
**Tags**:`#retro` `#process`

---

## 協作

### L-C-001:跨團隊介面要兩邊都 review
**情境**:Order 與 Inventory 對「庫存預扣」的語意理解不同,整合時才發現,白做 3 天。
**學到**:介面文件兩邊 owner 都要 review,不只是技術 lead 一個人簽。
**對應**:interfaces.md。
**Tags**:`#collaboration` `#interface`

### L-C-002:遠距同步比想像中重要
**情境**:全遠距 sprint,光靠文件協作,大家對優先序理解差很多。
**學到**:每週至少一次同步會,即使只有 30 分鐘。文件補不了 high-bandwidth 對話。
**Tags**:`#remote` `#communication`

---

## 上線與維運

### L-O-001:Resource 監控不只是「夠不夠」,還要「離極限多遠」
**情境**:INC-2026-0042,連線池滿了才被發現,因為只監控了「服務是否健康」。
**學到**:資源類指標(連線池、thread pool、queue depth)要監控使用率,80% 預警。
**對應**:monitoring-dashboards.md。
**Tags**:`#monitoring` `#sre`

### L-O-002:Rollback 演練不能省
**情境**:第一次回滾才發現 K8s rollout undo 在某個版本壞掉,當場現查 30 分鐘。
**學到**:重大 release 前在 staging 跑一次完整 rollback,確認工具與步驟有效。
**對應**:rollback-plan.md 第 7 節。
**Tags**:`#deployment` `#sre`

### L-O-003:行銷活動上線要事先壓測
**情境**:雙 11 流量是平日 5 倍,我們只壓測 2 倍,當天爆掉。
**學到**:壓測目標要參考「預期峰值 × 安全係數(通常 2x)」,不是平日 × 倍數。
**Tags**:`#performance` `#planning`

---

## 反模式

> 「不要做的事」的清單。新人特別容易踩。

### A-001:catch (e) {} 吞錯誤
- **後果**:bug 沉默地在 prod 累積,難以追蹤
- **正確**:至少要 log,或往上拋

### A-002:hardcoded secret
- **後果**:一旦 leak 全公司危險
- **正確**:env var / secret manager

### A-003:在 prod console 手動下指令
- **後果**:不可追溯、無法重現、容易出錯
- **正確**:寫成腳本,進版控,peer review

### A-004:「先這樣之後再改」
- **後果**:之後通常不會改,變成永久技術債
- **正確**:當下就建 ticket,排進 backlog

### A-005:用 console.log 除錯然後忘了刪
- **後果**:logs 噪音、可能 log 敏感資料
- **正確**:用 debugger / 結構化 log,CI 擋 console.log

---

## 模板:新增 Lesson

```markdown
### L-?-XXX:[一句話標題]
**情境**:發生了什麼
**學到**:具體 takeaway
**對應**:已寫進哪份文件 / 對應的 PR
**Tags**:`#tag1` `#tag2`
```

---

## Review 紀錄

| 日期 | Reviewer | 變動 |
|------|---------|------|
| YYYY-MM-DD | | 季度 review,移除 2 條過時、新增 5 條 |
