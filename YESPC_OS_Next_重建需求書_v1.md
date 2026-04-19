# YESPC OS Next 重建需求書 v1

本文件定義 YESPC OS Next 的重建範圍、技術方向、資料策略、模組優先順序與 AI 協作方式。  
目標不是把現有單檔系統硬改到看起來比較新，而是依照現有已驗證的營運流程，建立一個**更容易維護、更適合 AI 協作、更能成長成 SaaS 的新版本**。[file:23]

## 重建結論

YESPC OS 現在正處於最適合重建的時機，因為現有功能骨架已經成形，主要集合與模組也已經明確，但資料量仍少、歷史包袱仍低。[file:23]  
在這個時間點開一個平行的新專案，比繼續把所有新功能堆在單檔 `index.html` 上，更能降低未來維護成本，也更能發揮 Claude Code / Cursor 這類 AI agent 在標準工程專案中的優勢。[file:23][web:19]

## 專案定位

YESPC OS Next 的定位不是「新版介面」，而是「下一代產品骨架」。  
現有版本可繼續充當營運中的原型與需求驗證來源，而 YESPC OS Next 則負責承接未來 6 到 24 個月的功能成長、模組拆分、多人維護與商業化擴張需求。[file:23]

## 重建目標

YESPC OS Next 應同時達成以下五個目標：

- 保留現有營運邏輯，不破壞實際可用流程。[file:23]
- 改用標準化 React 專案架構，讓 AI 可以安全協作。[web:19]
- 把共用 UI、資料欄位、模組邊界明確化。[file:23]
- 讓新舊版可以並行一段時間，不影響營運。[file:23]
- 為未來多店版、角色權限、SaaS 化預留空間。[cite:1][file:23]

## 現況摘要

目前 YESPC OS 是以 React CDN、ReactDOM CDN、Firebase compat SDK 與單檔 `index.html` 組成的前端系統，使用 `React.createElement` 而不是 JSX，全部模組都放在同一份檔案裡。[file:23]  
核心模組已包含 `PortalModule`、`BoardModule`、`SalesModule`、`RepairModule`、`CrmModule`、`StockModule`、`AdminModule`，資料來源則集中於 `sales`、`stocks`、`repairs`、`customers`、`dailySettlements`、`settings`、`todos`、`deletedbackups` 等集合。[file:23]

## 為什麼不是直接改舊版

現有系統的優點是部署快、可以立即修改、你也已經熟悉它的結構。[file:23]  
但它的主要限制也很明顯：單檔過長、模組邊界模糊、命名有歷史混用、畫面與邏輯耦合度高，而且 AI 在這種非標準專案中進行大規模自動重構的風險明顯高於標準 React/Vite 專案。[file:23][web:19]

## 專案原則

### 1. 平行重建，不直接覆蓋

YESPC OS Next 必須是一個新專案，而不是直接在現有單檔上大改。  
這樣舊版可以繼續提供營運功能，新版則可逐步做模組驗證與遷移。[file:23]

### 2. 功能先對齊，再優化

第一階段的重建重點不是「比舊版更強」，而是「至少與舊版同等可用」。[file:23]  
像 Sales 的日結、Repair 的 CRM 自動帶入、Stock 的即時查找、safeDelete 這些營運核心，都必須先一比一對齊，之後再做體驗優化。[file:23]

### 3. 先做高價值模組

不是所有模組都要同時搬。  
應先搬最常用、最核心、最容易定義規格的模組，再逐步處理次要模組。[file:23]

### 4. 資料層要先穩

已有的欄位字典與 UI 改版藍圖應成為重建基礎文件，避免 AI 在重建過程中自行發明欄位與流程。[file:23]

## 技術棧建議

| 項目 | 建議方案 | 原因 |
|---|---|---|
| 前端框架 | React 18 + Vite [web:14] | 標準化、快、適合 AI 維護 |
| 語法 | JSX | 比 createElement 更適合拆模組與 AI 協作 [web:19] |
| UI 樣式 | Tailwind CSS [file:23] | 你現有系統已使用 Tailwind CDN，轉移成本低 [file:23] |
| 路由 | React Router 或單頁模組切換 | 視第一期需求而定 |
| 狀態管理 | 先用 React state/context，第二期再評估 Zustand | 先求穩定，不急著複雜化 |
| 後端資料 | Firebase Firestore [file:23] | 現有資料已在這裡 |
| 認證 | Firebase Auth [file:23] | 現有機制沿用 |
| 日期處理 | dayjs 或 date-fns | 取代零散手寫時間處理 |
| 圖示 | Lucide React | 取代 emoji 與自製符號 |

## 專案結構建議

```text
yespc-os-next/
├─ src/
│  ├─ app/
│  │  ├─ router/
│  │  ├─ providers/
│  │  └─ layouts/
│  ├─ modules/
│  │  ├─ dashboard/
│  │  ├─ sales/
│  │  ├─ repairs/
│  │  ├─ customers/
│  │  ├─ stocks/
│  │  ├─ board/
│  │  └─ admin/
│  ├─ components/
│  │  ├─ ui/
│  │  ├─ forms/
│  │  └─ cards/
│  ├─ services/
│  │  ├─ firebase/
│  │  ├─ repositories/
│  │  └─ mappers/
│  ├─ hooks/
│  ├─ utils/
│  └─ constants/
├─ public/
├─ docs/
└─ package.json
```

