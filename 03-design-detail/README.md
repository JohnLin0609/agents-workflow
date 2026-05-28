# 03 · 細節設計(Detailed Design)

> **流程的第三步。** 輸入是上一階段的系統設計。系統設計是**骨架**,細節設計是**肉**——
> 兩者不能矛盾,發現矛盾時回頭修系統設計。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §6「階段二:設計（細節設計）」。
> 用法(全域包 + `docs/` 產出)見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

## 這個資料夾的模板

| 模板 | 用途 | 何時填 |
|------|------|--------|
| `data-schema.md` | 表結構、欄位、型別、索引、關聯、migration 策略 | 有資料儲存就必填 |
| `api-spec.md` | 每個端點的路徑、方法、參數、回應、狀態碼 | 有對外/對內 API 就必填 |
| `error-handling.md` | 錯誤分類、錯誤碼規範、log 策略 | 必填 |
| `edge-cases.md` | 空值、極大/極小、併發、重試、超時、部分失敗 | 必填 |

## 這一步要達成什麼

對齊到下面這幾件事就算完成(對應方法論 §6 細節設計的把關標準):

- **細設與系統設計一致** —— 若有偏離,系統設計已同步更新,不是兩份文件各說各話。
- **介面契約雙方對齊** —— 跨模組的介面細設,呼叫端與被呼叫端認知一致。
- **邊界條件夠完整** —— 不是只列了 happy path;空值、極值、併發、重試、超時、部分失敗都想過。
- **錯誤處理有規範** —— 錯誤怎麼分類、錯誤碼怎麼編、什麼該 log、log 什麼。

---

## 操作流程(搭配 grill-me skill)

一樣用 [@mattpocock](https://github.com/mattpocock/skills) 的 `grill-me` skill 逐題追問。
這一階段 grill-me 最有價值的地方是**逼出邊界條件**——它會問你 happy path 以外的情況。
安裝見根 README「一次性設定」。

### 前置(artifact 交接)

> 這一步的輸入是系統設計。**先讓 Claude 讀那幾份設計文件**,別憑對話記憶。

在你的開發專案根目錄:

- **開一輪乾淨的對話**。
- 先給 Claude 輸入文件:
  ```
  先讀 docs/02-design-system/ 下的設計文件(domain-model、module-decomposition、
  tech-stack 等),這是這次細節設計的依據。
  ```
- 把這一步要填的模板複製進 `docs/`(只複製要用的那幾份):
  ```bash
  mkdir -p docs/03-design-detail
  cp ~/dev-process/03-design-detail/data-schema.md    docs/03-design-detail/<專案名>-data-schema.md
  cp ~/dev-process/03-design-detail/api-spec.md       docs/03-design-detail/<專案名>-api-spec.md
  cp ~/dev-process/03-design-detail/error-handling.md docs/03-design-detail/<專案名>-error-handling.md
  cp ~/dev-process/03-design-detail/edge-cases.md     docs/03-design-detail/<專案名>-edge-cases.md
  ```

### 第一步:用 grill-me 對齊細節設計

複製下面這段(假設 Claude 已讀過系統設計文件):

```
/grill-me

根據 docs/02-design-system/ 的系統設計,針對「細節設計」訪談我,涵蓋:
- 資料表:每張表的欄位、型別、索引、關聯、約束;migration 策略
- API 規格:每個端點的路徑、方法、參數、回應格式、狀態碼
- 錯誤處理:錯誤怎麼分類、錯誤碼怎麼編、什麼該 log、log 哪些欄位
- 邊界條件:空值/null、極大極小值、併發、重試、超時、部分失敗——逐一問我打算怎麼處理

請特別用力逼問邊界條件與失敗情境,別讓我只想到 happy path。
如果細節跟系統設計矛盾,直接指出來。
一次問一題,每題附上你建議的答案。
```

回答時的要點:

- **邊界條件是這一步的重頭戲**。grill-me 問「如果這個值是空的呢?」「同時兩個請求進來呢?」
  時,認真回答——這些就是上線後最容易炸的地方。
- **跨模組介面要雙向想**。一個 API 的細設,同時站在呼叫端與被呼叫端的角度檢查。
- **發現與系統設計矛盾**:回頭修 `docs/02-design-system/` 的文件,別讓骨架與肉分家。
- **別過度規格化**:還沒決定的、之後才知道的,標 TBD,不要硬編一個假精確的值。

### 第二步:把對話填進細設文件

問答收斂後,複製下面這段:

```
根據我們剛剛的對話,把 docs/03-design-detail/ 下對應的文件就地填好:
data-schema、api-spec、error-handling、edge-cases。
沒談到的欄位標 TBD,不要自己編內容。
edge-cases 要列完整,不要只有 happy path。
```

> 進 context 的只有系統設計文件 + 你複製進 `docs/` 的那幾份模板 + 對話。全域那份模板保持乾淨。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **訪談進行中** | 兩個都**不要用** | grill-me 靠完整對話脈絡推進,中途清掉會弄丟剛逼出來的邊界條件 |
| **對話很長、開始變慢** | 用 `/compact`,不要 `/clear` | `/compact` 壓縮但保留脈絡 |
| **文件已寫到檔案、要進下一步前** | 先確認檔案存在,再 `/clear` | 下一步(`04-planning`)的 context 來源是**已落地的設計文件**,不是這輪對話 |

呼應核心原則:**階段之間靠檔案交接,不靠對話記憶**。細設文件確實寫進 `docs/03-design-detail/` 後,
就能放心 `/clear`,下一步重新讀那幾份檔案即可。

---

## 完成標準(進入 `04-planning/` 前)

- [ ] `data-schema`(如有資料)、`api-spec`(如有 API)、`error-handling`、`edge-cases` 已填
- [ ] 細設與 `docs/02-design-system/` 的系統設計一致(若偏離,系統設計已同步更新)
- [ ] 跨模組介面契約呼叫雙方認知一致
- [ ] 邊界條件清單完整,涵蓋空值、極值、併發、重試、超時、部分失敗,不只 happy path

全部打勾後 → `/clear` → 進入第四步 `04-planning/`,在那裡讀設計文件,把工作拆成可獨立完成、
可驗證的小任務並排程。
