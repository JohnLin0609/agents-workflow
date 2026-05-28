# 06 · 測試(Testing)

> **流程的第六步。** 輸入是實作成果與設計文件(尤其 `edge-cases` 與需求的驗收標準)。
> 核心觀念:**沒有正確性,其他特性都沒意義**——這一步系統性地驗證它。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §9「階段五:測試」。
> 用法見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

## 這個資料夾的模板

| 模板 | 用途 | 維護方式 |
|------|------|----------|
| `test-plan.md` | 要測哪些層、用哪些環境、覆蓋目標、進入/退出標準 | 一次性(大改前重寫) |
| `test-cases.md` | 具體測試案例(含邊界與失敗情境) | 隨功能更新 |
| `test-report.md` | 一輪測試跑完的結果、缺陷分級、是否放行 | **執行後**才填 |

## 這一步要達成什麼(§9 的把關標準)

- **測試金字塔該有的層都意識到** —— 不一定每層都做,但要知道你跳過哪一層、它的風險是什麼。
- **所有自動化測試在 CI 跑**,失敗就擋 merge。
- **QA sign-off** —— 上線前有一道簽核;重大 release 前做完整迴歸。
- **缺陷分級清楚**(P0 擋上線 / P1 上線前修 / P2 排程 / P3 backlog)。

### 測試金字塔(挑你專案需要的層)

| 層 | 目標 | 何時跑 |
|----|------|--------|
| 單元 | 核心邏輯 ≥ 80% | 每次 commit / CI |
| 整合 | 模組間關鍵介面 | CI |
| 契約 | 服務間 API 契約 | CI |
| E2E | 主要 user journey | 每晚 / 上線前 |
| 效能 | 高流量端點 | 大改前 |
| 安全 | OWASP Top 10 | 上線前 |
| 探索式 / UAT | 預期外情境 / 使用者驗收 | 上線前 |

---

## 操作流程(搭配 grill-me skill)

跟規劃階段類似:**先讓 Claude 從設計與需求草擬測試計畫與案例,再用 grill-me 挑戰覆蓋缺口**。
grill-me 是 [@mattpocock](https://github.com/mattpocock/skills) 的 skill,安裝見根 README。

### 前置(artifact 交接)

> 測試案例不要憑空想——它們直接來自 `edge-cases` 與需求的驗收標準。

- 先給 Claude 輸入文件:
  ```
  先讀 docs/01-requirements/<專案名>-requirements.md(驗收標準)、
  docs/03-design-detail/<專案名>-edge-cases.md 與 api-spec.md,
  以及 docs/04-planning/ 的任務 DoD。這是這次測試的依據。
  ```
- 把模板複製進 `docs/`:
  ```bash
  mkdir -p docs/06-testing
  cp ~/dev-process/06-testing/test-plan.md   docs/06-testing/<專案名>-test-plan.md
  cp ~/dev-process/06-testing/test-cases.md  docs/06-testing/<專案名>-test-cases.md
  cp ~/dev-process/06-testing/test-report.md docs/06-testing/<專案名>-test-report.md
  ```

### 第一步:讓 Claude 草擬計畫與案例,再用 grill-me 挑戰

先請 Claude 出初稿:

```
根據需求的驗收標準與 edge-cases,草擬:
1. 測試計畫:這個專案需要金字塔的哪幾層、各層覆蓋目標、用哪些環境、進入/退出標準
2. 測試案例:把每條驗收標準與每個 edge case 轉成具體案例(Given/When/Then),含失敗情境
先別寫進檔案,我們先討論。
```

拿到初稿後,用 grill-me 戳它:

```
/grill-me

針對剛剛的測試計畫與案例訪談我:
- 金字塔哪一層被跳過了?跳過的風險是什麼?可以接受嗎?
- 有沒有只測了 happy path、漏掉的失敗/邊界情境?
- 安全(OWASP)、併發、效能熱路徑有沒有對應的測試?
- 哪些案例其實是在測實作細節、refactor 就會壞?
一次問一題,每題附上你建議的答案。
```

回答時的要點:

- **意識到你跳過什麼**。沒資源全測沒關係,但要知道「我沒測這層」以及它的風險,別假裝覆蓋了。
- **案例來自 artifact**:每條驗收標準、每個 edge case 都該有對應案例,不是另想一套。
- **別測實作細節**(呼應 tdd skill 的原則):測行為、走 public interface,refactor 不該弄壞。

### 第二步:把對話填進測試計畫與案例

收斂後:

```
根據我們剛剛的對話,把 docs/06-testing/<專案名>-test-plan.md 與 test-cases.md 就地填好。
test-report.md 先不填——那是等實際跑完測試才填的。
```

### 第三步:執行測試 → 填 test-report

實際跑測試(CI / 本機 / staging)後,把結果填進報告:

```
測試已經跑完,結果是 <貼結果或摘要>。
幫我填 docs/06-testing/<專案名>-test-report.md:通過/失敗、發現的缺陷依 P0–P3 分級、
是否建議放行。沒有的數據標 TBD,不要編。
```

> 進 context 的只有那幾份輸入文件 + 你複製進 `docs/` 的模板 + 對話。全域那份模板保持乾淨。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **訪談 / 設計案例進行中** | 不要 `/clear` | 會弄丟剛討論出的覆蓋取捨 |
| **對話很長、開始變慢** | 用 `/compact` | 壓縮但保留脈絡 |
| **計畫與案例已落地、要去跑測試** | 可 `/clear` | 跑測試是另一件事;之後填 report 時重新讀 test-plan 即可 |

呼應核心原則:**靠檔案交接,不靠對話記憶**。

---

## 完成標準(進入 `07-deployment/` 前)

- [ ] `test-plan` 寫明測哪些層、覆蓋目標、進入/退出標準
- [ ] `test-cases` 涵蓋所有驗收標準與 edge cases,不只 happy path
- [ ] 自動化測試在 CI 綠燈,失敗會擋 merge
- [ ] `test-report` 已填,缺陷依 P0–P3 分級;P0/P1 已處理或有明確決議
- [ ] QA sign-off(你自己或 QA 簽核可以放行)

全部打勾後 → `/clear` → 進入第七步 `07-deployment/`,準備部署手冊、release notes 與回滾計畫。
