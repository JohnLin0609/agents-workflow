# 08 · 監控與維護(Operations)

> **流程的第八步。** 核心觀念:**上線不是結束,是另一個開始**。
> 這一步有兩種活動:**事前**設好監控與告警,**事後**(出事時)寫 postmortem。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §12「階段八:監控與維護」。
> 用法見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

## 這個資料夾的模板

| 模板 | 用途 | 維護方式 |
|------|------|----------|
| `monitoring-dashboards.md` | 監控哪些訊號、儀表板怎麼擺、SLI/SLO | **活文件** |
| `alert-runbook.md` | 告警分級、每條告警的觸發條件與處理步驟 | **活文件** |
| `incident-report.md` | 事故的時間軸、根因、action items(postmortem) | 每次事故一份,餵進 09 |

## 這一步要達成什麼(§12 的把關標準)

- **黃金訊號都有監控**(Google SRE 四訊號):
  | 訊號 | 量測 |
  |------|------|
  | Latency | P50 / P99 |
  | Traffic | RPS |
  | Errors | 5xx 比例 |
  | Saturation | CPU/記憶體/連線池使用率(**80% 預警**) |
- **每條告警都可行動、且有 runbook** —— 醒來知道要幹嘛。
- **SLI/SLO 定義清楚**。
- **事故走 blameless postmortem**,action items 有人、有期限、有票號。

---

## 操作流程 A:事前設好監控與告警(搭配 grill-me)

grill-me 是 [@mattpocock](https://github.com/mattpocock/skills) 的 skill,這裡用它**挑掉爛告警**。
安裝見根 README。

### 前置(artifact 交接)

- 先給 Claude 輸入文件:
  ```
  先讀 docs/01-requirements/<專案名>-requirements.md(成功指標可推 SLO)、
  docs/02-design-system/ 與 docs/03-design-detail/api-spec.md(關鍵端點)、
  docs/04-planning/<專案名>-risk-register.md。這是這次設定監控告警的依據。
  ```
- 把模板複製進 `docs/`:
  ```bash
  mkdir -p docs/08-operations
  cp ~/dev-process/08-operations/monitoring-dashboards.md docs/08-operations/<專案名>-monitoring-dashboards.md
  cp ~/dev-process/08-operations/alert-runbook.md         docs/08-operations/<專案名>-alert-runbook.md
  cp ~/dev-process/08-operations/incident-report.md       docs/08-operations/  # 出事時才複製一份來填
  ```

### 第一步:草擬監控與告警,再用 grill-me 挑

先請 Claude 出初稿:

```
根據需求的成功指標、關鍵端點與風險登記簿,草擬:
1. 監控:四個黃金訊號各監控什麼、要不要加業務指標、SLI/SLO 建議
2. 告警:哪些條件要告警、分級(P0–P3)、每條告警的處理 runbook
先別寫進檔案,我們先討論。
```

拿到初稿後,用 grill-me 挑掉爛告警:

```
/grill-me

針對剛剛的監控與告警訪談我:
- 每條告警都「可行動」嗎?醒來知道幹嘛嗎?還是純告知性(那該用 dashboard 不是 alert)?
- 哪些告警會造成 alert fatigue(太吵、假警報多)?
- 每條告警有對應 runbook 嗎?
- 是對症還是對因?Saturation 有沒有看「離極限多遠」而不只是「夠不夠」?
一次問一題,每題附上你建議的答案。
```

回答要點:

- **好告警四標準**:可行動、有意義、即時、有 context。做不到就別設成告警。
- **沒 runbook 的告警是負債** —— 寫告警就配處理步驟。
- **Saturation 看使用率**(連線池/thread pool/queue),80% 預警,不要等到 100% 才知道。

### 第二步:把對話填進監控與告警文件

```
根據我們剛剛的對話,把 docs/08-operations/<專案名>-monitoring-dashboards.md
與 alert-runbook.md 就地填好。沒定的標 TBD,不要編。
```

---

## 操作流程 B:出事時寫 postmortem(incident-report)

重大事故發生並止血後,複製 `incident-report.md` 來填。**對事不對人**。

```
cp ~/dev-process/08-operations/incident-report.md docs/08-operations/<YYYY-MM-DD>-incident.md
```
```
根據這次事故經過 <貼時間軸與事實>,幫我填 docs/08-operations/<YYYY-MM-DD>-incident.md:
摘要、影響、時間軸、用 5 Whys 做根因分析、哪些做得好、哪些可改善、
action items(每項要有負責人、期限、票號)。
語氣 blameless——對事不對人,目的是學習不是處罰。
```

> incident-report 的學習要餵進 `09-retrospectives/` 的 lessons-learned 累積文件。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **訪談 / 寫 postmortem 進行中** | 不要 `/clear` | 會弄丟剛討論的告警取捨或事故脈絡 |
| **對話很長、開始變慢** | 用 `/compact` | 壓縮但保留脈絡 |
| **文件落地後** | 可 `/clear` | 監控與事故是兩件獨立的事,各自開新一輪讀對應文件 |

> **monitoring-dashboards 與 alert-runbook 是活文件**:系統變了、學到新教訓就回來更新。

---

## 完成標準(進入 `09-retrospectives/` 前)

- [ ] 四個黃金訊號都有監控,Saturation 看使用率且有 80% 預警
- [ ] 每條告警都可行動、有分級、有對應 runbook;沒有純告知性告警
- [ ] SLI/SLO 已定義
- [ ] `incident-report` 模板就緒;真有事故時走 blameless、action items 有人有期限有票號

事故的 action items 與監控學到的東西 → 帶進第九步 `09-retrospectives/` 做回顧並累積成 lessons learned。
