# Dev Notes 模板

> **目的**:記錄這個任務做的時候遇到什麼、取捨什麼、留下什麼。
> **負責人**:該任務的實作者
> **何時寫**:任務做完、開 PR 時順手寫
> **長度**:預計 5–15 分鐘可讀完。寫太長沒人看,寫太短沒價值。

---

# [票號] - [任務標題]

**作者**:
**日期**:YYYY-MM-DD
**相關 PR**:#xxx
**狀態**:✅ Done / 🚧 Partial / 🔄 Needs Follow-up

---

## TL;DR(一段話總結)
> 這個任務做了什麼,最後是什麼結果,有什麼要注意。

例:實作 POST /orders 端點。採用 idempotency key 防止重複下單,但有個 known issue:超過 24h 的 key 會被清掉,理論上不影響(client 不該那麼久後重試)。

---

## 做了什麼

- 新增 OrderController.create()
- 新增 OrderService.createOrder() 業務邏輯
- 整合 Inventory Service 庫存檢查
- 加上 idempotency middleware

---

## 取捨與決策

> 列出實作過程中做的非顯而易見的決定,以及為什麼。

### 決策 1:Idempotency key 存 Redis 而非 DB
- **為什麼**:Redis TTL 機制方便,效能好
- **代價**:Redis 掛掉時 idempotency 失效(降級為一般行為)
- **替代方案**:存 DB 的 idempotency 表 — 較慢但更可靠

### 決策 2:庫存檢查同步而非非同步
- **為什麼**:使用者需要立即知道是否下單成功
- **代價**:庫存服務慢時連帶下單慢
- **緩解**:庫存服務有 200ms timeout,逾時當作不可用

---

## 遇到的坑

> 別人之後可能會踩到的。寫下來省下別人重新踩的時間。

1. **Inventory API 的 stock 欄位是 string 不是 number**
   - 第三方 API 文件沒寫,實測才發現
   - 加了型別轉換,有測試覆蓋

2. **TypeORM 的 `@Transaction` decorator 在 NestJS Guard 內失效**
   - 原因:Guard 比 Interceptor 早跑,Transaction context 還沒建立
   - 解法:把交易邏輯移到 Service 層

---

## 偏離設計文件的地方

> 如果有,要列出,並更新設計文件。

- **原設計**:訂單 ID 用 auto-increment
- **實作**:改用 UUID
- **原因**:auto-increment 在分散式環境有競爭問題,且暴露業務規模
- **動作**:✅ 已更新 data-schema.md

---

## 測試覆蓋

| 類型 | 覆蓋率 / 案例數 |
|------|---------------|
| 單元測試 | 92% / 18 cases |
| 整合測試 | Happy path + 3 個錯誤情境 |
| E2E | 未寫(由 QA 補) |

**未涵蓋的情境**(已知,風險可接受):
- 並發超過 1000 req/s 的行為(交給壓測)

---

## Follow-up / Tech Debt

> 這次沒做,但之後要做的。**有票號,不要只有 TODO 註解**。

- [ ] [ORD-099] Idempotency 改存 DB(若可靠性要求提升)
- [ ] [ORD-100] 訂單 ID 加上 prefix(`ord_xxxx`)以利區分

---

## 部署注意事項

> 若有特殊步驟,寫清楚給部署的人。

- 需先跑 migration `20260528_create_orders_table`
- 需要新環境變數:`INVENTORY_API_TIMEOUT=200`
- 需要 Redis(若未就緒,部署會 fail-fast)
- 可滾動部署(無 breaking change)

---

## 相關連結
- 需求文件:[連結]
- 設計文件:[連結]
- 設計討論紀錄:[連結]
- 相關 issue:[連結]
