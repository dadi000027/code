# YESPC OS 資料欄位字典 v1

本文件根據 YESPC OS 目前單檔 `index.html` 的實際資料使用情況整理而成，目的是把系統目前散落在各模組內的資料結構，整理成可維護、可交接、可重構的統一欄位字典。[file:23]  
目前系統主要依賴 Firestore，核心集合包含 `sales`、`stocks`、`repairs`、`customers`、`dailySettlements`、`settings`，另外還有 `todos` 與 `deletedbackups` 等輔助集合。[file:23]

## 文件用途

這份欄位字典的用途不是只給工程師看，而是讓未來的你、AI 工具、接手工程師都能用同一套語言理解資料。[file:23]  
當你未來要做 UI 現代化、模組拆分、改成 React + Vite 多檔專案，或導入 Claude Code / Cursor 時，這份文件就是最重要的資料地基。[file:23][web:19]

## 命名原則

目前 YESPC OS 的資料欄位命名有中英混搭與歷史遺留情況，例如 `item`、`customer`、`phone` 與 `pr`、`dp`、`nm`、`ph` 並存，這代表系統已經可用，但還沒有完全標準化。[file:23]  
本版字典採取「**先忠實反映現況，再給標準化建議**」的方式處理，也就是先記錄現在系統真的在用什麼欄位，再補上未來建議欄位名稱與用途。[file:23]

## 集合總覽

| 集合名稱 | 主要用途 | 目前主要模組 |
|---|---|---|
| `sales` | 交易原始帳，包含收入與支出 [file:23] | SalesModule [file:23] |
| `stocks` | 庫存主檔與數量 [file:23] | StockModule、SalesModule [file:23] |
| `repairs` | 維修 / 收購單主檔 [file:23] | RepairModule、SalesModule [file:23] |
| `customers` | 客戶主檔與摘要資訊 [file:23] | CrmModule、SalesModule、RepairModule [file:23] |
| `dailySettlements` | 日結摘要資料 [file:23] | SalesModule [file:23] |
| `settings` | 系統設定、下拉選單、自訂欄位 [file:23] | AdminModule、RepairModule、SalesModule、StockModule [file:23] |
| `todos` | 待辦事項 [file:23] | BoardModule [file:23] |
| `deletedbackups` | 軟刪除備份 [file:23] | SalesModule，目前可擴大到全系統 [file:23] |

## sales 欄位字典

`sales` 是整個 YESPC OS 最重要的原始交易帳集合，所有日營收、月報、現金流、毛利、日結歷史等計算，都從這裡出發。[file:23]  
此集合同時包含收入與支出，因此 `type` 欄位是關鍵判斷欄位。[file:23]

| 欄位 | 型別 | 現況用途 | 來源模組 | 建議 |
|---|---|---|---|---|
| `item` | string | 品項或支出名稱 [file:23] | SalesModule [file:23] | 保留 |
| `type` | string | `income` / `expense`，區分收入與支出 [file:23] | SalesModule [file:23] | 保留，未來改 enum |
| `price` | number | 金額，收入為售價，支出為支出額 [file:23] | SalesModule [file:23] | 保留 |
| `cost` | number | 成本，主要用於收入毛利計算 [file:23] | SalesModule [file:23] | 保留 |
| `profit` | number | 毛利，通常來自 `price - cost` [file:23] | SalesModule [file:23] | 可保留，但應視為衍生欄位 |
| `paymentMethod` | string | `cash` / `online` 等付款方式 [file:23] | SalesModule [file:23] | 保留 |
| `customer` | string | 客戶名稱 [file:23] | SalesModule [file:23] | 保留 |
| `phone` | string | 客戶電話 [file:23] | SalesModule [file:23] | 保留 |
| `channel` | string | 客戶來源或通路 [file:23] | SalesModule [file:23] | 保留 |
| `spec` | string | 備註或規格說明 [file:23] | SalesModule [file:23] | 建議未來改名 `note` |
| `date` | string | 表單日期字串 [file:23] | SalesModule [file:23] | 可保留為顯示值 |
| `createdAt` | Timestamp | 真正排序與統計使用時間 [file:23] | SalesModule [file:23] | 必留，主時間欄位 |
| `isSettled` | boolean | 是否已日結 [file:23] | SalesModule [file:23] | 保留 |

### sales 建議新增欄位

| 欄位 | 型別 | 目的 |
|---|---|---|
| `operator` | string | 記錄是哪位店員建立或修改交易 |
| `sourceModule` | string | 標記來自手動銷售、收購同步、匯入等 |
| `linkedStockId` | string/null | 關聯庫存項目 |
| `linkedRepairId` | string/null | 關聯收購 / 維修單 |
| `settlementId` | string/null | 對應哪一筆日結摘要 |
| `storeId` | string | 未來多店版必要 |
| `updatedAt` | Timestamp | 保留修改時間 |

