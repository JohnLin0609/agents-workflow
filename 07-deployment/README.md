# 07 · 部署(Deployment)

> **流程的第七步。** 輸入是通過測試的成果。核心觀念:**回滾計畫事先寫好,不要等出事才想**。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §11「階段七:部署」。
> 用法見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

## 這個資料夾的模板

| 模板 | 用途 | 維護方式 |
|------|------|----------|
| `runbook.md` | 怎麼部署、怎麼回滾、誰有權限、緊急聯絡人 | **活文件**——隨環境/流程變動更新 |
| `rollback-plan.md` | 出問題時的具體步驟,**部署前就寫好** | **活文件** |
| `release-notes.md` | 這次上線了什麼、修了什麼、已知問題 | 每次 release 一份 |

## 這一步要達成什麼(§11 的把關標準)

- **回滾計畫已寫好且可行** —— 在部署「之前」就寫,含明確的回滾觸發條件與步驟。
- **部署策略選定** —— 藍綠 / 金絲雀 / Feature Flag / Rolling,依風險選。
- **Production 部署需要 approval**(至少一位 senior 或 release manager)。
- **部署後有 smoke test** 自動確認基本功能還在。
- **避開危險窗口** —— 建議週二~週四 10:00–16:00;避開週五下午、週末、節日前與促銷 freeze 期。

### 部署策略(依風險選)

| 策略 | 說明 | 適合 |
|------|------|------|
| 藍綠 | 兩套環境切換,快速回滾 | 一般情況 |
| 金絲雀 | 先放 1%/5%/25%/100% | 高風險變更 |
| Feature Flag | 上線但用開關控制,可隨時關 | 逐步開放 / A B 測試 |
| Rolling | 逐台更新 | stateless 服務 |

---

## 操作流程(搭配 grill-me skill)

部署文件不是憑空寫,而是從實作與設計推出來,再用 grill-me **壓力測試回滾與失敗情境**。
grill-me 是 [@mattpocock](https://github.com/mattpocock/skills) 的 skill,安裝見根 README。

### 前置(artifact 交接)

> 部署前先確認測試通過。輸入包含 schema 變更(migration 順序很關鍵)與風險登記簿。

- 先給 Claude 輸入文件:
  ```
  先讀 docs/06-testing/<專案名>-test-report.md(確認可放行)、
  docs/03-design-detail/<專案名>-data-schema.md(migration)、
  docs/04-planning/<專案名>-risk-register.md。這是這次部署的依據。
  ```
- 把模板複製進 `docs/`:
  ```bash
  mkdir -p docs/07-deployment
  cp ~/dev-process/07-deployment/runbook.md       docs/07-deployment/<專案名>-runbook.md
  cp ~/dev-process/07-deployment/rollback-plan.md docs/07-deployment/<專案名>-rollback-plan.md
  cp ~/dev-process/07-deployment/release-notes.md docs/07-deployment/<專案名>-release-notes.md
  ```

### 第一步:讓 Claude 草擬 runbook + rollback,再用 grill-me 壓測

先請 Claude 出初稿:

```
根據設計(尤其 data-schema 的 migration)與風險登記簿,草擬:
1. 部署 runbook:步驟、依賴順序、需要的權限、smoke test 怎麼確認
2. 回滾計畫:什麼條件觸發回滾、回滾的具體步驟、資料怎麼處理
並建議這次該用哪種部署策略(藍綠/金絲雀/Feature Flag/Rolling)與理由。
先別寫進檔案,我們先討論。
```

拿到初稿後,用 grill-me 壓力測試:

```
/grill-me

針對剛剛的部署與回滾計畫訪談我,逼問失敗情境:
- migration 如果跑到一半失敗?資料怎麼辦?加欄位/部署/開新邏輯的順序對嗎?
- 回滾的觸發條件具體嗎?誰按下回滾?幾分鐘內?
- 部署過程哪一步最可能炸?那一步的 smoke test 抓得到嗎?
- 這個部署窗口安全嗎?有沒有撞到 freeze 期?
一次問一題,每題附上你建議的答案。
```

回答時的要點:

- **回滾要可執行,不是口號**。寫到「誰、什麼條件、哪些指令」的程度。
- **migration 順序**:通常是「加欄位(向下相容)→ 部署 → 開新邏輯」,別讓舊版本撞到新 schema。
- **smoke test 要對準最可能壞的那步**。
- **避開危險窗口**:週五下午、週末、freeze 期不要部署。

### 第二步:把對話填進部署文件

收斂後:

```
根據我們剛剛的對話,把 docs/07-deployment/<專案名>-runbook.md 與 rollback-plan.md 就地填好。
release-notes.md 填這次 release 的內容(上線什麼、修了什麼、已知問題)。
沒定的標 TBD,不要編。
```

> 進 context 的只有那幾份輸入文件 + 你複製進 `docs/` 的模板 + 對話。全域那份模板保持乾淨。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **訪談 / 壓測進行中** | 不要 `/clear` | 會弄丟剛逼出來的失敗情境與回滾觸發 |
| **對話很長、開始變慢** | 用 `/compact` | 壓縮但保留脈絡 |
| **文件落地、要去實際部署** | 可 `/clear` | 實際部署是另一件事;之後出狀況時讀 runbook/rollback 即可 |

呼應核心原則:**靠檔案交接,不靠對話記憶**。

> **runbook 與 rollback-plan 是活文件**:環境、權限、流程變了就回來更新,不是寫完歸檔。

---

## 完成標準(進入 `08-operations/` 前)

- [ ] `rollback-plan` 在部署前就寫好,含明確觸發條件與可執行步驟
- [ ] `runbook` 寫明部署步驟、權限、緊急聯絡人、smoke test
- [ ] 部署策略已選定且有理由
- [ ] Production 部署有 approval;部署後 smoke test 會自動跑
- [ ] `release-notes` 已填;部署排在安全窗口(非週五下午/週末/freeze)

全部打勾、實際部署完成後 → 進入第八步 `08-operations/`,設定監控、告警與事故處理。
