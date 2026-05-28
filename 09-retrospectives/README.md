# 09 · 回顧與迭代(Retrospectives)

> **流程的最後一步,也是讓流程進化的引擎。** 核心原則:**action items 要有人、有期限、有追蹤——
> 沒這三項的回顧就是聊天**。最重要的習慣就是回顧;沒有回顧,流程不會進化。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §13「階段九:回顧與迭代」。
> 用法見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

## 這個資料夾的模板

| 模板 | 用途 | 維護方式 |
|------|------|----------|
| `retrospective.md` | 一次 Sprint / Project Retro 的記錄與 action items | 每次回顧一份 |
| `lessons-learned.md` | 經驗知識庫,新人必讀 | **累積文件**——持續加,不刪歷史 |

## 這一步要達成什麼(§13 的把關標準)

- **先 review 上次的 action items** —— 每次回顧第一件事,確認上次說要做的做了沒。
- **拿數據鋪墊** —— 用事實(交付、缺陷、事故、週期時間)起頭,不要憑感覺。
- **action items 三要素齊全**:負責人、期限、衡量成功。少一個就變廢話。
- **action items ≤ 5** —— 超過 5 個 = 一個都不會做。
- **durable 的學習進 lessons-learned**,讓下一輪 / 新人受惠。

---

## 操作流程(搭配 grill-me skill)

grill-me 是 [@mattpocock](https://github.com/mattpocock/skills) 的 skill,這裡用它**逼出誠實的反思**——
它會追問「為什麼」,不讓你停在表面。安裝見根 README。

### 前置(artifact 交接)

> 回顧要拿這一輪累積的 artifact 當數據,不是空談。

- 先給 Claude 輸入文件:
  ```
  先讀這一輪累積的 artifact 當數據:docs/05-implementation/ 的 dev-notes、
  docs/06-testing/<專案名>-test-report.md、docs/08-operations/ 的 incident-report(若有)、
  以及上一次的 docs/09-retrospectives/ retro 的 action items。
  ```
- 把模板複製進 `docs/`:
  ```bash
  mkdir -p docs/09-retrospectives
  cp ~/dev-process/09-retrospectives/retrospective.md  docs/09-retrospectives/<YYYY-MM-DD>-retro.md
  # lessons-learned 是累積文件:整個專案/團隊維護同一份,沒有就建一份
  test -f docs/09-retrospectives/lessons-learned.md || \
    cp ~/dev-process/09-retrospectives/lessons-learned.md docs/09-retrospectives/lessons-learned.md
  ```

### 第一步:先 review 上次 action items,再用 grill-me 反思

```
先列出上一次 retro 的 action items,逐一確認:做了沒?有效嗎?沒做的為什麼?
```

接著用 grill-me 逼出誠實反思:

```
/grill-me

根據這一輪的數據(dev-notes、test-report、incident),帶我做回顧:
- 哪些做得好、值得保留?(具體,不要「還不錯」)
- 哪些不順、拖慢了我們?根因是什麼(不是症狀)?
- 有哪些想試試看的改善?
針對最值得處理的 3–5 個深入追問「為什麼」,別讓我停在表面。
一次問一題,每題附上你建議的答案。
```

回答要點:

- **對事不對人**(blameless):問題出在系統與流程,不是某個人。
- **追根因不停在症狀**:「review 太慢」往下問為什麼——PR 太大?reviewer 太忙?
- **收斂**:討論最重要的 3–5 個就好,不要每件事都列 action。

### 第二步:寫 retro + 累積 lessons learned

```
根據我們剛剛的對話,把 docs/09-retrospectives/<YYYY-MM-DD>-retro.md 填好,
重點是 action items——每項都要有「負責人 / 期限 / 怎麼算做到」。最多 5 個。
另外,把這次值得長期記住的學習,追加(不要覆蓋)到 docs/09-retrospectives/lessons-learned.md。
```

> `lessons-learned.md` 是**累積文件**:只加不刪,保留歷史。它是新人加入時的必讀。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **回顧進行中** | 不要 `/clear` | 會弄丟剛逼出來的根因與共識 |
| **對話很長、開始變慢** | 用 `/compact` | 壓縮但保留脈絡 |
| **retro 與 lessons 已落地** | 可 `/clear` | 下一輪回顧重新讀 retro 的 action items 即可 |

呼應核心原則:**靠檔案交接,不靠對話記憶**。

---

## 完成標準

- [ ] 上一次的 action items 已逐一 review
- [ ] 這次的 action items 每項都有負責人、期限、衡量成功,且 ≤ 5 個
- [ ] 根因有追到(不是停在症狀),全程 blameless
- [ ] durable 的學習已追加到 `lessons-learned.md`

---

## 這是循環,不是終點

回顧完,action items 餵回下一輪的 `01-requirements/` 或 `04-planning/`;`risk-register` 與
`lessons-learned` 持續累積。這套流程本身也該被回顧——用一陣子後,依你的痛點增刪、改寫各階段的
劇本(下一步就是把跑順的劇本固化成 slash command)。
