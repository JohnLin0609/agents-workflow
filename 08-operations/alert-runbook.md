# 告警手冊(Alert Runbook)

> **目的**:on-call 收到告警時,知道下一步該做什麼。
> **負責人**:服務 owner + SRE
> **核心原則**:**每個告警必須對應一個 runbook**。沒 runbook 的告警不該存在(要嘛刪掉,要嘛補 runbook)。

---

## 1. 告警設計原則

### 1.1 好告警的標準
- ✅ 可行動(知道下一步)
- ✅ 有意義(真的需要人介入)
- ✅ 即時(發生時告,不是事後)
- ✅ 有 context(告警內含足夠資訊)

### 1.2 反模式(避免)
- ❌ Alert fatigue:太多假警報,真的出事沒人理
- ❌ 沒 runbook 的告警:醒了卻不知道幹嘛
- ❌ 純告知性 alert:該用 dashboard 不是 alert
- ❌ 對症不對因:每次都 restart,根本原因沒解

---

## 2. 告警分級

| 等級 | 定義 | 回應時間 | 通知方式 | 範例 |
|------|------|---------|---------|------|
| **P0** | 全面服務中斷、資料損毀 | 立即(< 5 min) | PagerDuty 電話 + SMS | 服務不可用、DB 主節點掛掉 |
| **P1** | 主要功能受影響、SLO 即將違反 | 15 min 內 | PagerDuty 推播 + Slack | 錯誤率 > 1%、P99 > 閾值 |
| **P2** | 次要問題、未影響使用者 | 工作時間內 | Slack 通知 | 單一 pod 重啟、磁碟 70% |
| **P3** | 預警 / 趨勢警告 | 隨時 | 信箱 / dashboard | 慢成長趨勢 |

---

## 3. On-call 流程

### 3.1 收到告警時(前 5 分鐘)
1. **確認收到**:在 PagerDuty / Slack 標記 acknowledged
2. **判斷真假**:看 dashboard 確認真的有問題,排除誤報
3. **判斷嚴重度**:是否需要升級
4. **開 incident channel**(P0/P1):#incident-YYYYMMDD-xxx

### 3.2 處理中
- 在 incident channel 同步進度,即使沒進展也說一下
- 不要單打獨鬥,P0/P1 主動找人
- 紀錄時間軸(自動或手動)

### 3.3 升級流程
- 15 分鐘無進展 → 找服務 owner
- 30 分鐘無進展 → 找技術 lead
- 涉及多服務 → 找 incident commander

---

## 4. 告警 Runbook(每個告警一份)

### Alert: HighErrorRate-OrderService

| 欄位 | 內容 |
|------|------|
| **觸發條件** | 5xx 比例 > 1% 持續 5 min |
| **嚴重度** | P1 |
| **負責服務** | order-service |
| **常見原因** | DB 慢、下游服務不可用、最近部署 |

#### 排查步驟

1. **看 dashboard**:[Order Service Dashboard 連結]
   - 是哪個 endpoint 在錯?
   - 是所有 pod 都錯,還是某幾個?
   - 同時看 DB、Redis、下游服務指標

2. **看 logs**:[Logs 連結 with 預設過濾]
   - 看 ERROR 等級訊息
   - 看是否有特定 stack trace 反覆出現

3. **看最近變更**:
   - 最近 24h 有沒有部署?→ 考慮 rollback
   - 設定有沒有改?

4. **看下游**:
   - Payment / Inventory 是否健康?
   - DB 連線池有沒有滿?

#### 緩解措施

按優先序嘗試:

1. **若是最近部署造成** → 立即 rollback(見 rollback-plan.md)
2. **若是流量暴衝** → 手動 scale up
   ```bash
   kubectl scale deployment/order-service -n order-prod --replicas=10
   ```
3. **若是下游問題** → 觸發 circuit breaker / 降級
4. **若是 DB 問題** → 聯絡 DBA

#### 確認解除
- 錯誤率回到 < 0.1%
- 持續觀察 15 分鐘無回升

#### 後續
- 開 ticket 記錄
- 若 P0/P1 → 排 postmortem

---

### Alert: HighLatency-PaymentEndpoint
(同上結構)

### Alert: DatabaseConnectionPoolExhausted
(同上)

### Alert: DiskSpaceLow
(同上)

---

## 5. 常見緊急操作 Cheat Sheet

### Scale up service
```bash
kubectl scale deployment/<service> -n <ns> --replicas=<n>
```

### Rollback deployment
```bash
kubectl rollout undo deployment/<service> -n <ns>
```

### Restart pods(慎用,先確認 root cause)
```bash
kubectl rollout restart deployment/<service> -n <ns>
```

### 看最近 logs
```bash
kubectl logs -l app=<service> -n <ns> --tail=200 -f
```

### 進 pod 內排查
```bash
kubectl exec -it -n <ns> <pod-name> -- /bin/sh
```

### Enable circuit breaker(降級)
```bash
# 透過 feature flag
curl -X POST $FLAG_URL/flags/payment-circuit/enable
```

---

## 6. 緊急聯絡

| 角色 | 聯絡 | 時段 |
|------|------|------|
| 服務 owner | @order-team | 工作時間 |
| 二級 on-call | PagerDuty escalation | 24/7 |
| DBA | #dba-emergency | 24/7 |
| Security | #security-oncall | 24/7 |
| Platform | #platform-oncall | 24/7 |
| CTO | (僅 P0) | 24/7 |

---

## 7. 告警審視與調整

### 7.1 每月 review
- 哪些告警太吵?→ 調閾值或刪除
- 哪些告警太晚?→ 收緊條件
- 哪些 incident 沒被告警抓到?→ 補上

### 7.2 指標
- False positive rate < 30%
- 每位 on-call 每週 P0/P1 不超過 X 次
- 告警響到 ack 中位數 < 5 分鐘

---

## 8. 相關文件
- Monitoring Dashboards:[連結]
- Incident Response Process:[連結]
- Service Runbooks:[連結]
- Postmortem 範本:[連結]
