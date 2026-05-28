# Coding Standards(程式碼規範)

> **目的**:讓團隊寫出風格一致、可讀、可維護的程式碼。
> **負責人**:技術 lead
> **Review**:全體工程師(共識,不是強推)
> **核心原則**:**能被工具自動檢查的,不要靠人記**——把規範寫進 linter / formatter / pre-commit hook,而不是寫成 wiki。

---

## 1. 自動化工具(優先)

> 這份文件絕大部分內容應該被工具強制執行,不靠人自律。

| 用途 | 工具 | 範例 |
|------|------|------|
| Formatter | Prettier / Black / gofmt / rustfmt | 統一縮排、引號、換行 |
| Linter | ESLint / Pylint / golangci-lint | 抓常見錯誤與風格 |
| Type Check | TypeScript / mypy | 型別檢查 |
| Import 排序 | isort / eslint-plugin-import | 統一 import 順序 |
| Pre-commit | husky / pre-commit | commit 前自動跑 |
| CI 強制 | GitHub Actions | PR 必須通過才能 merge |

**設定檔位置**:`/.prettierrc`、`/.eslintrc.js`、`/pyproject.toml` 等,都進版控。

## 2. 命名

### 2.1 通用原則
- **可讀性 > 簡短**:`userOrderCount` 好過 `uoc`
- **避免縮寫**,除非業界通用(URL、ID、API)
- **動詞開頭給函式**(`getUser`、`createOrder`),名詞給變數(`user`、`order`)
- **布林值用 is/has/can 開頭**(`isActive`、`hasPermission`、`canEdit`)
- **避免否定命名**:`isEnabled` 好過 `isNotDisabled`

### 2.2 各語言慣例
| 元素 | JavaScript/TS | Python | Go |
|------|--------------|--------|------|
| 變數 / 函式 | camelCase | snake_case | camelCase / PascalCase(公開) |
| 類別 | PascalCase | PascalCase | PascalCase |
| 常數 | UPPER_SNAKE | UPPER_SNAKE | UpperCamel(公開) |
| 檔名 | kebab-case.ts | snake_case.py | snake_case.go |

### 2.3 業務語彙統一
- 跟 domain-model.md 的 glossary 對齊
- 同一概念用同一字:不要混用 `customer` / `client` / `user`

## 3. 檔案與目錄結構

### 3.1 範例(可依專案調整)

```
src/
├── modules/
│   ├── order/
│   │   ├── order.controller.ts
│   │   ├── order.service.ts
│   │   ├── order.repository.ts
│   │   ├── order.types.ts
│   │   └── __tests__/
│   └── payment/
├── shared/           # 跨模組共用
│   ├── errors/
│   ├── utils/
│   └── middleware/
└── config/
```

### 3.2 規則
- 一個檔案一個主要 export
- 測試與被測檔案放近(`__tests__/` 或同目錄 `*.test.ts`)
- 不要有「misc」、「utils」、「helpers」黑洞資料夾——名稱要說明內容

## 4. 函式與類別

- **單一職責**:一個函式做一件事
- **長度**:函式 > 50 行、類別 > 300 行,考慮拆分
- **參數**:超過 3 個改用 options object
- **避免 boolean 旗標參數**:`createUser(name, true)` → 改用兩個函式或 enum
- **純函式優先**:輸入相同永遠輸出相同,沒副作用
- **早返回**(early return),減少巢狀

```typescript
// ✗ 巢狀地獄
function process(user) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        // 主邏輯在這裡
      }
    }
  }
}

// ✓ 早返回
function process(user) {
  if (!user) return;
  if (!user.isActive) return;
  if (!user.hasPermission) return;
  // 主邏輯在這裡
}
```

## 5. 註解

### 5.1 寫什麼
- **為什麼**(why),不是「做什麼」(what)
- 業務規則來源、特殊處理原因、已知限制
- TODO / FIXME / HACK 要附票號

### 5.2 不寫什麼
- 重複程式碼意思的廢話
- 過時的程式碼註解掉(刪掉就好,版控會留)

```typescript
// ✗ 廢話
i++;  // i 加 1

// ✓ 解釋為什麼
// 客服反映訂單 ID 不能含 0,因為 POS 系統會吃掉
const orderId = generateId().replace(/0/g, '1');
```

### 5.3 文件註解(JSDoc / docstring)
- 公開 API、複雜函式必寫
- 包含:用途、參數、回傳、可能拋出的錯誤、範例

## 6. 錯誤處理

- 對應 error-handling.md
- 不吞錯:`catch (e) {}` 是反模式
- 自訂 exception 類別對應錯誤碼
- 不要用 exception 當 control flow

## 7. 非同步

- **統一風格**:async/await 優先於 then chain
- **不要忘記 await**:孤立的 promise 是常見 bug
- **錯誤處理**:async 函式內必須 try/catch 或讓呼叫端處理
- **避免 race condition**:並行操作用 Promise.all / allSettled

## 8. 測試規範

- 對應 test-plan.md
- 命名:`should_<behavior>_when_<condition>`
- AAA pattern:Arrange / Act / Assert
- 一個 test case 測一件事
- 不要測私有方法,測公開行為
- Mock 第三方,不要 mock 自己的 code(信號:設計可能有問題)

## 9. Git / Commit

### 9.1 Branch 命名
- `feature/ORD-001-create-order-api`
- `fix/ORD-042-stock-check-race`
- `chore/upgrade-node-18`

### 9.2 Commit message(Conventional Commits)
```
<type>(<scope>): <subject>

<body>

<footer>
```

| Type | 用途 |
|------|------|
| feat | 新功能 |
| fix | 修 bug |
| refactor | 重構,不改行為 |
| docs | 文件 |
| test | 測試 |
| chore | 雜項 |
| perf | 效能 |

範例:
```
feat(order): add idempotency key to create endpoint

POS system sometimes double-sends requests due to network
glitches. Adds X-Idempotency-Key header support.

Refs: ORD-042
```

### 9.3 PR 規範
- 標題清楚、含票號
- 描述包含:做了什麼、為什麼、怎麼測
- 一個 PR 一個目的(不要把重構與新功能混在一起)
- 大 PR(> 500 行)考慮拆分

## 10. 安全與效能

### 10.1 安全紅線(永遠不能違反)
- 不在 log 印密碼、token、信用卡
- 不把 secret 寫進程式碼(用環境變數 / secret manager)
- 不信任前端傳的權限資訊,後端再驗證
- SQL 用參數化查詢

### 10.2 效能注意
- 避免迴圈中查 DB(N+1)
- 大集合用 stream / generator
- 慎用 sync 操作阻塞 event loop

## 11. 例外情況

- 規範與現實衝突時,**先討論再例外**
- 例外要有註解說明原因
- 累積的例外要定期 review,可能規範本身要調整

## 12. 變更紀錄

| 日期 | 變更 | 原因 |
|------|------|------|
| YYYY-MM-DD | 初版 | |
