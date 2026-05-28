# 04 · 拆解與排程(Planning)

> **流程的第四步。** 輸入是設計文件。把工作拆成**可獨立完成、可驗證**的小任務並排程。
> 核心精神:**高風險的先做**,不要把簡單任務排前面只為了「先看到進度」。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §7「階段三:拆解與排程」。
> 用法(全域包 + `docs/` 產出)見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

## 這個資料夾的模板

| 模板 | 用途 | 維護方式 |
|------|------|----------|
| `task-breakdown.md` | 所有任務的清單、依賴關係、完成定義(DoD) | 一次性(進實作後以 issue tracker 為準) |
| `milestones-timeline.md` | 主要里程碑、預估完成日、關鍵交付物 | 一次性 |
| `risk-register.md` | 風險、影響、機率、緩解策略 | **活文件**——持續維護,不是寫完就丟 |

## 這一步要達成什麼

對齊到下面這幾件事就算完成(對應方法論 §7 的把關標準):

- **所有任務都拆到 1–2 天等級** —— 太大代表還沒拆夠細,通常意味著風險點還沒被識別出來。
- **每個任務有明確的 DoD** —— 程式寫完不算;要含測試、文件、review 通過。
- **任務依賴關係標出來**。
- **高風險、高不確定性的排前面** —— 會讓整個方案失敗的假設,放最前面用 spike/POC 驗證。
- **主要風險已識別**,有緩解策略,寫進 risk-register。

---

## 操作流程(搭配 grill-me skill)

這一階段跟前面略不同:**先讓 Claude 從設計文件草擬一份拆解,再用 grill-me 挑戰它**
(任務是不是太大?最危險的假設是什麼?DoD 是什麼?)。
grill-me 是 [@mattpocock](https://github.com/mattpocock/skills) 的 skill,安裝見根 README。

### 前置(artifact 交接)

> 輸入是設計文件。**先讓 Claude 讀設計**,別憑對話記憶。

在你的開發專案根目錄:

- **開一輪乾淨的對話**。
- 先給 Claude 輸入文件:
  ```
  先讀 docs/02-design-system/ 與 docs/03-design-detail/ 的設計文件,這是這次拆解排程的依據。
  ```
- 把模板複製進 `docs/`:
  ```bash
  mkdir -p docs/04-planning
  cp ~/dev-process/04-planning/task-breakdown.md      docs/04-planning/<專案名>-task-breakdown.md
  cp ~/dev-process/04-planning/milestones-timeline.md docs/04-planning/<專案名>-milestones-timeline.md
  cp ~/dev-process/04-planning/risk-register.md       docs/04-planning/<專案名>-risk-register.md
  ```

### 第一步:讓 Claude 草擬拆解,再用 grill-me 挑戰

先請 Claude 出初稿:

```
根據設計文件,草擬一份任務拆解:把工作拆成可獨立完成、可驗證的小任務,
每個任務 1–2 天可完成,標出依賴關係,並給每個任務一句 DoD。先別寫進檔案,我們先討論。
```

拿到初稿後,用 grill-me 戳它:

```
/grill-me

針對剛剛這份任務拆解訪談我,逐一確認:
- 哪些任務其實還太大(超過 1–2 天)?該再拆嗎?
- 最危險、最不確定的假設是什麼?那塊有沒有排在最前面用 spike/POC 驗證?
- 排序有沒有掉進「簡單的先做、難的拖到最後」的反模式?
- 每個任務的 DoD 夠具體嗎(含測試、文件、review)?
- 有哪些風險該進 risk-register(影響、機率、緩解)?

一次問一題,每題附上你建議的答案。
```

回答時的要點:

- **太大就拆**。一個任務講不清楚 DoD,通常就是還太大。
- **風險排前面**。問自己「哪個假設一旦不成立,整個方案要重來?」那個先驗證。
- **抵抗『先做簡單的』誘惑**。高風險拖到最後炸掉,整個排程都要重排。
- **估算重點不是精準**,是讓你對「大小」有概念;用 S/M/L 或人天都行。

### 第二步:把對話填進規劃文件

收斂後,複製下面這段:

```
根據我們剛剛的對話,把 docs/04-planning/ 下的文件就地填好:
task-breakdown(任務、依賴、DoD)、milestones-timeline、risk-register。
高風險任務排在前面。沒定的標 TBD,不要自己編日期或數字。
```

> 進 context 的只有設計文件 + 你複製進 `docs/` 的那幾份模板 + 對話。全域那份模板保持乾淨。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **訪談進行中** | 兩個都**不要用** | grill-me 靠完整對話脈絡推進,中途清掉會弄丟剛排好的優先序與風險 |
| **對話很長、開始變慢** | 用 `/compact`,不要 `/clear` | `/compact` 壓縮但保留脈絡 |
| **文件已寫到檔案、要進下一步前** | 先確認檔案存在,再 `/clear` | 下一步(`05-implementation`)的 context 來源是**已落地的任務清單**,不是這輪對話 |

呼應核心原則:**階段之間靠檔案交接,不靠對話記憶**。

> **例外:risk-register 是活文件**。它不是寫完歸檔——後面實作、測試、上線過程中發現新風險,
> 都要回來更新它。

---

## 完成標準(進入 `05-implementation/` 前)

- [ ] 所有任務都拆到 1–2 天等級,且各有明確 DoD
- [ ] 任務依賴關係已標出
- [ ] 高風險 / 高不確定性任務排在前面(關鍵假設有 spike/POC)
- [ ] 主要風險已進 `risk-register`,各有緩解策略
- [ ] 團隊(或你自己)對「每個任務做完是什麼樣子」有共識

全部打勾後 → `/clear` → 進入第五步 `05-implementation/`,一次取一個任務,小步前進、邊寫邊測。
