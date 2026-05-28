# Release Notes 模板

> **目的**:讓 stakeholder、使用者、運維、客服知道這次上線了什麼。
> **負責人**:PM(對外版) + 技術 lead(對內版)
> **版本**:對內 / 對外通常分兩份,因為受眾不同。

---

# Release Notes - v[版本號]

| 欄位 | 內容 |
|------|------|
| 版本 | v1.2.0 |
| 上線日期 | 2026-05-28 |
| 上線時段 | 10:00–10:15 UTC |
| 部署者 | |

---

## 📦 對外版(Customer-facing)

> 用使用者語言,不用技術術語。

### ✨ 新功能
- **訂單批次操作** — 後台現可一次處理多筆訂單,節省客服時間
- **訂單匯出 CSV** — 商家可下載月度訂單報表

### 🚀 改善
- 結帳速度提升約 30%
- 行動版界面優化

### 🐛 修正
- 修正某些優惠券無法正確套用的問題
- 修正訂單頁面在 Safari 偶爾顯示錯誤的問題

### ⚠️ 注意事項
- 5/30 之前建立的舊訂單,匯出時部分欄位為空(資料不存在)

---

## 🔧 對內版(Internal / Technical)

### 變更摘要

| 類型 | 數量 | 票號範圍 |
|------|------|---------|
| Feature | 3 | ORD-101 ~ 103 |
| Bug fix | 5 | BUG-040 ~ 044 |
| Refactor | 2 | TECH-012, TECH-015 |
| Tech debt | 1 | DEBT-007 |

### 詳細變更

#### Features
- **[ORD-101] 訂單批次操作 API**
  - 新增 POST /orders/batch
  - 支援批次取消、批次標記出貨
  - 文件:[api-spec.md 連結]

- **[ORD-102] CSV 匯出**
  - 新增 GET /orders/export
  - 大量資料用 streaming,避免 OOM

- **[ORD-103] Idempotency key 改存 DB**
  - 從 Redis 改為 DB,提升可靠性
  - Redis 仍作為一級快取

#### Bug Fixes
- **[BUG-040] 優惠券精度問題** — 用 decimal 取代 float
- **[BUG-041] Safari 訂單頁渲染** — CSS Grid 相容性修正
- **[BUG-042] 高並發超賣** — 改用樂觀鎖(version 欄位)
- **[BUG-043] 通知延遲** — 改用獨立 worker
- **[BUG-044] Timezone 顯示** — 統一改用使用者 timezone

#### Refactor / Tech Debt
- **[TECH-012] OrderService 拆分** — 從 800 行拆為 3 個 service
- **[TECH-015] 移除 deprecated API v0**
- **[DEBT-007] 升級 Node.js 18 → 20**

---

### ⚠️ Breaking Changes

> 任何破壞性變更必須列出,並提前公告。

- **API v0 已移除**(2026-01 已公告,半年遷移期)
- **訂單事件 schema** 新增 `version` 欄位(向後相容,不需 client 修改)

---

### 🗄️ Migration

> DB / 設定變更,需 ops 注意。

| 項目 | 動作 | 風險 | 是否破壞性 |
|------|------|------|----------|
| `2026_05_add_version_to_orders` | 加欄位 + 預設值 | 低 | 否 |
| `2026_05_create_idempotency_table` | 新表 | 低 | 否 |
| 環境變數新增 `IDEMPOTENCY_TTL_HOURS` | 改 ConfigMap | 低 | 否(有預設) |

---

### 📊 預期影響

| 指標 | 預期變化 |
|------|---------|
| Redis 流量 | ↓ 30%(idempotency 改 DB) |
| DB 寫入 | ↑ 5% |
| API latency P99 | 不變 |
| 訂單錯誤率 | ↓(BUG-042 修正) |

---

### 🔄 Rollback 計畫

見 rollback-plan.md。**簡述**:
1. K8s rollout undo(< 5 分鐘)
2. 新 DB 欄位向後相容,不需 rollback
3. Idempotency 表保留,但程式 fallback 回 Redis

---

### 🎯 上線後觀察重點(72 小時)

- [ ] 訂單錯誤率 < 0.1%
- [ ] BUG-042 是否還重現
- [ ] Idempotency DB 表 size 成長趨勢
- [ ] P99 latency 維持

---

### 👥 貢獻者

@alice, @bob, @carol

---

### 📚 相關連結

- 完整 commit 紀錄:[GitHub Compare 連結]
- Test Report:[連結]
- 對應的 issue / 需求文件:[連結]

---

## 變更紀錄

| 日期 | 版本 | 變更 |
|------|------|------|
| 2026-05-28 | v1.2.0 | 初版發布 |
