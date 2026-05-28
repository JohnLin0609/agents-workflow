# Retrospective(回顧)

> **目的**:從上一段工作(sprint / 專案 / 事件)中學習,讓下一段更好。
> **負責人**:Scrum Master / Tech Lead / 任何願意主持的人
> **核心原則**:**Action items 要有人、有期限、有追蹤**——沒這三項的回顧就是聊天。

---

# Retrospective: [Sprint / Project 名稱]

| 欄位 | 內容 |
|------|------|
| 期間 | YYYY-MM-DD ~ YYYY-MM-DD |
| 主持人 | |
| 參與者 | |
| 形式 | 線上 / 實體 / 混合 |
| 時長 | 60 min |

---

## 1. 暖身(5 min)

> 不要直接跳進檢討。先讓大家狀態進來。

**選一個**(輪流):
- 用一個顏色形容這個 sprint
- 用天氣形容這個 sprint
- 你的能量等級 1–10

---

## 2. 數據回顧(5 min)

> 拿事實鋪墊,不要憑感覺。

| 指標 | 本次 | 上次 | 趨勢 |
|------|------|------|------|
| 完成 story points | 32 | 28 | ↑ |
| 計畫 vs 完成 | 80% | 75% | ↑ |
| 上線次數 | 5 | 4 | ↑ |
| Incident | 1 (P1) | 0 | ↓ |
| Bug 新增 / 修復 | 12 / 15 | 8 / 9 | |
| Code review 平均時長 | 6h | 8h | ↑ |

---

## 3. 收集(15 min)

> 大家寫便利貼 / 線上協作工具。**先寫不討論**,避免被別人影響。

### 3.1 做得好(What went well)👍
- 範例:Pair programming 在 ORD-005 救了我們
- ...

### 3.2 沒做好(What didn't go well)👎
- 範例:Code review 卡了 2 天才有人看
- ...

### 3.3 困惑 / 問題(Questions / Confusions)🤔
- 範例:不確定為什麼這個 sprint 突然加了 5 個任務
- ...

### 3.4 想嘗試(Ideas to try)💡
- 範例:每天早上 15 分鐘 pair review
- ...

> 變化型:Start / Stop / Continue、Mad / Sad / Glad、4Ls(Liked / Learned / Lacked / Longed for)

---

## 4. 群組與排序(10 min)

- 把類似的便利貼歸類
- 大家投票挑出最想討論的 3–5 個主題(每人 3 票)

---

## 5. 討論(20 min)

> 針對票數最高的主題深入討論。每個主題約 5–7 分鐘。

### 主題 1:Code Review 卡很久

**現況**:平均 8h 才有人看,有時甚至兩天。

**討論**:
- 大家覺得卡因為:正在做自己的事、不知道有 PR、PR 太大
- Alice 建議:設 SLA 24h、Slack 整合自動 ping
- Bob 建議:大 PR 拆小

**結論 / Action**:
- 設 review SLA 24h(Bob 設定 GitHub Slack 通知)
- PR 超過 500 行請拆分(寫進 review checklist)

### 主題 2:...

---

## 6. Action Items(5 min)

> **這是最重要的一段**。沒有 action items 的 retro 等於沒開。

| ID | 行動 | 負責人 | 期限 | 衡量成功 |
|----|------|--------|------|---------|
| RT-01 | 設定 PR 通知到 Slack | Bob | 下週一 | 通知到位 |
| RT-02 | PR 超過 500 行請拆分 | 全員 | 立即生效 | 下次 retro 看平均 PR 大小 |
| RT-03 | 每天站會加 5 分鐘預估 | Alice | 下個 sprint | 試行兩週 |

**追蹤**:每次 retro 第一件事先 review 上次 action items。

---

## 7. 上次 Action Items 追蹤

| ID | 行動 | 狀態 | 備註 |
|----|------|------|------|
| RT-(prev-01) | 引入 ADR 文件 | ✅ 完成 | 已寫 3 份 ADR |
| RT-(prev-02) | 升級 Node 18 | 🚧 進行中 | 排在下個 sprint |
| RT-(prev-03) | 引入 pair programming | ❌ 未跟進 | 為什麼?討論 |

---

## 8. 結束(5 min)

### 8.1 ROTI(Return on Time Invested)
> 大家評分這次 retro 值不值得花這個時間(1–5)。

### 8.2 一句感謝
> 輪流感謝某位夥伴的某件事。正能量結尾,不是儀式,是維繫團隊的小事。

---

## 9. 主持人注意事項

### 9.1 安全感
- Blameless,不指責個人
- 「對事不對人」是規則,不是建議
- 主管在場時更要注意,避免人不敢講

### 9.2 引導
- 安靜的人主動點名問
- 講太久的人禮貌打斷
- 主持人不該主導意見,只引導

### 9.3 避免
- 變成抱怨大會,沒有 action
- 變成只討論技術,忽略流程與協作
- Action items 太多(超過 5 個 = 一個都不會做)

### 9.4 變化(避免 retro 疲勞)
- 換形式:Sailboat、Starfish、Timeline
- 換主持人
- 偶爾線下、換場地

---

## 10. 相關文件
- 上次 Retro:[連結]
- Action Items 追蹤板:[連結]
- 團隊 Working Agreement:[連結]
