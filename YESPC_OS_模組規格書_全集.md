# YESPC OS 模組規格書全集

本文件根據 YESPC OS 目前的實際 `index.html` 單檔系統整理而成，目的是把現有系統從「可用的內部工具」提升為「可維護、可交接、可重構、可商業化」的標準化資產。[file:23]  
目前系統已包含 Portal、Board、Sales、Repair、CRM、Stock、Admin 等模組，資料主體來自 Firestore 的 `sales`、`repairs`、`stocks`、`customers`、`dailySettlements`、`settings` 等集合，且模組之間存在明顯交叉連動。[file:23]

## 使用方式

這份文件不是寫給純工程師看的，而是寫給「未來的你、AI 工具、或接手工程師」看的。[file:23]  
每一個模組規格書都會盡量回答 6 件事：這個模組負責什麼、讀哪些資料、寫哪些資料、有哪些核心流程、哪些地方高風險、未來 UI / 架構升級時怎麼處理。[file:23]

## 模組地圖

| 模組 | 主要責任 | 主要集合 | 風險等級 |
|---|---|---|---|
| PortalModule | 系統入口與模組切換 | 無直接寫入 [file:23] | 低 [file:23] |
| BoardModule | 待辦、提醒、私密任務 | `todos` [file:23] | 中 [file:23] |
| SalesModule | 銷售、支出、日結、月報、歷史帳務 | `sales`、`dailySettlements`、`customers`、`stocks`、`repairs` [file:23] | 高 [file:23] |
| RepairModule | 維修 / 收購單、欄位自訂、CRM 提示 | `repairs`、`customers` [file:23] | 高 [file:23] |
| CrmModule | 客戶主檔與交易歷史整合 | `customers`、`sales`、`repairs` [file:23] | 中高 [file:23] |
| StockModule | 庫存主檔、數量維護、品項編輯 | `stocks` [file:23] | 高 [file:23] |
| AdminModule | 設定中心、清單維護、自訂欄位 | `settings` [file:23] | 中高 [file:23] |

## 帳務模組規格書

### 模組定位

帳務模組目前對應的是 `SalesModule`，但實際上它不只是「銷售頁」，而是整個 YESPC OS 的交易與日結核心。[file:23]  
它同時處理收入、支出、支付方式、客戶同步、庫存扣減、自動建立收購單、日結封存、月報統計與歷史日結檢視，因此是整個系統目前最常改、也最不適合直接亂改的模組。[file:23]

### 模組責任

| 責任區塊 | 說明 |
|---|---|
| 交易輸入 | 新增收入或支出交易，欄位包含日期、品項、售價、成本、客戶、電話、通路、備註等 [file:23] |
| 帳務統計 | 顯示當日營收、現金、線上收入、支出、月營收、月毛利、毛利率等 [file:23] |
| 日結封存 | 將當天未結算交易標記為 `isSettled=true`，並寫入 `dailySettlements` [file:23] |
| 歷史帳務 | 顯示已結算資料與歷史細項，可重新載入與修補 [file:23] |
| 客戶同步 | 銷售時依手機或姓名建立 / 更新 `customers` 主檔 [file:23] |
| 庫存連動 | 啟用 `autoStock` 時會自動扣除既有庫存或建立 `AUTO-GEN` 庫存資料 [file:23] |
| 維修連動 | 啟用同步時會自動新增一筆 `mode: purchase` 的 `repairs` 資料 [file:23] |

### 讀取資料

帳務模組目前依賴以下 props 與集合：

- `sales`：主要交易來源，也是當日、月度、歷史統計的基礎。[file:23]
- `settlements`：日結後的摘要資料，作為歷史財務視圖來源。[file:23]
- `customers`：CRM 自動提示與同步寫入來源。[file:23]
- `repairs`：CRM 提示整合，協助帶出客戶歷史與消費資訊。[file:23]
- `stocks`：商品名稱建議、自動帶入成本與售價、自動扣庫存的來源。[file:23]
- `crmCategories`：通路下拉選單來源。[file:23]

### 寫入資料

