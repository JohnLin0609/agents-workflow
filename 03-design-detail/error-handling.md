# 錯誤處理(Error Handling)

> **目的**:定義錯誤分類、錯誤碼規範、log 策略、對使用者顯示的訊息。
> **負責人**:負責該模組的工程師
> **Review**:另一位工程師、前端、客服(訊息是否易懂)

---

## 1. 錯誤處理原則

1. **錯誤是程式的一部分,不是例外狀況**——預期到的失敗要顯式處理
2. **使用者訊息與內部訊息分離**——使用者看友善訊息,log 留完整技術細節
3. **不洩漏敏感資訊**——錯誤訊息不能含密碼、token、內部路徑、SQL
4. **錯誤可追蹤**——每個錯誤有 request_id 可對應到 log
5. **失敗要明確**——不要默默吞掉錯誤

## 2. 錯誤分類

| 類別 | 範例 | HTTP | 處理 |
|------|------|------|------|
| 客戶端錯誤 | 格式錯誤、缺欄位 | 400 | 修正後重試 |
| 認證失敗 | Token 過期 | 401 | 重新登入 |
| 權限不足 | 無權存取資源 | 403 | 不可重試 |
| 資源不存在 | 找不到該訂單 | 404 | 不可重試 |
| 衝突 | 庫存不足、重複建立 | 409 | 視情況 |
| 驗證失敗 | 業務規則不通過 | 422 | 修正後重試 |
| 限流 | 超過 rate limit | 429 | 等待後重試 |
| 伺服器錯誤 | 程式 bug、未預期 | 500 | log + alert |
| 服務不可用 | 第三方掛掉、維護中 | 503 | 退避後重試 |
| 超時 | 上游服務無回應 | 504 | 重試 |

## 3. 錯誤碼規範

### 3.1 格式
- 全大寫 + 底線
- 模組_類別_細節
- 例:`ORDER_INSUFFICIENT_STOCK`、`AUTH_TOKEN_EXPIRED`

### 3.2 錯誤碼清單

| Code | HTTP | 訊息(內部) | 使用者訊息 | 可重試 |
|------|------|------------|----------|--------|
| INVALID_INPUT | 400 | Invalid request payload | 輸入有誤,請檢查 | ✗ |
| AUTH_TOKEN_EXPIRED | 401 | JWT expired | 請重新登入 | ✗ |
| FORBIDDEN | 403 | Permission denied | 您沒有權限 | ✗ |
| ORDER_NOT_FOUND | 404 | Order {id} not found | 找不到訂單 | ✗ |
| ORDER_INSUFFICIENT_STOCK | 409 | Stock not enough | 商品庫存不足 | ✗ |
| RATE_LIMITED | 429 | Rate limit exceeded | 請求過於頻繁,請稍後 | ✓ |
| INTERNAL_ERROR | 500 | Unexpected error | 系統暫時無法處理 | ✓ |
| UPSTREAM_TIMEOUT | 504 | Payment service timeout | 處理中,請稍後查詢 | ✓ |

## 4. 統一錯誤回應格式

```json
{
  "error": {
    "code": "ORDER_INSUFFICIENT_STOCK",
    "message": "商品庫存不足",
    "details": {
      "product_id": "uuid",
      "requested": 5,
      "available": 2
    }
  },
  "meta": {
    "request_id": "req_abc123",
    "timestamp": "2026-05-28T10:00:00Z"
  }
}
```

- `code`:機器讀
- `message`:使用者看(已 i18n)
- `details`:額外結構化資訊(若有)
- `request_id`:追蹤用,客服可拿來查 log

## 5. Log 策略

### 5.1 等級
| Level | 何時用 |
|-------|--------|
| ERROR | 影響功能、需要關注 |
| WARN | 異常但不影響(例:重試成功) |
| INFO | 重要業務事件(下單、付款) |
| DEBUG | 除錯細節(只在 dev/staging) |

### 5.2 結構化 log

```json
{
  "level": "ERROR",
  "timestamp": "2026-05-28T10:00:00Z",
  "request_id": "req_abc123",
  "service": "order-service",
  "user_id": "uuid",
  "code": "ORDER_INSUFFICIENT_STOCK",
  "message": "...",
  "stack": "...",
  "context": { "order_id": "...", "product_id": "..." }
}
```

### 5.3 禁止 log 的內容
- 密碼、token、API key
- 信用卡號、CVV
- 完整身分證(若需,遮罩 A12****567)
- 大段使用者上傳的內容

## 6. 例外處理約定

### 6.1 程式碼層
- 不要寫 `catch (e) {}` 吞掉錯誤
- 不要把 exception 當 control flow
- 自訂 exception 類別對應錯誤碼

### 6.2 邊界處理
- API 層統一 exception handler,轉成標準錯誤格式
- 不要讓內部 stack trace 流到回應

## 7. 監控與告警

| 條件 | 動作 |
|------|------|
| 5xx 比例 > 1% / 5min | P1 告警 |
| 同一錯誤碼短時間激增 | P2 告警 |
| 第三方 timeout 連續 N 次 | P1 告警 |
