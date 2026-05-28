# 介面定義(Interfaces)

> **目的**:定義模組之間、系統與外部的介面契約(高層次,不到端點細節)。
> **負責人**:技術 lead / 架構師
> **Review**:資深工程師、介面兩端的 owner
> **注意**:這是「契約」,不是「實作」。一旦定下,變更要走變更流程。

---

## 1. 介面類型總覽

| 類型 | 用途 | 範例 |
|------|------|------|
| 對外 REST API | 給前端/第三方使用 | /api/orders |
| 內部 RPC | 服務間同步呼叫 | OrderService.createOrder |
| 事件(Events) | 服務間非同步通訊 | order.created |
| Webhook | 通知外部系統 | 訂單狀態變更通知客戶 |
| 第三方整合 | 我們呼叫別人 | 金流商 API、SMS 服務 |

## 2. 對外 API(High-level)

> 端點細節在 api-spec.md。這裡只列高層資源與用途。

| 資源 | 主要操作 | 對應模組 |
|------|---------|---------|
| /orders | CRUD + 狀態轉換 | Order Service |
| /payments | 建立、查詢、退款 | Payment Service |
| /products | 查詢 | Inventory Service |
| /users/me | 個人資料 | User Service |

## 3. 服務間介面

### 3.1 Order Service → Inventory Service
| 操作 | 同步/非同步 | 用途 |
|------|------------|------|
| checkAvailability | 同步 | 下單前檢查庫存 |
| reserveStock | 同步 | 預扣庫存(15 分鐘) |
| confirmStock | 同步 | 付款成功後正式扣減 |
| releaseStock | 同步 | 取消訂單時釋放 |

### 3.2 Order Service → Payment Service
(同上)

## 4. 事件契約(Events)

> 事件一旦發布就有人訂閱,變更比 API 更難。Schema 要嚴格管理。

### 4.1 order.created
| 項目 | 內容 |
|------|------|
| 觸發時機 | 訂單成功建立 |
| 發布者 | Order Service |
| 訂閱者 | Notification、Analytics、Inventory |
| Payload schema | order_id, customer_id, total, items[], created_at |
| 版本策略 | 向後相容;不相容變更需發新版本 event |

### 4.2 order.paid
(同上)

### 4.3 ...

## 5. 第三方整合

### 5.1 金流商 X
| 項目 | 內容 |
|------|------|
| 用途 | 信用卡付款處理 |
| 通訊方式 | REST + Webhook |
| 認證 | API Key + Signature |
| SLA | 99.9% |
| 失敗處理 | 重試 3 次 → 進入待人工處理佇列 |
| 文件連結 | [vendor docs] |

### 5.2 ...

## 6. 通訊規範

### 6.1 認證與授權
- 對外:OAuth 2.0 + JWT
- 內部:mTLS / service account
- 第三方:依各家規範

### 6.2 錯誤回應格式
> 所有 API 統一格式(細節在 error-handling.md)。

### 6.3 版本策略
- URL 版本(/v1/、/v2/)
- 同時支援舊版至少 6 個月
- Deprecation 提前 3 個月公告

## 7. 介面變更流程
1. 提出 RFC
2. 兩邊 owner 同意
3. 更新本文件
4. 必要時版本並行