| 寫入集合 | 寫入時機 | 說明 |
|---|---|---|
| `sales` | `addSale`、`saveSaleEdit`、刪除銷售、日結回寫 [file:23] | 核心交易資料 |
| `dailySettlements` | `finalizeDaily`、`rebuildMissingSettlements`、手動修補 [file:23] | 每日結算摘要 |
| `customers` | 新增收入時依電話 / 姓名 upsert [file:23] | 客戶主檔與最近通路 |
| `stocks` | 開啟自動庫存時扣減數量或建立 `AUTO-GEN` 項目 [file:23] | 庫存自動連動 |
| `repairs` | 勾選同步時新增 `mode: purchase` 記錄 [file:23] | 與收購 / 維修流程接軌 |
| `deletedbackups` | `safeDelete` 執行時 [file:23] | 軟刪除備份 |

### 核心流程

#### 新增交易

`addSale` 會從 DOM 直接讀取 `SI`、`SP`、`SC`、`SCu`、`SPh`、`SSpec`、`SDa` 等欄位值，組成交易 payload 後寫入 `sales`。[file:23]  
若為收入交易且開啟 `autoStock`，會先嘗試扣減已選擇的庫存品項，找不到時則建立一筆 `AUTO-GEN` 的庫存項目，這是帳務與庫存目前最重要的耦合點之一。[file:23]

#### 編輯交易

`saveSaleEdit` 會更新 `sales` 單筆資料的 `item`、`price`、`cost`、`profit`、`paymentMethod`、`customer`、`phone`、`spec`、`createdAt` 等欄位。[file:23]  
目前編輯交易時並未同步回補或重算庫存，也沒有自動更新日結彙總，因此若編輯的是已結算資料，會有帳實落差風險。[file:23]

#### 刪除交易

刪除銷售不是直接 `delete`，而是先經過 `safeDelete`，把原始資料寫入 `deletedbackups`，再刪除 `sales` 文件。[file:23]  
這個機制很好，建議未來擴大成所有核心模組的標準刪除流程。[file:23]

#### 日結

`finalizeDaily` 會將當天所有未結算的 `sales` 設為 `isSettled=true`，並新增一筆 `dailySettlements` 資料，欄位包含 `date`、`revenue`、`expense`、`cashTurnedIn`、`onlineRevenue`。[file:23]  
這代表日結模組不是單純顯示結果，而是具有「改變整批交易狀態」的能力，因此屬於高風險操作。[file:23]

#### 補建日結

`rebuildMissingSettlements` 會重新掃描 `sales where isSettled == true`，依日期彙總出缺漏的日結資料，再批次寫入 `dailySettlements`。[file:23]  
這是一個救援工具，也說明目前帳務資料模型並非完全不可變，因此未來最好明確定義「何者為原始帳、何者為衍生摘要」。[file:23]

### 畫面區塊建議分層

帳務模組目前最適合拆成以下 5 層：

| 層級 | 現況內容 | 未來重構方向 |
|---|---|---|
| 輸入層 | 新增交易表單、CRM 提示、庫存建議 [file:23] | 獨立成 `SalesForm` |
| 交易層 | `addSale`、`saveSaleEdit`、刪除交易 [file:23] | 獨立成 `salesService` |
| 統計層 | 當日與月度統計計算 [file:23] | 獨立成 `salesSummaryUtils` |
| 結算層 | `finalizeDaily`、`loadHistDetail`、`rebuildMissingSettlements` [file:23] | 獨立成 `settlementService` |
| 備援層 | `safeDelete` 與歷史修補 [file:23] | 系統級共用工具 |

### 高風險點

- `addSale` 同時可能改動 `sales`、`customers`、`stocks`、`repairs`，屬於單點多寫入的高風險函式。[file:23]
- `saveSaleEdit` 會直接重寫 `createdAt` 與財務欄位，但未同步修正日結與庫存，容易造成歷史資料失真。[file:23]
- `finalizeDaily` 屬於批次狀態轉換操作，若重複執行或中途失敗，可能造成帳務不一致。[file:23]
- `cashInHand` 目前含固定 `5000` 基底，這是業務規則，但沒有被抽成設定值，未來難以多店適配。[file:23]

### 規格化建議

1. 將 `sales` 定義為「原始交易帳」，禁止 UI 隨意覆蓋歷史交易日期。[file:23]
2. 將 `dailySettlements` 定義為「衍生摘要」，可重建，但必須有重建紀錄。[file:23]
3. 將 `cashInHand` 的初始底金改為 `settings` 可設定值，而非硬編碼 5000。[file:23]
4. 為交易新增 `source`、`operator`、`linkedStockId`、`linkedRepairId` 欄位，方便追溯。[file:23]
5. 未來 UI 現代化時，優先將此模組做成 dashboard + form drawer 的分欄式結構，而不是全部垂直堆疊。[file:23]