這種結構的目的，是把「畫面元件」、「模組業務邏輯」、「資料存取」、「舊資料轉換」分開，避免再次回到單檔耦合模式。[file:23]

## 第一階段重建範圍

第一階段建議只重建以下四個模組：

| 模組 | 是否第一期 | 原因 |
|---|---|---|
| SalesModule | 是 [file:23] | 最常用、牽涉日結與營收，是核心中的核心 |
| StockModule | 是 [file:23] | 與銷售高度關聯，重建價值高 |
| RepairModule | 是 [file:23] | 流程明確，且有 CRM 自動帶入價值 |
| CrmModule 基礎版 | 是 [file:23] | 作為銷售與維修的客戶中心 |
| Portal / Dashboard | 是 [file:23] | 作為新版首頁與捷徑總覽 |
| BoardModule | 否 [file:23] | 可晚搬，不是營運最核心 |
| AdminModule | 否 [file:23] | 可先保留舊版使用，避免一次做太多 |

## 第一階段功能對齊清單

### SalesModule 必須對齊

- 新增收入 / 支出交易。[file:23]
- 自動從庫存帶入品名、售價、成本。[file:23]
- CRM 搜尋提示與客戶自動帶入。[file:23]
- `autoStock` 行為與同步寫入邏輯。[file:23]
- 今日收入、現金庫存、月收入、月毛利、毛利率等 KPI 計算。[file:23]
- `safeDelete` 刪除保護。[file:23]
- 日結流程與歷史日結列表。[file:23]

### StockModule 必須對齊

- 新增庫存品項。[file:23]
- 以品名 / SN 搜尋。[file:23]
- 顯示成本、售價、數量。[file:23]
- 快速加減數量。[file:23]
- 可編輯既有品項。[file:23]

### RepairModule 必須對齊

- `repair` / `purchase` 模式切換。[file:23]
- 客戶姓名 / 電話搜尋並套用 CRM 提示。[file:23]
- 顯示與儲存自訂欄位 `customFields`。[file:23]
- 狀態更新。[file:23]
- 清單展開詳情與刪除。[file:23]

### CrmModule 必須對齊

- 客戶搜尋。[file:23]
- 顯示消費總額與最近資訊。[file:23]
- 顯示客戶歷史交易與維修紀錄。[file:23]
- 可更新基本資料。[file:23]

## 第二階段範圍

第二階段再納入以下內容：

- BoardModule 遷移。[file:23]
- AdminModule 設定中心遷移。[file:23]
- Repair 的欄位 schema 管理 UI 正式模組化。[file:23]
- 權限層級與角色分工。[cite:1]
- 報表輸出與更完整的分析頁。[cite:1]

## 資料策略

### 1. Firestore 先沿用

第一階段不更換資料庫，仍以現有 Firestore 為主，避免同時更動前後端造成風險。[file:23]

### 2. 建立 Mapper 層

因為現有資料欄位存在 `nm` / `customer`、`ph` / `phone`、`pr` / `price` 這類混用情況，YESPC OS Next 應在資料層建立 mapper，把舊資料轉成新專案統一格式。[file:23]  
例如：

- Firestore 原欄位：`nm`
- App 內部標準欄位：`customerName`

這樣可以讓 UI 與業務邏輯都只面對乾淨欄位，不用到處判斷舊欄位名稱。[file:23]

### 3. 衍生欄位不當唯一真相

像 `profit`、日結摘要、月 KPI 都應被視為衍生結果，而不是唯一真相；原始交易仍以 `sales` 集合為準。[file:23]

### 4. 所有資料操作統一走 repository

不要讓畫面元件直接寫 `db.collection(...).add(...)`。  
應建立 `salesRepository`、`stockRepository`、`repairRepository`、`customerRepository` 等層，未來 AI 修改時也比較容易控制影響範圍。[web:19]

## UI / UX 原則

YESPC OS Next 的畫面方向應沿用現代化 UI 改版藍圖，核心原則如下：

- 手機優先，但桌機版也要可用。[file:23]
- 每個模組資訊分層清楚，不再全部堆在單一長頁。[file:23]
- 新增與編輯動作盡量統一為 Bottom Sheet 或 Drawer。[file:23]
- KPI 卡、清單卡、分頁 Tab 都應共用元件。[file:23]
- 配色改為中性底色 + 模組主色點綴，不再整頁大面積彩色。[file:23]

## 重建流程建議

### 階段 0：建立專案骨架

- 初始化 `yespc-os-next` 專案。
- 安裝 React、Vite、Tailwind、Firebase SDK。
- 建立基本 layout、router、theme 與共用 UI 元件。
- 接上測試用 Firebase 專案或現有專案只讀模式。

### 階段 1：建資料層

- 建立 Firebase service。
- 建立 repository 層。
- 建立 mapper 層。
- 先打通 `sales`、`stocks`、`repairs`、`customers` 讀取流程。

