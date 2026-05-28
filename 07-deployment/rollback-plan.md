# 回滾計畫(Rollback Plan)

> **目的**:出事時能在最短時間恢復服務,而不是當場想辦法。
> **負責人**:該服務 owner + SRE
> **何時寫**:**部署前就要寫好**,不是出事才想。
> **核心原則**:**假設它一定會出事**,把回滾當成標準流程而不是特例。

---

# [服務 / Release] Rollback Plan

| 欄位 | 內容 |
|------|------|
| 對應 Release | v1.2.0 |
| 撰寫者 | |
| 撰寫日期 | YYYY-MM-DD |
| 預計回滾時間 | < 5 分鐘 |
| 風險等級 | 低 / 中 / 高 |

---

## 1. 觸發回滾的條件(Decision Criteria)

> **預先定義**清楚什麼狀況要回滾,避免出事時人在猶豫。

### 立即回滾(P0,< 5 分鐘決策)
- 服務完全不可用(健康檢查失敗)
- 錯誤率 > 5%
- 資料損毀風險(寫入錯誤資料)
- 安全漏洞被觸發

### 評估後回滾(P1,15 分鐘內決策)
- 錯誤率 > 1% 持續 10 分鐘
- 關鍵 endpoint P99 latency 翻倍
- 重要客戶反映無法使用

### 不回滾,改用 hotfix 的情況
- 影響範圍小且有 workaround
- 回滾風險 > 繼續風險(例:資料 schema 已變)

---

## 2. 決策者

| 等級 | 誰可決定 |
|------|---------|
| P0 | On-call 工程師(事後通知) |
| P1 | 服務 owner + on-call |
| 評估爭議 | 技術 lead / Engineering Manager |

---

## 3. 回滾步驟

### 3.1 應用程式回滾(主要)

```bash
# 1. 確認當前版本
kubectl rollout history deployment/order-service -n order-prod

# 2. 回到前一版本
kubectl rollout undo deployment/order-service -n order-prod

# 或回到特定版本
kubectl rollout undo deployment/order-service -n order-prod --to-revision=42

# 3. 觀察 rollout 狀態
kubectl rollout status deployment/order-service -n order-prod

# 4. 驗證
curl https://api.example.com/v1/health
```

**預計時間**:3–5 分鐘

### 3.2 資料庫回滾(若需要)

> **這是最危險的部分。如果可以,設計時就要避免需要 DB rollback。**

#### 本次 release 的 DB 變更
| Migration | 是否可回滾 | 怎麼回滾 |
|-----------|-----------|---------|
| 加 `version` 欄位 | ✅ 是 | 程式碼回滾後欄位仍在,不影響 |
| 新增 `idempotency_keys` 表 | ✅ 是 | 程式碼回滾後表仍在,不影響 |
| 改 `orders.status` 為 ENUM | ⚠️ 困難 | 需手動轉回 VARCHAR |

#### 設計原則(Expand-Contract)
- **永遠先擴充再收縮**:加欄位 → 部署 → 用 → 移除舊欄位,要跨多個 release
- **避免破壞性變更**:rename、drop column 都是高風險
- 大表 schema 變更必須有 rollback 演練

### 3.3 配置回滾

若改了環境變數或 ConfigMap:
```bash
# Git 回滾
git revert <commit>
git push

# 或直接編輯 K8s
kubectl edit configmap order-config -n order-prod
```

### 3.4 第三方依賴

| 服務 | 是否需要協調 | 聯絡窗口 |
|------|------------|---------|
| 金流商 X | 否 | - |
| SMS 服務 | 否 | - |
| CDN | 若改了 cache 規則 | #platform |

---

## 4. 回滾後驗證

跑跟部署後一樣的 smoke test:
- [ ] Health endpoint 200
- [ ] 主要 endpoint 正常
- [ ] 監控儀表板綠燈
- [ ] 錯誤率回到正常
- [ ] 沒有殘留的 in-flight 請求卡住

---

## 5. 回滾後的善後

### 5.1 立即(< 1 小時)
- [ ] 通知 stakeholder:#release, #ops Slack
- [ ] 對外公告(若有影響使用者):status page 更新
- [ ] 開 incident ticket

### 5.2 短期(< 24 小時)
- [ ] 寫 postmortem(對應 incident-report.md)
- [ ] 分析 root cause
- [ ] 決定 hotfix 還是等下個 release 修

### 5.3 中期
- [ ] Retro 中討論
- [ ] 改善流程,避免重演
- [ ] 補上對應的測試

---

## 6. 已知不能回滾 / 困難回滾的部分

> 誠實列出。事先知道才能在出事時做對決策。

- **訂單資料一旦寫入,不會自動回滾**(這次 release 不影響舊資料)
- **發出的 email / SMS 無法收回**
- **第三方金流交易需手動退款**

---

## 7. 演練紀錄

| 日期 | 演練人 | 結果 | 學到什麼 |
|------|--------|------|---------|
| YYYY-MM-DD | | 4 分鐘完成 | rollout undo 比預期快;但 DNS cache 讓 client 看到舊資料約 1 分鐘 |

> **建議**:重大 release 前先在 Staging 跑一次回滾演練。

---

## 8. 相關文件
- Deployment Runbook:[連結]
- Release Notes:[連結]
- Monitoring Dashboard:[連結]
- Incident Response:[連結]