### UI 現代化方向

帳務模組很適合未來改成「上方 KPI 摘要、左側交易清單、右側新增 / 編輯抽屜、底部日結區塊」的 web app 介面。[file:23]  
目前介面偏手機卡片式，操作快但資訊密度高；若未來要商業化，建議桌面版改為雙欄或三欄 dashboard，手機版保留精簡卡片流程。[file:23]

## Portal 模組規格書

### 模組定位

`PortalModule` 是系統首頁入口，主要負責把使用者導向 `board`、`sales`、`repair`、`crm`、`stock`、`admin` 等模組。[file:23]  
它本身沒有業務資料寫入，但它定義了整個系統的入口心智模型。[file:23]

### 模組責任

- 顯示功能入口按鈕。[file:23]
- 呼叫 `props.setTab()` 切換主要模組。[file:23]
- 提供營運人員最短操作路徑。[file:23]

### 高風險點

此模組風險低，但未來若功能持續增加，入口會失控，造成首頁變成「功能按鈕牆」。[file:23]

### UI 現代化方向

未來可改成 dashboard 式首頁，加入「今日營收、待處理維修、低庫存、近期客戶」摘要卡，而不只是圖示跳轉。[file:23]

## Board 模組規格書

### 模組定位

`BoardModule` 是內部待辦與提醒中心，管理 `todos` 集合，支援提醒日期、置頂、私密、完成狀態與人員分派。[file:23]

### 模組責任

- 新增與顯示待辦事項。[file:23]
- 依提醒日期判斷警示狀態。[file:23]
- 支援私密 / 公開 / 混合顯示。[file:23]
- 允許完成、取消完成、編輯與刪除。[file:23]

### 讀寫資料

- 讀取：`todos`。[file:23]
- 寫入：新增、更新 `completed` / `isPinned`、刪除 `todos`。[file:23]

### 高風險點

目前直接在 UI 中執行 `db.collection('todos').doc(id).update(...)` 與 `delete()`，資料層未抽離。[file:23]  
若未來導入權限管理，這個模組需要先定義誰能看私密、誰能刪除。[file:23]

### UI 現代化方向

可改成 Kanban 或 Today / Upcoming / Done 三欄結構，並保留手機版單欄清單。[file:23]

## Repair 模組規格書

### 模組定位

`RepairModule` 是維修與收購單核心模組，支援雙模式：`repair` 與 `purchase`。[file:23]  
它同時結合自訂欄位、CRM 自動帶入、收付款欄位、狀態流轉與詳細展開檢視，是另一個高複雜模組。[file:23]

### 模組責任

- 建立維修單或收購單。[file:23]
- 透過 `svcFields` / `svcBaseFields` 管理欄位顯示與順序。[file:23]
- 透過客戶搜尋將 `customers`、`sales`、`repairs` 歷史資料整合為 CRM 提示。[file:23]
- 顯示詳細內容與更新狀態。[file:23]

### 讀寫資料

- 讀取：`repairs`、`customers`、`sales`、`svcFields`、`crmCategories`。[file:23]
- 寫入：`repairs`、`customers`。[file:23]

### 高風險點

- 模式切換會影響欄位與狀態預設值，若未規格化，容易混淆「維修」與「收購」資料語意。[file:23]
- 自訂欄位 `customFields` 目前很靈活，但未建立欄位 schema 版本概念，未來資料相容性要注意。[file:23]

### UI 現代化方向

很適合改成主清單 + 右側詳情 panel 結構，並把表單做成步驟式，而非長表單一次到底。[file:23]

## CRM 模組規格書

### 模組定位

`CrmModule` 是客戶主檔與歷史交易視圖，會把 `customers` 主檔、`sales`、`repairs` 整合成單一客戶視角。[file:23]

### 模組責任

- 顯示客戶主檔與搜尋結果。[file:23]
- 呈現歷史交易與維修記錄。[file:23]
- 編輯客戶主檔，例如公司、統編、地址、Email、備註、通路。[file:23]

### 讀寫資料

- 讀取：`customers`、`sales`、`repairs`。[file:23]
- 寫入：`customers`。[file:23]

### 高風險點

客戶 ID 目前依電話或 `NAME_姓名` 規則產生，這在初期很實用，但未來容易出現重複客戶、改名合併困難等問題。[file:23]

### UI 現代化方向

可改為左側客戶列表、中央主檔、右側交易時間軸的三欄 CRM 版型。[file:23]

## 庫存模組規格書

