# 01 · 對齊需求(Requirements Alignment)

> **流程的第一步。** 這一步**只做功能面**的需求對齊——釐清要解決什麼問題、給誰用、
> 做到哪算成功。**技術選型與架構不在這一步**,留到第二步 `02-design-system/`。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §5「階段一:需求釐清」。

## 這個資料夾的模板

| 模板 | 用途 | 何時填 |
|------|------|--------|
| `requirements.md` | PRD 主文件(背景、Scope、使用者故事、限制、假設與風險) | 必填 |
| `user-persona.md` | 目標使用者輪廓 | 多數情況填 |
| `competitive-analysis.md` | 競品 / 現況分析 | 對話中有提到競品或現況才填 |

## 這一步要達成什麼

對齊到下面這幾件事就算完成(對應方法論 §5 的進入下一階段標準):

- **為什麼做** —— 描述問題本身,不是描述功能。
- **給誰用** —— 主要 / 次要使用者,他們現在怎麼解決。
- **做到哪算成功** —— 可衡量的成功指標,不只是「好用」。
- **Must / Nice-to-have / Out of scope** —— 尤其 out-of-scope 要明確,這欄最能防後期爭議。
- **關鍵使用者故事有驗收標準**。

---

## 操作流程(搭配 grill-me skill)

用 `grill-me` skill 讓 Claude Code 逐題追問,把你腦中模糊的需求逼出結構。
skill 裝在使用者層 `~/.claude/skills/grill-me/`(注意是**連字號** `grill-me`,不是底線)。

### 前置

> 假設這個流程包 clone 在全域(如 `~/dev-process`),`/grill-me` 已裝在 `~/.claude/skills/`。
> 用法見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

在你的開發專案根目錄:

- **開一輪乾淨的對話**(這一步從零開始,不要接在別的任務後面)。
- 確認 skill 可用:輸入 `/grill-me` 應該會觸發訪談。
- 把這一步要填的模板複製進專案 `docs/`(只複製要用的那份,避免雜訊進 context):
  ```bash
  mkdir -p docs/01-requirements
  cp ~/dev-process/01-requirements/requirements.md docs/01-requirements/<專案名>-requirements.md
  cp ~/dev-process/01-requirements/user-persona.md docs/01-requirements/<專案名>-user-persona.md
  ```

### 第一步:用 grill-me 對齊功能需求

複製下面這段,把括號內容換成你的專案:

```
/grill-me

我想做 [一句話描述產品或功能]。

請只針對「功能面需求」訪談我,涵蓋:
- 為什麼要做(要解決誰的什麼問題、現在怎麼解)
- 成功的衡量標準(可量化)
- Must-have / Nice-to-have
- 明確的 Out of scope(這次不做什麼)
- 關鍵使用者故事與驗收標準(Given/When/Then)

請「不要」討論技術選型、架構或實作方式——那是下一步。
一次問一題,每題附上你建議的答案。
```

回答時的要點:

- **一次只回一題**,跟著它的節奏走。
- **卡住或沒想法時**,直接說「就用你的建議」,讓它先填一個合理預設,之後再調。
- **別被帶進技術細節**。如果你自己開始講「這要用什麼資料庫」,提醒自己留到 `02-design`。
- **發散時收斂**:「先確定 must-have,其餘都丟到 nice-to-have」。

### 第二步:把對話填進需求文件

問答收斂、你覺得「需求講清楚了」之後,複製下面這段:

```
根據我們剛剛的對話,把 docs/01-requirements/<專案名>-requirements.md 這份文件填好
(它是需求文件模板,就地填寫)。沒談到的欄位標 TBD,不要自己編內容。
使用者輪廓填進 docs/01-requirements/<專案名>-user-persona.md。
```

如果剛剛的訪談有聊到競品或現況,先把模板複製進來再請 Claude 填:

```
cp ~/dev-process/01-requirements/competitive-analysis.md docs/01-requirements/<專案名>-competitive-analysis.md
```
```
剛剛有提到競品/現況,把 docs/01-requirements/<專案名>-competitive-analysis.md 填好。
```

> 進 context 的只有你複製進 `docs/` 的那幾份模板 + 對話。全域那份模板保持乾淨,
> 永遠是拿來複製的,不要就地填。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **訪談進行中** | 兩個都**不要用** | grill-me 靠完整對話脈絡推進,中途清掉會弄丟剛建立的共識 |
| **對話很長、開始變慢** | 用 `/compact`,不要 `/clear` | `/compact` 壓縮但保留脈絡;`/clear` 會整個清空 |
| **文件已寫到檔案、要進下一步前** | 先確認檔案存在,再 `/clear` | 下一步(`02-design`)的 context 來源是**已落地的需求文件**,不是這輪對話 |

這呼應一條核心原則:**階段之間靠檔案交接,不靠對話記憶**。
只要需求文件確實寫進了 `docs/01-requirements/`,你就可以放心 `/clear`,下一步重新讀那份檔案即可。

---

## 完成標準(進入 `02-design-system/` 前)

- [ ] `docs/01-requirements/<專案名>-requirements.md` 已填,且 Must / Nice-to-have / Out-of-scope 清楚
- [ ] 成功指標可衡量(不是「好用」這種)
- [ ] 關鍵使用者故事都有驗收標準
- [ ] 沒有未解的「這到底要不要做」
- [ ] (視需要)使用者輪廓、競品分析已填

全部打勾後 → `/clear` → 進入第二步 `02-design-system/`,在那裡讀本資料夾產出的需求文件,開始技術討論。