### sales 標準化建議

`sales` 應被定義成「不可輕易扭曲的原始交易帳」，因此像 `profit`、月毛利、日營收這類可重算的值，最好視為衍生結果而不是唯一真相。[file:23]  
未來如果要提高資料可靠度，建議限制已日結資料的直接編輯，改成透過更正單或重建流程處理。[file:23]

## stocks 欄位字典

`stocks` 目前是庫存主檔集合，支援品名查詢、SN 查詢、成本 / 售價帶入、數量顯示與快速增減。[file:23]  
它同時也是 `SalesModule` 的商品建議來源，因此欄位穩定性非常重要。[file:23]

| 欄位 | 型別 | 現況用途 | 來源模組 | 建議 |
|---|---|---|---|---|
| `name` | string | 品名 [file:23] | StockModule、SalesModule [file:23] | 保留 |
| `tag` / `category` | string | 分類或標籤 [file:23] | StockModule [file:23] | 建議統一成 `category` |
| `supplier` | string | 供應商 [file:23] | StockModule [file:23] | 保留 |
| `cost` | number | 進貨成本 [file:23] | StockModule、SalesModule [file:23] | 保留 |
| `price` | number | 建議售價 [file:23] | StockModule、SalesModule [file:23] | 保留 |
| `qty` | number | 目前庫存數量 [file:23] | StockModule、SalesModule [file:23] | 保留 |
| `sn` | string | 序號或識別碼 [file:23] | StockModule、SalesModule [file:23] | 保留，但需定規則 |
| `createdAt` | Timestamp | 建立時間 [file:23] | StockModule [file:23] | 建議保留 |

### stocks 建議新增欄位

| 欄位 | 型別 | 目的 |
|---|---|---|
| `minQty` | number | 最低庫存警戒值 |
| `status` | string | `active` / `archived` / `out_of_stock` |
| `isTrackedBySN` | boolean | 是否逐件追蹤 SN |
| `lastMovementAt` | Timestamp | 最後異動時間 |
| `storeId` | string | 多店版必要 |
| `note` | string | 品項補充說明 |

### stocks 標準化建議

`stocks` 現在只有結果值 `qty`，未來若要變成真正可稽核的庫存系統，應新增 `stockMovements` 集合，記錄每次進貨、銷售、盤點、報廢、手動調整。[file:23]  
這樣 `stocks.qty` 才會變成快取後的結果，而不是唯一依據。[file:23]

## repairs 欄位字典

`repairs` 同時承載維修單與收購單，透過 `mode` 來區分資料語意。[file:23]  
這個集合欄位最複雜，因為除了固定欄位，還有 `customFields` 會跟著設定中心變動。[file:23]

| 欄位 | 型別 | 現況用途 | 來源模組 | 建議 |
|---|---|---|---|---|
| `mode` | string | `repair` / `purchase` [file:23] | RepairModule、SalesModule [file:23] | 保留 |
| `repairId` / `rid` | string | 維修 / 收購單號 [file:23] | RepairModule [file:23] | 建議統一成 `repairId` |
| `date` | string | 表單日期 [file:23] | RepairModule [file:23] | 保留為顯示值 |
| `customer` / `nm` | string | 客戶姓名 [file:23] | RepairModule [file:23] | 建議統一成 `customerName` |
| `phone` / `ph` | string | 客戶電話 [file:23] | RepairModule [file:23] | 建議統一成 `customerPhone` |
| `company` | string | 公司名稱 [file:23] | RepairModule [file:23] | 保留 |
| `address` / `addr` | string | 地址 [file:23] | RepairModule [file:23] | 建議統一成 `address` |
| `email` / `em` | string | Email [file:23] | RepairModule [file:23] | 建議統一成 `email` |
| `model` / `mo` | string | 品項 / 型號 [file:23] | RepairModule [file:23] | 建議統一成 `model` |
| `spec` | string | 規格 [file:23] | RepairModule [file:23] | 保留 |
| `sn` | string | 序號 [file:23] | RepairModule [file:23] | 保留 |
| `description` / `ds` | string | 問題描述或說明 [file:23] | RepairModule [file:23] | 建議統一成 `description` |
| `price` / `pr` | number | 維修價或收購價 [file:23] | RepairModule [file:23] | 建議統一成 `price` |
| `deposit` / `dp` | number | 訂金 [file:23] | RepairModule [file:23] | 建議統一成 `deposit` |
| `paymentMethod` / `pay` | string | 付款方式 [file:23] | RepairModule [file:23] | 建議統一成 `paymentMethod` |
| `status` | string | 狀態流程欄位 [file:23] | RepairModule [file:23] | 保留 |
| `customFields` | object | 動態欄位值 [file:23] | RepairModule、AdminModule [file:23] | 保留 |
| `createdAt` | Timestamp | 建立時間 [file:23] | RepairModule [file:23] | 保留 |