### 模組定位

`StockModule` 目前管理 `stocks` 集合，負責品項資料、分類、供應商、成本、售價、數量、SN 與快速數量調整。[file:23]  
它已經是銷售模組的上游基礎，但現在仍偏向「主檔列表」，尚未完全進化成「可稽核的庫存流動系統」。[file:23]

### 模組責任

- 新增庫存品項。[file:23]
- 顯示分類、供應商、成本、售價、數量與 SN。[file:23]
- 透過 `+1 / -1` 快速調整數量。[file:23]
- 編輯既有品項資料。[file:23]
- 提供 `SalesModule` 商品建議來源。[file:23]

### 讀寫資料

- 讀取：`stocks`、`stockTagList`、`supplierList`。[file:23]
- 寫入：`stocks`。[file:23]

### 核心問題

目前庫存模組只維護結果值 `qty`，缺少「異動原因」與「流水帳」概念。[file:23]  
因此雖然你知道現在剩幾個，但很難回答「為什麼少了一個」「是誰調的」「是賣掉還是手動修掉」。[file:23]

### 高風險點

- 直接 `qty + 1 / qty - 1` 雖然方便，但沒有盤點記錄。[file:23]
- 刪除庫存目前是直接刪除，未經 `safeDelete`，與帳務模組標準不一致。[file:23]
- 銷售自動建立 `AUTO-GEN` 庫存時，可能造成資料品質不一致。[file:23]

### 規格化建議

1. 新增 `stockMovements` 集合，記錄每次進貨、銷售、盤點、報廢、手動調整。[file:23]
2. 為 `stocks` 加入 `minQty`、`status`、`isTrackedBySN` 欄位。[file:23]
3. 定義 SN 是否唯一、是否可空白、是否一品多件共用。[file:23]
4. 將刪除改為 `safeDelete` 標準流程。[file:23]
5. 銷售連動時，必須明確紀錄 `linkedSaleId` 或 `linkedStockId`。[file:23]

### UI 現代化方向

未來可改為「庫存總覽 KPI + 篩選工具列 + 表格式主清單 + 品項側欄詳情」的現代化後台介面。[file:23]

## Admin 模組規格書

### 模組定位

`AdminModule` 是設定中心，負責維護 `staffList`、`tagList`、`stockTagList`、`supplierList`、`crmCategoryList`、`serviceFieldConfig` 等清單與欄位設定。[file:23]

### 模組責任

- 維護系統下拉選單資料。[file:23]
- 編輯服務欄位顯示名稱。[file:23]
- 新增與刪除自訂欄位。[file:23]

### 高風險點

這個模組雖然 UI 簡單，但它控制了其他模組的選單與欄位結構，因此本質上屬於「系統 schema 管理器」。[file:23]  
尤其 `serviceFieldConfig` 會直接影響 Repair 模組資料結構，任何不當修改都有歷史相容性風險。[file:23]

### UI 現代化方向

未來應改成真正的設定頁分頁介面，例如「人員 / 客戶通路 / 庫存分類 / 維修欄位」分頁管理，並增加變更說明與版本提示。[file:23]

## 全系統重構優先順序

| 優先順序 | 模組 | 原因 |
|---|---|---|
| 1 | SalesModule | 最常改、最核心、跨集合最多、風險最高 [file:23] |
| 2 | StockModule | 與銷售高度耦合，且最需要建立規則 [file:23] |
| 3 | RepairModule | 自訂欄位多、商業流程重、未來可成為差異化賣點 [file:23] |
| 4 | CrmModule | 作為整合視角，應建立客戶主檔標準 [file:23] |
| 5 | AdminModule | 決定系統設定與欄位治理 [file:23] |
| 6 | BoardModule | 重要但較不影響商業化核心 [file:23] |
| 7 | PortalModule | 可最後一起做 UI 現代化 [file:23] |

## 下一步建議

第一步，先把本文件中的「帳務模組規格書」當作模板，未來每次改帳務需求前都先寫成需求句，再交給 AI 修改。[file:23]  
第二步，優先建立「資料欄位字典」，特別是 `sales`、`stocks`、`repairs`、`customers`、`dailySettlements` 的欄位定義，這會是你未來改成 Vite/React 多檔專案時最重要的橋樑。[file:23]  
第三步，如果你未來真的要做更現代化的 UI，建議先做視覺與資訊架構重畫，不要一邊重構邏輯一邊大改外觀，否則風險會疊加。[file:23]
