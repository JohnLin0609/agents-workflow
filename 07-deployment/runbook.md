# 部署手冊(Deployment Runbook)

> **目的**:讓任何 on-call 工程師(包括沒寫過這塊的人)能照著步驟安全部署 / 回滾。
> **負責人**:該服務 owner
> **Review**:SRE / DevOps、技術 lead
> **核心原則**:**寫給半夜被叫起來的人看**——明確、可執行、不要假設讀者知道背景。

---

# [服務名稱] Deployment Runbook

| 欄位 | 內容 |
|------|------|
| 服務 | order-service |
| Owner | 訂單組 |
| 最後更新 | YYYY-MM-DD |
| 緊急聯絡 | #ops Slack channel / on-call: PagerDuty |

---

## 1. 服務簡介(背景)

> 給沒接觸過的人 30 秒理解這個服務。

- 用途:處理訂單建立、查詢、取消
- 技術棧:Node.js 20, NestJS, PostgreSQL, Redis
- 部署形式:Kubernetes Deployment, 3 replicas
- 上游:API Gateway
- 下游:Payment Service, Inventory Service, PostgreSQL, Kafka

---

## 2. 環境與存取

| 環境 | URL | K8s namespace | 誰可部署 |
|------|-----|--------------|---------|
| Dev | dev-api.example.com | order-dev | 任何開發者 |
| Staging | stg-api.example.com | order-stg | QA + Dev |
| Prod | api.example.com | order-prod | 限授權人員 |

### 2.1 存取需求
- `kubectl` context 已設定:`order-prod`
- AWS SSO 已登入
- 部署權限角色:`role/order-deployer`

### 2.2 取得權限
若沒權限:聯絡 #platform-team

---

## 3. 部署前檢查

- [ ] PR 已 merge 到 main
- [ ] CI 全綠(test、build、security scan)
- [ ] QA sign-off(對應 test report)
- [ ] Release Notes 已撰寫
- [ ] **Rollback 計畫已準備**(對應 rollback-plan.md)
- [ ] 確認沒有 freeze 期(週五下午、節日前禁部署)
- [ ] On-call 已知會

---

## 4. 部署流程

### 4.1 自動部署(預設)

正常流程透過 GitHub Actions:

1. PR merge → 自動部署 Dev
2. 在 GitHub Actions 介面點 "Deploy to Staging" → 部署 Staging
3. QA 驗證後,點 "Deploy to Production" → 需 approval → 部署 Prod

### 4.2 部署策略

採用 **Rolling Update**,逐 pod 替換:
- maxSurge: 1
- maxUnavailable: 0
- 預期時間:約 5 分鐘完成全部 pod

對於高風險變更,改用 **Canary**:
1. 部署 1 個 canary pod(5% 流量)
2. 觀察 15 分鐘
3. 確認正常後逐步擴大到 25% → 50% → 100%

### 4.3 手動部署(緊急情況)

若 CI/CD 不可用:

```bash
# 1. 切換 context
kubectl config use-context order-prod

# 2. 確認當前版本
kubectl get deployment order-service -n order-prod -o jsonpath='{.spec.template.spec.containers[0].image}'

# 3. 更新 image
kubectl set image deployment/order-service \
  order-service=registry.example.com/order-service:v1.2.0 \
  -n order-prod

# 4. 觀察 rollout
kubectl rollout status deployment/order-service -n order-prod

# 5. 驗證
curl https://api.example.com/v1/health
```

---

## 5. 部署後驗證

### 5.1 自動 Smoke Test
- CI 會自動跑 smoke test
- 失敗會自動觸發 rollback(若有設定)

### 5.2 手動檢查

執行以下,**全部通過才算部署成功**:

- [ ] Health endpoint 回 200:`curl https://api.example.com/v1/health`
- [ ] 主要端點可用:`curl ... GET /orders/test-id`
- [ ] 監控儀表板無紅燈:[Datadog 連結]
- [ ] 錯誤率未上升:[Grafana 連結]
- [ ] 確認沒有大量 5xx
- [ ] 確認 P99 latency 正常

### 5.3 部署後觀察期
- 前 30 分鐘:工程師持續監看
- 前 24 小時:on-call 留意告警

---

## 6. 部署窗口

### 6.1 建議時段
- 週二~週四 10:00 ~ 16:00
- 避開週五下午、週末、節日前

### 6.2 Freeze 期(禁部署)
- 雙 11、黑五、年底促銷
- 重大行銷活動前後 48h

### 6.3 緊急部署
- 需 release manager 或 CTO 簽核
- 必須有對應 incident ticket

---

## 7. 配置與環境變數

| 變數名 | 用途 | 預設 | 來源 |
|--------|------|------|------|
| DATABASE_URL | DB 連線 | - | AWS Secrets Manager |
| REDIS_URL | Redis 連線 | - | AWS Secrets Manager |
| INVENTORY_API_URL | Inventory 服務 | - | ConfigMap |
| LOG_LEVEL | 日誌等級 | info | ConfigMap |

**變更環境變數**:走 GitOps,改 `k8s/overlays/prod/configmap.yaml` 並提 PR。

---

## 8. 常見問題

### Q1:部署卡在 rollout
**排查**:
```bash
kubectl describe deployment order-service -n order-prod
kubectl logs -l app=order-service -n order-prod --tail=100
```
**常見原因**:
- ImagePull 失敗 → 確認 image 名稱與權限
- Readiness probe 失敗 → 服務沒順利啟動,看 logs

### Q2:部署完發現嚴重 bug
→ 立即執行 rollback(見 rollback-plan.md)。**不要硬修**。

### Q3:DB migration 跑很慢
→ 不要中斷。確認是否鎖大表。若超過 10 分鐘,聯絡 DBA。

---

## 9. 聯絡人

| 角色 | 對象 | 何時找 |
|------|------|--------|
| Service Owner | 訂單組 lead | 業務邏輯問題 |
| Platform | #platform-team | K8s / CI/CD 問題 |
| DBA | #dba | DB 異常 |
| On-call | PagerDuty | 任何 prod 緊急 |

---

## 10. 相關文件
- Rollback Plan: [連結]
- Release Notes: [連結]
- Architecture Doc: [連結]
- Monitoring Dashboard: [連結]
