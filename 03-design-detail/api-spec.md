# API 規格(API Spec)

> **目的**:定義每個 API 端點的契約。
> **負責人**:負責該模組的工程師
> **Review**:另一位工程師 + 前端/呼叫方
> **建議**:這份可以是 OpenAPI / Swagger YAML 的補充說明,正式契約用 OpenAPI 維護(機器可讀、可生成 SDK)。

---

## 1. 通用規範

### 1.1 Base URL
- Production:`https://api.example.com/v1`
- Staging:`https://api-staging.example.com/v1`

### 1.2 認證
- 方式:Bearer Token(JWT)
- Header:`Authorization: Bearer <token>`
- Token 取得:POST /auth/login

### 1.3 通用 Header

| Header | 用途 | 必填 |
|--------|------|------|
| Authorization | 認證 | 是(除公開端點) |
| Content-Type | application/json | 是(有 body 時) |
| X-Request-Id | 追蹤 ID(client 可帶) | 否 |
| Accept-Language | i18n | 否 |

### 1.4 通用回應格式

**成功**
```json
{
  "data": { ... },
  "meta": { "request_id": "..." }
}
```

**錯誤**(細節在 error-handling.md)
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Email format is invalid",
    "details": { "field": "email" }
  },
  "meta": { "request_id": "..." }
}
```

### 1.5 分頁

| 參數 | 預設 | 範圍 |
|------|------|------|
| page | 1 | ≥ 1 |
| page_size | 20 | 1–100 |

回應:
```json
{
  "data": [...],
  "meta": {
    "page": 1,
    "page_size": 20,
    "total": 153,
    "total_pages": 8
  }
}
```

### 1.6 Rate Limit
- 預設:60 req / min / user
- 超過回 429,Header 帶 `Retry-After`

---

## 2. 端點清單

### 2.1 Orders

#### POST /orders — 建立訂單

**用途**:客戶建立新訂單

**認證**:需要

**Request**
```json
{
  "items": [
    { "product_id": "uuid", "quantity": 2 }
  ],
  "shipping_address_id": "uuid",
  "coupon_code": "WELCOME10"
}
```

**驗證規則**
- items:至少 1 項
- quantity:1–99
- product_id:必須存在且未下架

**Response 201**
```json
{
  "data": {
    "id": "uuid",
    "status": "draft",
    "total_amount": 1200,
    "currency": "TWD",
    "payment_url": "https://..."
  }
}
```

**可能錯誤**
| HTTP | code | 情境 |
|------|------|------|
| 400 | INVALID_INPUT | 格式錯誤 |
| 404 | PRODUCT_NOT_FOUND | 商品不存在 |
| 409 | INSUFFICIENT_STOCK | 庫存不足 |
| 422 | INVALID_COUPON | 優惠券無效 |

**範例**
```bash
curl -X POST https://api.example.com/v1/orders \
  -H "Authorization: Bearer ..." \
  -H "Content-Type: application/json" \
  -d '{"items":[{"product_id":"...","quantity":1}]}'
```

#### GET /orders/{id} — 查詢訂單
(同上格式)

#### PATCH /orders/{id}/cancel — 取消訂單
(同上格式)

### 2.2 Payments
(同上)

---

## 3. Webhook(我們發出的)

### 3.1 order.status_changed

**觸發**:訂單狀態變更
**方法**:POST 到使用者註冊的 URL
**Header**:`X-Signature: sha256=...`
**Body**:
```json
{
  "event": "order.status_changed",
  "occurred_at": "2026-05-28T10:00:00Z",
  "data": { "order_id": "...", "old_status": "...", "new_status": "..." }
}
```
**重試**:失敗 exponential backoff,最多 5 次,共約 24h

---

## 4. 棄用與版本

| 端點 | 棄用日期 | 移除日期 | 替代 |
|------|---------|---------|------|
| GET /orders/legacy | 2026-01-01 | 2026-07-01 | GET /orders |

## 5. 變更紀錄
| 日期 | 版本 | 變更 |
|------|------|------|
| YYYY-MM-DD | v1.0 | 初版 |
