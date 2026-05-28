# 監控儀表板(Monitoring Dashboards)

> **目的**:讓任何人在 30 秒內知道「服務現在是不是健康」。
> **負責人**:服務 owner + SRE
> **核心原則**:**儀表板要對應決策**——看到紅燈知道下一步該做什麼,而不是只是好看。

---

## 1. 儀表板設計原則

1. **分層**:上層看健康度(綠/黃/紅),下層看細節
2. **聚焦**:一頁不超過 10 個 panel,看不完等於沒看
3. **可比較**:時間軸對齊,環比 / 同比一目了然
4. **可行動**:看到異常知道下一步——配上 alert runbook 連結
5. **避免儀表板癌症**:沒人看的 panel 定期清掉

---

## 2. 服務健康度概覽(Top-Level Dashboard)

### 2.1 黃金訊號(Golden Signals)

對應 Google SRE 書,**最重要的 4 個指標**:

| 訊號 | 量測 | 目標 | 告警閾值 |
|------|------|------|---------|
| **Latency** | P50 / P99 回應時間 | P99 < 1s | P99 > 2s |
| **Traffic** | RPS(requests per second) | - | 異常下降 50% |
| **Errors** | 5xx 比例 | < 0.1% | > 1% 持續 5min |
| **Saturation** | CPU / Memory / 連線池使用率 | < 70% | > 85% |

### 2.2 SLO 達成狀況

| SLO | 目標 | 當前(30 天) | 剩餘 error budget |
|-----|------|-------------|------------------|
| 可用性 | 99.9% | 99.95% | 60% |
| 訂單成功率 | 99.5% | 99.7% | 70% |
| P99 latency | < 1s | 850ms | - |

---

## 3. 業務指標儀表板(Business Metrics)

> 技術指標健康不代表業務健康。

### 3.1 核心 KPI

| 指標 | 即時 | 1h 前 | 24h 前 | 7d 平均 |
|------|------|-------|--------|---------|
| 訂單數 / 小時 | | | | |
| 訂單成功率 | | | | |
| 平均客單價 | | | | |
| 取消率 | | | | |

### 3.2 漏斗
```
瀏覽商品 → 加入購物車 → 結帳 → 付款 → 完成
   100%        45%       28%    24%    23%
```
**異常徵兆**:某一段轉換率突降。

---

## 4. 各服務細節儀表板

### 4.1 Order Service

| Panel | 用途 |
|-------|------|
| RPS by endpoint | 看流量分佈 |
| Error rate by endpoint | 找出有問題的 endpoint |
| Latency heatmap | 看分佈,不只是平均 |
| DB connection pool | 看是否快滿 |
| Redis hit rate | 看快取效率 |
| Pod count & status | 看 K8s 狀態 |

### 4.2 Payment Service
(同上結構)

### 4.3 Inventory Service
(同上)

---

## 5. 基礎設施儀表板

| 類別 | 重點指標 |
|------|---------|
| Kubernetes | Pod 數量、重啟次數、Pending pods、Node CPU/Mem |
| Database | 連線數、慢查詢、replication lag、磁碟使用 |
| Redis | 記憶體用量、命中率、evicted keys、連線數 |
| Kafka | Lag、partition 平衡、producer/consumer 健康 |
| Network | 對外頻寬、5xx 比例(從 LB) |

---

## 6. 使用者體驗監控(RUM)

從**真實使用者端**量,不只是後端視角。

| 指標 | 目標 |
|------|------|
| First Contentful Paint | < 1.5s |
| Largest Contentful Paint | < 2.5s |
| Cumulative Layout Shift | < 0.1 |
| JS 錯誤率 | < 0.5% |
| 行動 vs 桌機效能差距 | < 30% |

---

## 7. 日誌儀表板

> 不是看 raw logs,是看「日誌中能聚合出的訊號」。

- 各 service 的 ERROR 等級數量趨勢
- Top 10 錯誤訊息
- 各 endpoint 的 4xx / 5xx 分佈
- 慢查詢 top 10

---

## 8. 自訂儀表板(依場景)

| 場景 | 儀表板 |
|------|--------|
| 上線觀察 | release-watch:對比 release 前後關鍵指標 |
| 壓測 | load-test:RPS、latency、resource |
| 黑五 / 雙 11 | event-day:即時訂單、支付成功率、客服佇列 |
| Postmortem | incident-{id}:出事時間段的所有指標凍結 |

---

## 9. 儀表板維護

### 9.1 定期 review
- 每季 review 一次,刪除沒人看的 panel
- 新功能上線時,評估要不要加新 panel

### 9.2 命名規範
- `[service]-[purpose]`:例 `order-overview`、`order-debug`
- 一個 service 別超過 5 個 dashboard

### 9.3 權限
- 唯讀:所有工程師
- 編輯:服務 owner

---

## 10. 工具配置

| 工具 | 用途 | 連結 |
|------|------|------|
| Datadog | APM + 儀表板 | [連結] |
| Grafana | 自訂視覺化 | [連結] |
| Prometheus | metrics 儲存 | [連結] |
| Loki / ELK | 日誌 | [連結] |
| Jaeger | 分散式追蹤 | [連結] |
| Sentry | 錯誤追蹤 | [連結] |

### 配置即程式碼
- Dashboard 用 JSON / Terraform 管理,進版控
- 不在 UI 上手改 prod dashboard
