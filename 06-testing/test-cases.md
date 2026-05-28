# 測試案例(Test Cases)

> **目的**:把要測什麼具體寫出來,可被執行、可被追蹤。
> **負責人**:QA(主) + 開發者
> **格式建議**:本文件適合作為總覽。實際大量 cases 建議用 TestRail / Xray / Zephyr 等工具管理。

---

## 撰寫原則

1. **一個 case 測一件事**,失敗時知道是哪裡壞
2. **可重現**:有人照著走就能跑
3. **獨立**:不依賴其他 case 的執行結果
4. **明確驗證點**:預期結果不能是「應該正常」這種模糊話

---

## 命名與編號

- 格式:`TC-[模組]-[流水號]`,例:`TC-ORD-001`
- 標題:用「應該…當…」格式
  - ✓ "TC-ORD-001:應該成功建立訂單當所有輸入合法"
  - ✗ "TC-ORD-001:測試訂單"

---

## 測試案例模板

### TC-ORD-001:應該成功建立訂單(Happy Path)

| 欄位 | 內容 |
|------|------|
| **模組** | Order |
| **類型** | 整合測試 |
| **優先序** | P0 |
| **前置條件** | 1. 使用者已登入<br>2. 商品 ID `prod-001` 存在且庫存 ≥ 5 |
| **測試資料** | 商品:prod-001、數量:2 |

**步驟**
1. 發送 POST /orders,body 含 product_id=prod-001, quantity=2
2. 等待回應

**預期結果**
- HTTP 201
- Response body 含 `id`、`status="draft"`、`total_amount` 等於商品單價 × 2
- DB orders 表新增一筆對應紀錄
- 庫存被預扣 2(從 5 變 3)
- 發出 order.created 事件

**清理**
- 取消測試訂單,還原庫存

---

### TC-ORD-002:應該回 409 當庫存不足

| 欄位 | 內容 |
|------|------|
| **模組** | Order |
| **類型** | 整合測試 |
| **優先序** | P0 |
| **前置條件** | 商品 prod-002 庫存 = 1 |

**步驟**
1. POST /orders,product_id=prod-002, quantity=5

**預期結果**
- HTTP 409
- error.code = "ORDER_INSUFFICIENT_STOCK"
- error.details 含 requested=5, available=1
- DB 無新訂單
- 庫存維持 1

---

### TC-ORD-003:應該以 idempotency key 防止重複建立

| 欄位 | 內容 |
|------|------|
| **模組** | Order |
| **類型** | 整合測試 |
| **優先序** | P0 |

**步驟**
1. POST /orders,header `X-Idempotency-Key: abc123`,記錄回傳的 order_id
2. 用相同 key 與 body 再 POST 一次

**預期結果**
- 兩次回應的 order_id 相同
- DB 僅有一筆訂單
- 第二次回應 HTTP 200(不是 201)

---

## 邊界條件 cases 範例

### TC-ORD-004:應該拒絕 quantity = 0
- 預期:HTTP 400,error.code = INVALID_INPUT

### TC-ORD-005:應該拒絕 quantity = 負數
- 預期:HTTP 400

### TC-ORD-006:應該拒絕 quantity > 99
- 預期:HTTP 400(對應業務規則)

### TC-ORD-007:應該處理超長商品列表(100 項)
- 預期:HTTP 201 或 HTTP 400(視業務上限)

### TC-ORD-008:應該處理空 items 陣列
- 預期:HTTP 400

### TC-ORD-009:應該拒絕未登入請求
- 預期:HTTP 401

### TC-ORD-010:應該拒絕無權限的使用者
- 預期:HTTP 403

---

## E2E User Journey 範例

### TC-E2E-001:完整下單到出貨流程

**情境**:新使用者註冊到完成第一筆訂單

**步驟**
1. 註冊新帳號
2. 登入
3. 瀏覽商品列表
4. 加入購物車(2 個商品)
5. 進入結帳
6. 填寫收件資訊
7. 選擇付款方式
8. 完成付款(測試金流)
9. 確認收到訂單確認 email
10. 後台模擬出貨
11. 確認收到出貨通知

**預期**
- 每一步順利進入下一步
- 訂單狀態正確流轉:draft → submitted → paid → shipped
- 通知正確發送

**測試時長**:約 5 分鐘(自動化)

---

## 效能測試 cases 範例

### TC-PERF-001:POST /orders 在 500 req/s 下回應時間
- **負載**:500 req/s 持續 5 分鐘
- **預期**:
  - P50 < 200ms
  - P99 < 1000ms
  - 錯誤率 < 0.1%

### TC-PERF-002:資料庫連線池在尖峰下不耗盡
- **負載**:1000 並發
- **預期**:
  - 無 connection timeout
  - 連線使用率 < 80%

---

## 安全測試 cases 範例

### TC-SEC-001:應該拒絕 SQL injection
**步驟**:在 customer_id 帶入 `' OR 1=1 --`
**預期**:HTTP 400,DB 無異常

### TC-SEC-002:應該防止 IDOR
**步驟**:使用者 A 嘗試讀使用者 B 的訂單
**預期**:HTTP 403 或 404(不洩漏存在性)

---

## 追蹤表(Coverage Matrix)

> 對應到需求 / 設計 / 票號,確認都有測試覆蓋。

| 需求 | 票號 | Test Cases | 狀態 |
|------|------|-----------|------|
| 建立訂單 | ORD-004 | TC-ORD-001~010 | ✅ |
| 取消訂單 | ORD-006 | TC-ORD-011~015 | 🚧 撰寫中 |
| 庫存扣減 | INV-002 | TC-INV-001~008 | ✅ |
| 安全 | - | TC-SEC-001~020 | 🚧 |
