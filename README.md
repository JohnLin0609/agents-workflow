# 開發流程工作包(完整版)

> 一套可操作的軟體開發流程。拆成 9 個階段資料夾,從需求釐清到上線後回顧。
>
> - **每個階段資料夾**內含該階段的文件模板(共 27 份),以及一份 `README.md`——
>   該階段的**操作劇本**:怎麼一步步驅動 Claude Code 跑完這個階段、建議的 prompt 話術、
>   context 管理建議。先讀該資料夾的 `README.md` 再動手。
> - **`software-development-guide.md`** 是這套流程背後的**方法論**(觀念、原則、把關、
>   反模式)。階段劇本講「怎麼做」,方法論講「為什麼這樣做」。

## 結構

```
.
├── 01-requirements/           # 需求釐清
│   ├── requirements.md         # PRD(主文件)
│   ├── user-persona.md         # 使用者輪廓
│   └── competitive-analysis.md # 競品/現況分析
│
├── 02-design-system/          # 系統設計
│   ├── domain-model.md         # 領域模型
│   ├── module-decomposition.md # 模組切分
│   ├── data-flow.md            # 資料流
│   ├── interfaces.md           # 介面定義
│   └── tech-stack.md           # 技術選型(含 ADR)
│
├── 03-design-detail/          # 細節設計
│   ├── data-schema.md          # 資料表
│   ├── api-spec.md             # API 規格
│   ├── error-handling.md       # 錯誤處理
│   └── edge-cases.md           # 邊界條件
│
├── 04-planning/               # 拆解與排程
│   ├── task-breakdown.md       # 任務拆解
│   ├── milestones-timeline.md  # 里程碑與時程
│   └── risk-register.md        # 風險登記簿
│
├── 05-implementation/         # 實作
│   ├── coding-standards.md     # 程式碼規範
│   ├── review-checklist.md     # Code Review Checklist
│   └── dev-notes-template.md   # 開發筆記模板
│
├── 06-testing/                # 測試
│   ├── test-plan.md            # 測試計畫
│   ├── test-cases.md           # 測試案例
│   └── test-report.md          # 測試報告
│
├── 07-deployment/             # 部署
│   ├── runbook.md              # 部署手冊
│   ├── release-notes.md        # Release Notes
│   └── rollback-plan.md        # 回滾計畫
│
├── 08-operations/             # 監控與維運
│   ├── monitoring-dashboards.md # 監控儀表板
│   ├── alert-runbook.md        # 告警手冊
│   └── incident-report.md      # 事故報告 / Postmortem
│
└── 09-retrospectives/         # 回顧與學習
    ├── retrospective.md        # Sprint / Project Retro
    └── lessons-learned.md      # 經驗知識庫
```

## 對應的開發流程階段

| 階段 | 資料夾 | 把關標準 |
|------|--------|---------|
| 需求釐清 | 01-requirements/ | Stakeholder 共識、must/nice 清楚、指標可衡量 |
| 系統設計 | 02-design-system/ | Reviewer 簽核、選型有理由、風險已識別 |
| 細節設計 | 03-design-detail/ | Reviewer 簽核、介面對齊、邊界完整 |
| 拆解與排程 | 04-planning/ | 任務 1-2 天等級、高風險排前面 |
| 實作 | 05-implementation/ | Lint/test 通過、PR review、文件同步 |
| 測試 | 06-testing/ | 進入/退出標準達成、QA sign-off |
| 部署 | 07-deployment/ | 部署前 checklist、有 rollback 計畫 |
| 維運 | 08-operations/ | 監控就緒、告警有 runbook |
| 回顧 | 09-retrospectives/ | Action 有人有期限、累積到 lessons learned |

## 如何在開發專案中使用(保持 context 乾淨)

核心原則:**操作劇本是給人讀的,不要進 Claude 的 context 窗口**。所以這個流程包
**不放進你的開發專案**,而是當作全域參考。三樣東西分三個地方:

