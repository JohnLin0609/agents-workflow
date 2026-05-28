# 02 · 系統設計(System Design)

> **流程的第二步。** 輸入是上一階段落地的需求文件,這裡開始談**技術與架構**。
> 核心精神:**探索多個方案、做 trade-off**,不是直接給一個答案。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §6「階段二:設計」。
> 用法(全域包 + `docs/` 產出)見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

## 這個資料夾的模板

| 模板 | 用途 | 何時填 |
|------|------|--------|
| `domain-model.md` | 核心實體、關係、業務規則 | 必填 |
| `module-decomposition.md` | 系統由哪些模組組成、職責、邊界 | 必填 |
| `data-flow.md` | 資料從哪來、經過什麼、到哪去 | 多數情況填 |
| `interfaces.md` | 模組間 / 對外的介面契約(高層次) | 多數情況填 |
| `tech-stack.md` | 技術選型,**重點是為什麼選 + 考慮過什麼替代方案** | 必填 |

## 這一步要達成什麼

對齊到下面這幾件事就算完成(對應方法論 §6 的把關標準):

- **領域模型清楚** —— 有哪些「東西」、彼此什麼關係、有哪些業務規則。
- **模組邊界清楚** —— 每個模組的職責;問自己「換掉這個模組要動到多少別的」。
- **技術選型有理由** —— 不是「我熟所以選這個」,而是講得出為什麼、比較過哪些替代方案。
- **主要技術風險已識別**,有初步緩解方向。
- **沒有明顯過度設計** —— 只有一個實作就先抽 interface、為想像中的未來加抽象層,都要被點出來。
- **與需求文件不矛盾** —— 設計是骨架,要長在需求上。

---

## 操作流程(搭配 grill-me skill)

一樣用 [@mattpocock](https://github.com/mattpocock/skills) 的 `grill-me` skill 逐題追問,
把架構決策一個個逼出來、挑戰假設。安裝見根 README「一次性設定」。

### 前置(artifact 交接)

> 這一步的輸入是上一階段的產出。**先讓 Claude 讀那份需求文件**,別憑對話記憶——
> 上一輪很可能已經 `/clear` 過了。

在你的開發專案根目錄:

- **開一輪乾淨的對話**。
- 先給 Claude 輸入文件:
  ```
  先讀 docs/01-requirements/<專案名>-requirements.md,這是這次系統設計的依據。
  ```
- 把這一步要填的設計模板複製進 `docs/`(只複製要用的那幾份):
  ```bash
  mkdir -p docs/02-design-system
  cp ~/dev-process/02-design-system/domain-model.md         docs/02-design-system/<專案名>-domain-model.md
  cp ~/dev-process/02-design-system/module-decomposition.md docs/02-design-system/<專案名>-module-decomposition.md
  cp ~/dev-process/02-design-system/tech-stack.md           docs/02-design-system/<專案名>-tech-stack.md
  # data-flow.md、interfaces.md 視需要再複製
  ```

### 第一步:用 grill-me 對齊系統設計

複製下面這段(假設 Claude 已讀過需求文件):

```
/grill-me

根據 docs/01-requirements/<專案名>-requirements.md 的需求,針對「系統設計」訪談我,涵蓋:
- 領域模型:有哪些核心實體、彼此什麼關係、有哪些業務規則
- 模組切分:系統分成哪些模組、各自職責、邊界在哪(換掉一個要動多少別的)
- 資料流:資料從哪進來、經過哪些處理、到哪裡去
- 介面契約:模組之間、以及對外的介面長什麼樣(高層次)
- 技術選型:選什麼,以及「為什麼選」「考慮過哪些替代方案」「trade-off 是什麼」

請主動挑戰我的假設,並在我有過度設計傾向時點出來
(例如只有一個實作就想抽 interface、為想像中的未來加抽象層)。
一次問一題,每題附上你建議的答案。
```

回答時的要點:

- **先模型、後技術**。先把領域與模組邊界講清楚,技術選型自然會收斂;一開始就跳到「用什麼框架」
  容易本末倒置。
- **選型要講理由**。每個技術決策都問自己:為什麼是它?比較過什麼?最差情況是什麼?
- **歡迎它戳你**。grill-me 點出過度設計或矛盾時,別急著辯護——那正是這一步的價值。
- **發現與需求矛盾**:回頭修需求文件,別讓設計與需求分家。

### 第二步:把對話填進設計文件

問答收斂後,複製下面這段:

```
根據我們剛剛的對話,把 docs/02-design-system/ 下對應的設計文件就地填好:
domain-model、module-decomposition、tech-stack(以及有複製進來的 data-flow、interfaces)。
沒談到的欄位標 TBD,不要自己編內容。
tech-stack 一定要寫清楚:選了什麼、為什麼選、考慮過哪些替代方案、trade-off。
```

> 進 context 的只有需求文件 + 你複製進 `docs/` 的那幾份模板 + 對話。全域那份模板保持乾淨。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **訪談進行中** | 兩個都**不要用** | grill-me 靠完整對話脈絡推進,中途清掉會弄丟剛建立的設計共識 |
| **對話很長、開始變慢** | 用 `/compact`,不要 `/clear` | 設計討論通常比需求更長;`/compact` 壓縮但保留脈絡 |
| **5 份文件已寫到檔案、要進下一步前** | 先確認檔案存在,再 `/clear` | 下一步(`03-design-detail`)的 context 來源是**已落地的設計文件**,不是這輪對話 |

呼應核心原則:**階段之間靠檔案交接,不靠對話記憶**。設計文件確實寫進 `docs/02-design-system/` 後,
就能放心 `/clear`,下一步重新讀那幾份檔案即可。

---

## 完成標準(進入 `03-design-detail/` 前)

- [ ] `domain-model`、`module-decomposition`、`tech-stack` 已填(視需要含 `data-flow`、`interfaces`)
- [ ] 技術選型寫清楚理由與替代方案,不是「我熟所以選」
- [ ] 主要技術風險已識別,有初步緩解方向
- [ ] 沒有明顯過度設計(抽象層都對應到當下的真實需求)
- [ ] 設計與 `docs/01-requirements/` 的需求文件不矛盾(若有偏離,需求文件已同步更新)

全部打勾後 → `/clear` → 進入第三步 `03-design-detail/`,在那裡讀本階段產出的設計文件,展開細節設計
(資料表、API 規格、錯誤處理、邊界條件)。