### repairs 建議新增欄位

| 欄位 | 型別 | 目的 |
|---|---|---|
| `customerId` | string/null | 關聯 CRM 客戶主檔 |
| `linkedSaleId` | string/null | 若由銷售同步建立，記錄來源交易 |
| `operator` | string | 建立或更新的人員 |
| `fieldSchemaVersion` | number | 自訂欄位版本控制 |
| `updatedAt` | Timestamp | 最後更新時間 |
| `storeId` | string | 多店版必要 |

### repairs 標準化建議

這個集合目前最大問題不是功能不夠，而是欄位命名混用短代碼與語意名稱，例如 `nm`/`customer`、`ph`/`phone`、`pr`/`price` 並存。[file:23]  
未來重構時應建立正式欄位命名規則，並保留資料轉換層處理舊資料。[file:23]

## customers 欄位字典

`customers` 是 CRM 的主檔集合，但目前也常被 `SalesModule` 與 `RepairModule` 當作同步寫入目標。[file:23]  
這代表它不是純展示資料，而是整個客戶視角的主樞紐。[file:23]

| 欄位 | 型別 | 現況用途 | 來源模組 | 建議 |
|---|---|---|---|---|
| `id` | string | 文件 ID，常依電話或 `NAME_姓名` 生成 [file:23] | SalesModule、RepairModule、CrmModule [file:23] | 未來改獨立 ID |
| `name` | string | 客戶姓名 [file:23] | CrmModule、SalesModule [file:23] | 保留 |
| `phone` | string | 客戶電話 [file:23] | CrmModule、SalesModule [file:23] | 保留 |
| `taxId` | string | 統編 [file:23] | CrmModule、RepairModule、SalesModule [file:23] | 保留 |
| `company` | string | 公司名稱 [file:23] | CrmModule、RepairModule [file:23] | 保留 |
| `address` | string | 地址 [file:23] | CrmModule、RepairModule [file:23] | 保留 |
| `email` | string | Email [file:23] | CrmModule、RepairModule [file:23] | 保留 |
| `lastChannel` | string | 最近一次客戶通路 [file:23] | SalesModule、CrmModule [file:23] | 保留 |
| `note` | string | 備註 [file:23] | CrmModule [file:23] | 保留 |
| `createdAt` | Timestamp | 建立時間 | 建議統一補上 | 建議必留 |

### customers 建議新增欄位

| 欄位 | 型別 | 目的 |
|---|---|---|
| `displayName` | string | 顯示名稱，處理公司客與個人客 |
| `customerType` | string | `individual` / `business` |
| `tags` | string[] | 客戶分類標籤 |
| `updatedAt` | Timestamp | 最後更新時間 |
| `storeId` | string | 多店版必要 |
| `mergedInto` | string/null | 客戶合併時使用 |

### customers 標準化建議

目前客戶文件 ID 若直接用電話或姓名組合，短期很方便，但長期會遇到手機變更、重複姓名、企業客戶多聯絡人等問題。[file:23]  
未來應改成獨立 `customerId`，電話與姓名只作查找索引，而不是主鍵。[file:23]

## dailySettlements 欄位字典

`dailySettlements` 是從 `sales` 衍生出來的日結摘要集合，不是原始交易帳。[file:23]  
目前主要由 `finalizeDaily` 與 `rebuildMissingSettlements` 生成。[file:23]

| 欄位 | 型別 | 現況用途 | 來源模組 | 建議 |
|---|---|---|---|---|
| `date` | string | 日結日期 [file:23] | SalesModule [file:23] | 保留 |
| `revenue` | number | 當日收入總額 [file:23] | SalesModule [file:23] | 保留 |
| `expense` | number | 當日支出總額 [file:23] | SalesModule [file:23] | 保留 |
| `cashTurnedIn` | number | 當日繳回現金 [file:23] | SalesModule [file:23] | 保留 |
| `onlineRevenue` | number | 線上收入 [file:23] | SalesModule [file:23] | 保留 |
| `createdAt` | Timestamp | 建立時間 | 建議補齊 | 建議必留 |

### dailySettlements 建議新增欄位

| 欄位 | 型別 | 目的 |
|---|---|---|
| `settlementBy` | string | 執行日結的人員 |
| `sourceSalesCount` | number | 當日納入幾筆交易 |
| `sourceSalesIds` | string[] | 關聯交易 ID（可選） |
| `rebuildFlag` | boolean | 是否由補建程式生成 |
| `storeId` | string | 多店版必要 |

