# 05 · 實作(Implementation)

> **流程的第五步。** 輸入是任務清單。核心精神:**一次一個任務、小步前進、邊寫邊測、先求對再求好**。
> 這一步跟前面不同——不是一次跑完的訪談,是 task-breakdown 裡的任務**一個個來**的迴圈。
>
> 背後的方法論見 [`../software-development-guide.md`](../software-development-guide.md) §8「階段四:實作」、
> §10/§14 Code Review。用法見根目錄 [`../README.md`](../README.md)「如何在開發專案中使用」。

## 這個資料夾的模板

| 模板 | 用途 | 維護方式 |
|------|------|----------|
| `coding-standards.md` | 團隊共識的風格規範(由 linter/formatter 強制) | 一次性設定,專案層級 |
| `dev-notes-template.md` | 每個任務的開發筆記:遇到的坑、取捨、follow-up | 每個任務一份 |
| `review-checklist.md` | Code Review 時的檢查清單(對應 §14) | review 時對照 |

## 用到的兩個第三方 skill(@mattpocock)

| skill | 何時用 | 安裝 |
|-------|--------|------|
| `tdd` | 適合 TDD 的任務(純邏輯、公式、有已知正確輸出) | 見根 README「一次性設定」 |
| `grill-me` | 任務介面不確定時,先把介面與要測的行為討論清楚 | 見根 README |

## 這一步每個任務要達成什麼(§8 的 DoD)

- **小步前進**:每次只改一件事,能編譯、能跑、能測;不要一次寫五百行才第一次執行。
- **關鍵邏輯有測試**。
- **經常 commit**,訊息寫清楚。
- **任何「魔法」都要註解**——為什麼這樣寫,比寫了什麼更重要。
- **文件同步**:改到 API/schema 就回頭更新 `docs/03-design-detail/`。
- **PR 經過 review**(對照 review-checklist)。

---

## 前置(artifact 交接)

> 輸入是任務清單與設計。**先讓 Claude 讀它們**,別憑對話記憶。

- 先給 Claude 輸入文件:
  ```
  先讀 docs/04-planning/<專案名>-task-breakdown.md(任務與 DoD)
  以及 docs/02-design-system/、docs/03-design-detail/(設計依據)。
  ```
- **一次性**:把 coding-standards 填好並讓 linter/formatter 落地(風格交給工具,不靠人記):
  ```bash
  mkdir -p docs/05-implementation
  cp ~/dev-process/05-implementation/coding-standards.md  docs/05-implementation/<專案名>-coding-standards.md
  cp ~/dev-process/05-implementation/dev-notes-template.md docs/05-implementation/  # 之後每個任務複製一份
  cp ~/dev-process/05-implementation/review-checklist.md   docs/05-implementation/
  ```

---

## 操作流程:一次一個任務的迴圈

從 task-breakdown 取**下一個任務**(已按高風險優先排好序),對每個任務跑下面的 A→D。
**任務之間 `/clear`**——每個任務是獨立的,開新一輪只讀任務清單與相關程式碼。

### A. 取一個任務,判斷它的形狀

判斷這個任務適不適合 TDD:

- **適合 TDD**:純函式、公式、演算法、有已知正確輸出、介面小而清楚 → 走 B-1。
- **不適合 TDD**:純結構/設定/glue code、UI、正確性偏主觀 → 走 B-2。不要硬套 TDD 寫出
  「斷言常數等於自己」的廢測試。

### B-1. TDD 路線(搭配 `tdd` skill)

`tdd` 是 [@mattpocock](https://github.com/mattpocock/skills) 的 skill,red-green-refactor。觸發:

```
/tdd

任務:<貼 task-breakdown 裡這個任務 + DoD>
介面依據:docs/03-design-detail/<專案名>-api-spec.md(或相關設計)
請帶我用 TDD 完成這個任務。
```

照這個 skill 的核心紀律走(它會帶,但你要知道為什麼):

- **垂直切片,不要水平**:一個 test → 一個 impl → 重複。**絕對不要一次寫完所有 test 再寫 code**——
  那會寫出測「想像中行為」的爛測試。
- **測行為,不測實作**:test 走 public interface,內部重構不該弄壞它。
- **先對齊再動**:skill 會先問「public interface 長怎樣?哪些行為最該測?」——你核准了它才寫。
- **只在綠燈時 refactor**:紅燈時先求通過,別重構。
- **只寫剛好通過當前 test 的 code**,別預判未來的 test。

### B-2. 非 TDD 路線

- **小步前進**:每次改一件事,馬上跑起來看。
- **先求對,再求好**:第一版能動就好,確認思路正確後再重構;一開始就追求完美架構常是過度設計的開端。
- **關鍵邏輯補測試**,即使沒走完整 TDD。

### C. 收尾這個任務

- **經常 commit**,訊息清楚(出問題好回溯、好 review)。
- **魔法加註解**:不直觀的地方寫「為什麼」,不是「做了什麼」。
- **文件同步**:改到 API/schema → 回頭更新 `docs/03-design-detail/`,別讓文件與程式碼分家。
- **寫 dev-notes**:複製 `dev-notes-template.md` 成 `docs/05-implementation/<任務名>-dev-notes.md`,
  記這個任務遇到的坑、做了什麼取捨、有哪些 follow-up。

### D. Code Review(對應 §10/§14)

- 開 PR 前**自己先用 reviewer 視角看一遍**,對照 `review-checklist.md`。
- Review 重點抓人才抓得到的:設計、邊界與錯誤情境、安全、業務邏輯、命名、測試完整性。
- **別抓工具能抓的**(縮排、未使用變數、型別)——那些交給 linter/formatter/type checker。
- 抓「驚訝」與「沒寫的東西」:錯誤情境、邊界、併發、只測 happy path、敏感資料 log。

---

## context 管理:`/compact` 還是 `/clear`?

| 時機 | 做法 | 原因 |
|------|------|------|
| **同一個任務進行中** | 不要 `/clear` | TDD 的紅綠循環、剛建立的介面共識會被清掉 |
| **任務變很長、開始變慢** | 用 `/compact` | 壓縮但保留脈絡 |
| **一個任務完成、要做下一個** | `/clear` | 每個任務獨立;下一個任務的 context 來源是 task-breakdown + 相關程式碼 + 累積的 dev-notes |

呼應核心原則:**靠檔案交接,不靠對話記憶**。任務間靠 task-breakdown 與 dev-notes 串接,
所以每個任務做完都能放心 `/clear`。

---

## 完成標準(進入 `06-testing/` 前)

- [ ] task-breakdown 裡所有任務完成,且各自 DoD 達成
- [ ] 關鍵邏輯有測試,本機/CI 綠燈
- [ ] 改動到的 API/schema 文件已同步更新(`docs/03-design-detail/`)
- [ ] 每個任務都有一份 dev-notes
- [ ] 每個 PR 都經過 review(對照 review-checklist),沒有自我 merge

全部完成後 → 進入第六步 `06-testing/`,做更完整的測試(整合、E2E、效能、安全)與 QA sign-off。