| 東西 | 放哪 | 誰讀 |
|------|------|------|
| 這個流程包(劇本 + 方法論 + 模板) | 全域 clone 一份,如 `~/dev-process` | **人**讀劇本、照著貼 prompt |
| `grill-me` skill | 使用者層 `~/.claude/skills/` | Claude(被 `/grill-me` 觸發) |
| 你填出來的文件(artifacts) | 你專案的 `docs/<階段>/` | 人 + Claude,跟專案一起 commit |

### 一次性設定

```bash
git clone <repo> ~/dev-process
```

本流程用到兩個第三方 skill,作者都是 [@mattpocock](https://github.com/mattpocock/skills),
**不隨本流程包散佈**——請從上游自行安裝到使用者層,讓它們在所有專案可用:

- `grill-me`(需求/設計/規劃階段的訪談):只有單一 `SKILL.md`,curl 即可。
- `tdd`(實作階段的 red-green-refactor):**含多個 spoke 檔案**,要整個資料夾複製。

```bash
# grill-me:單檔
mkdir -p ~/.claude/skills/grill-me
curl -fsSL https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/grill-me/SKILL.md \
  -o ~/.claude/skills/grill-me/SKILL.md

# tdd:整個資料夾(SKILL.md + tests/mocking/deep-modules/interface-design/refactoring 等 spoke 檔)
git clone --depth 1 https://github.com/mattpocock/skills /tmp/mp-skills
cp -r /tmp/mp-skills/skills/engineering/tdd ~/.claude/skills/tdd
rm -rf /tmp/mp-skills
```

> 上游來源:<https://github.com/mattpocock/skills>
> (本 repo 的 `skills-lock.json` 也記錄了這兩個 skill 的出處與雜湊。)

### 每跑一個階段

1. **人**讀對應資料夾的 `README.md` 劇本(例:`~/dev-process/01-requirements/README.md`)。
2. 把**那一份**要填的模板複製進專案 `docs/`,以專案名命名:
   ```bash
   mkdir -p docs/01-requirements
   cp ~/dev-process/01-requirements/requirements.md docs/01-requirements/my-project-requirements.md
   ```
3. 照劇本在 Claude Code 裡操作。叫 Claude 填文件時,指向 `docs/...` 那份。

這樣進 context 的只有**當下那一份模板 + 你的對話**——劇本、方法論、其他階段的模板
都不會汙染。你不需要把整包 clone 進專案,也不需要只 clone 模板。

## 使用建議

### 1. 不是每份都要寫滿
依專案規模裁切:

- **個人小專案 / Prototype**:requirements 寫一段、task-breakdown 列待辦,夠了。
- **中型專案**:加上 domain-model、api-spec、edge-cases、risk-register、test-plan、runbook。
- **大型 / 對外產品 / 多人團隊**:全套都寫。

判斷標準:**「不寫這份,會不會有人重複問同一個問題?」會 → 寫;不會 → 跳過。**

### 2. 文件分三類,維護策略不同

| 類型 | 範例 | 維護 |
|------|------|------|
| 一次性文件 | Requirements, Test Report, Incident Report | 寫完歸檔,出新版時重寫 |
| 活文件 | API Spec, Schema, Runbook, Risk Register | 隨程式碼一起改,過時即失效 |
| 累積文件 | Lessons Learned, ADR | 持續加,不刪除歷史 |

### 3. 自動化 > 文件
能被工具自動檢查的,優先用工具(linter、CI、API spec 自動產生),而不是寫進文件靠人記。

### 4. Review 不是儀式
每份文件都標明 reviewer。Reviewer 要實際讀、實際挑戰,不是掛名簽核。

### 5. 文件壞了比沒文件糟
過時的文件會誤導,定期 review。每份文件頂部有「最後更新」與「變更紀錄」,確實維護。

## 三個提醒

**第一**:這套是「資源豐富」版本的完整清單。實際使用時依專案規模裁切,不要全套硬套小專案。

**第二**:文件是降低風險的成本,不是品質的保證。真正讓品質高的是清楚的思考、誠實的回顧、把人當人對待的協作文化。流程只是讓這些事比較容易發生。

**第三**:這套是起點,不是終點。團隊用一陣子,根據自己的痛點調整、增刪、改寫。沒有一套通用模板適合所有團隊。