### 階段 2：先完成首頁與 Sales

- 做新版 Dashboard。
- 完成 SalesModule 對齊版。
- 驗證 KPI 與日結流程。[file:23]

### 階段 3：完成 Stock + Repair + CRM

- 完成庫存模組。
- 完成維修模組。
- 完成 CRM 基礎版。
- 驗證模組之間的同步流程。[file:23]

### 階段 4：並行測試

- 舊版繼續上線使用。
- 新版在測試環境操作同樣流程。
- 比對新增交易、日結、客戶帶入、維修流程結果是否一致。[file:23]

### 階段 5：局部切換

- 先讓特定模組在新版正式使用，例如先切 Sales。
- 其他模組仍由舊版繼續提供。
- 確認穩定後再逐步切換。

## 新舊切換策略

| 策略 | 做法 | 建議 |
|---|---|---|
| 一次全切 | 全部模組做完再一次上線 | 不建議，風險太高 |
| 模組逐步切換 | 例如先切 Sales，再切 Stock | 最建議 |
| 新版只當測試站 | 長期不切正式流量 | 只適合短期驗證 |

最建議的做法是「**模組逐步切換**」，因為你的模組邏輯彼此相關，但又不是完全不可拆。[file:23]

## AI 協作方式

YESPC OS Next 的最大優勢之一，就是它應該從一開始就設計成 AI 友善專案。  
Claude Code、Cursor 等工具在標準化 React 專案中的優勢，主要來自可掃描多檔、理解 import/export 結構、跑命令、調整元件與樣式、維持一致命名風格。[web:19]

### AI 的責任範圍

應讓 AI 負責：

- 生出模組骨架與元件檔案。[web:19]
- 根據藍圖改 UI。[file:23]
- 寫 repository / mapper / hook。[web:19]
- 抽共用元件。[web:19]
- 協助測試與錯誤修正。[web:19]

不應直接完全交給 AI 的部分：

- 核心商業邏輯是否正確。[file:23]
- 日結與資料一致性驗證。[file:23]
- 欄位最終命名決策。[file:23]
- 正式上線前的切換判斷。

## Claude Code / Cursor 任務規格模板

未來對 AI 下指令時，建議固定使用以下格式：

```text
專案：YESPC OS Next
模組：SalesModule
目標：建立新版 Sales 頁面，功能與舊版對齊

資料來源：
- 舊版行為以 YESPC OS 現行 SalesModule 為準
- 欄位定義以資料欄位字典 v1 為準
- UI 結構以現代化 UI 改版藍圖 v1 為準

禁止事項：
- 不得自行新增 Firestore 欄位
- 不得刪除日結邏輯
- 不得更動資料計算公式

輸出要求：
- 建立 `modules/sales/` 內完整檔案
- 使用 JSX
- 使用 repository 層，不可在 component 直接呼叫 Firestore
- 請附上需要建立的檔案清單
```

## 風險與控制措施

| 風險 | 說明 | 控制方式 |
|---|---|---|
| AI 自行改動商業邏輯 | 例如改壞日結或同步流程 [file:23] | 每次任務都明確標註「不可更動邏輯」 |
| 新舊欄位不一致 | 舊資料命名混用 [file:23] | 建 mapper 層、先出欄位對照表 |
| 一次做太多 | 容易變成大爛尾 | 分階段，只先做核心四模組 |
| 使用者習慣斷裂 | 新版看起來太陌生 | UI 先改善結構，不改操作本質 |
| 舊版資料被誤寫 | 測試中直接碰正式資料 | 測試環境先只讀或用獨立專案 |

## 成功判斷標準

YESPC OS Next 第一階段完成時，至少應達成以下條件：

- 可以在新版完成新增收入 / 支出交易。[file:23]
- 可以從庫存自動帶入交易資訊。[file:23]
- 可以完成日結，且結果與舊版一致。[file:23]
- 可以建立與查詢維修 / 收購單。[file:23]
- 可以在 CRM 看到對應客戶與歷史。[file:23]
- 可以在新版操作一整天主要流程，不必回舊版補做核心動作。[file:23]

## 建議的第一個開發衝刺（Sprint 1）

第一個 Sprint 不要做太多，建議只做這些：

1. 建立 Vite 專案。
2. 接上 Firebase。
3. 建立 layout、底部 Tab、Dashboard 骨架。
4. 建立共用元件：`BottomSheet`、`KpiCard`、`ListCard`。
5. 建立 `salesRepository` 與 `salesMapper`。
6. 完成 SalesModule 的「只讀版」：能顯示 KPI 與今日交易列表，不先開寫入功能。

這樣的好處是：你可以先看到新專案跑起來，並用最關鍵的模組驗證技術方向，而不是一開始就衝進最複雜的表單寫入。[file:23][web:19]

## 文件關聯

YESPC OS Next 重建時，應把以下文件一起視為基礎規格：

- Claude Code 導入需求書。[cite:1]
- YESPC OS 資料欄位字典 v1。[file:23]
- YESPC OS 現代化 UI 改版藍圖 v1。[file:23]

這三份文件加上本需求書，就構成 YESPC OS Next 的初版產品規格包。