### dailySettlements 標準化建議

這個集合應明確被視為摘要表，而不是唯一真相。[file:23]  
未來若要增加可信度，應記錄它來自哪些 `sales`，以及是正常日結還是補建生成。[file:23]

## settings 欄位字典

`settings` 是系統設定中心資料來源，主要支撐各模組的下拉選單、自訂欄位與欄位顯示邏輯。[file:23]  
雖然從 UI 看起來像簡單設定，但本質上它其實是系統 schema 的治理中心。[file:23]

| 設定項目 | 型別 | 現況用途 |
|---|---|---|
| `staffList` | string[] | 待辦、模組操作的人員清單 [file:23] |
| `tagList` | string[] | 待辦標籤 [file:23] |
| `stockTagList` | string[] | 庫存分類清單 [file:23] |
| `supplierList` | string[] | 庫存供應商清單 [file:23] |
| `crmCategoryList` | string[] | 客戶通路 / 分類清單 [file:23] |
| `serviceFieldConfig` / `svcFields` | object[] | Repair 模組自訂欄位配置 [file:23] |

### serviceFieldConfig 建議標準欄位

| 欄位 | 型別 | 用途 |
|---|---|---|
| `id` | string | 欄位唯一 ID |
| `label` | string | 顯示名稱 |
| `type` | string | 欄位型別，例如 text / select / number |
| `visible` | boolean | 是否顯示 |
| `required` | boolean | 是否必填 |
| `order` | number | 顯示順序 |
| `options` | string[] | 下拉欄位選項 |

### settings 標準化建議

`settings` 未來最好拆成多份明確文件，例如 `appSettings`、`dropdownOptions`、`serviceFieldConfig`，不要把所有系統設定都塞在單一文件裡。[file:23]  
這樣未來改版時，Repair 的欄位管理才不會跟庫存分類、人員清單混在一起。[file:23]

## todos 欄位字典

`todos` 是 Board 模組的待辦集合，雖然不是商業化核心，但仍是系統日常運作的一部分。[file:23]

| 欄位 | 型別 | 現況用途 |
|---|---|---|
| `text` | string | 待辦內容 [file:23] |
| `tag` | string | 分類標籤 [file:23] |
| `assignee` | string | 指派人員 [file:23] |
| `reminderDate` | string | 提醒日期 [file:23] |
| `completed` | boolean | 完成狀態 [file:23] |
| `isPinned` | boolean | 是否置頂 [file:23] |
| `isPriv` | boolean | 是否私密 [file:23] |
| `createdAt` | Timestamp | 建立時間 [file:23] |

## deletedbackups 欄位字典

`deletedbackups` 是目前 `safeDelete` 使用的刪除備份集合，屬於很有價值的安全設計。[file:23]

| 欄位 | 型別 | 現況用途 |
|---|---|---|
| `originalCollection` | string | 原集合名稱 [file:23] |
| `originalId` | string | 原始文件 ID [file:23] |
| `data` | object | 被刪除的完整資料 [file:23] |
| `deletedAt` | Timestamp | 刪除時間 [file:23] |
| `deletedBy` | string | 執行刪除者 [file:23] |

### deletedbackups 標準化建議

未來應把 `safeDelete` 擴大為全系統標準刪除機制，至少 `sales`、`stocks`、`repairs`、`customers` 都應適用。[file:23]  
這對非技術背景使用者尤其重要，因為你未來會更依賴 AI 幫你修改資料，備援機制越完整越安全。[file:23][web:19]

## 全系統欄位命名標準建議

目前系統中最明顯的問題，是同一語意欄位可能同時存在短代碼與完整英文欄位，例如 `nm` / `customer`、`ph` / `phone`、`pr` / `price`。[file:23]  
未來如果要重構成現代化專案，建議建立以下原則：

- 顯示欄位與資料欄位分開，資料層一律用完整英文名稱。[file:23]
- 所有時間欄位統一為 `createdAt`、`updatedAt`、`deletedAt`。[file:23]
- 所有關聯欄位統一用 `xxxId` 命名。[file:23]
- 所有衍生欄位要明確標示，可重算的不要當唯一真相。[file:23]
- 所有集合預留 `storeId`，為未來 SaaS 多店版做準備。[file:23]

## 下一步建議

這份欄位字典完成後，最適合接著做的是「YESPC OS 現代化 UI 改版藍圖」，因為現在資料語言已經開始固定，可以安全地往畫面層規劃。[file:23]  
之後再做 `SalesModule` 與 `StockModule` 的改版需求書，會比現在直接談畫面漂亮很多，因為屆時邏輯與欄位都比較穩。[file:23]
