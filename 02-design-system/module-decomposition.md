# 模組切分(Module Decomposition)

> **目的**:定義系統由哪些模組組成、各自職責、邊界、依賴關係。
> **負責人**:技術 lead / 架構師
> **Review**:資深工程師

---

## 1. 切分原則

> 說明為什麼這樣切。例:
> - 依業務領域切(DDD bounded context)
> - 高內聚低耦合
> - 同一團隊負責的模組放一起
> - 變更頻率相近的放一起

## 2. 模組總覽

```
[ Web UI ]  [ Mobile App ]
       \      /
     [ API Gateway ]
            |
   +--------+--------+--------+
   |        |        |        |
 Order   Payment  Inventory  User
   |        |        |        |
   +--------+--------+--------+
            |
     [ Shared Database / Event Bus ]
```

## 3. 各模組詳述

### 3.1 Order Service(訂單服務)
| 項目 | 內容 |
|------|------|
| 職責 | 訂單建立、查詢、狀態管理 |
| 不負責 | 金流處理、庫存扣減(委派給其他服務) |
| 對外提供 | REST API、訂單狀態變更事件 |
| 依賴 | Payment Service、Inventory Service、User Service |
| Owner | 訂單組 |

**核心對外介面**(高層次,API 細節在 api-spec.md)
- 建立訂單
- 查詢訂單
- 取消訂單
- 訂閱訂單狀態變更

### 3.2 Payment Service(金流服務)
(同上結構)

### 3.3 ...

## 4. 依賴關係矩陣

| 模組 | 依賴 | 被依賴 |
|------|------|--------|
| Order | Payment, Inventory, User | API Gateway |
| Payment | (第三方金流) | Order |
| Inventory | - | Order |
| User | - | Order, Payment |

**循環依賴檢查**:☐ 無循環依賴

## 5. 模組邊界規則

> 哪些東西不能跨界?
- 不能直接讀對方的資料庫,只能透過 API 或事件
- 共用程式碼放在獨立的 shared library,不能讓 A 模組依賴 B 模組的內部
- ...

## 6. 部署單位

> 哪些模組會打包在一起部署?哪些獨立部署?
- Order + Payment 獨立部署(可獨立擴展)
- User 與 Auth 共同部署(共用 session)

## 7. 演進策略

> 未來可能怎麼變?
- 初期 monolith,模組以套件分隔
- 流量到 X 規模時,Order 可獨立成 service
- ...
