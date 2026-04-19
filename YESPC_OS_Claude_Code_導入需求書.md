# YESPC OS — Claude Code 導入需求書
**版本 v1.0 ｜ 2026 年 4 月 ｜ 楊店長 · YESPC**

---

## 一、文件目的

本文件供楊店長作為「YESPC OS 是否導入 Claude Code / Cursor」之決策依據，並同時作為系統重構的分階段行動藍圖。文件不假設你會立刻導入 Claude Code，而是讓你能清楚知道：**現在能做什麼、什麼時候買才值、買了之後怎麼用最省事。**

---

## 二、YESPC OS 現況快照

### 2.1 技術架構

| 項目 | 現況 |
|---|---|
| 框架 | React 18（CDN 引入，非 npm 安裝） |
| 語法 | React.createElement，非 JSX |
| 結構 | 單檔 index.html，所有邏輯集中 |
| 資料儲存 | Firebase Firestore（主）+ IndexedDB（本地） |
| 部署方式 | Firebase Hosting，無編譯流程 |
| 樣式 | Tailwind CSS（CDN） |
| 外部整合 | Telegram Bot 通知 |

### 2.2 現有模組清單

| 模組名稱 | 功能說明 | 對應資料集 |
|---|---|---|
| SalesModule | 銷售記錄、日結、POS 收款 | sales |
| CrmModule | 客戶管理、通路分類、歷史記錄 | customers |
| RepairModule | 維修單開立、進度追蹤、通知 | repairs |
| StockModule | 庫存進銷存、零件管理 | stocks |
| AdminModule | 自訂欄位、下拉選單維護 | settings |
| FinanceModule | 財務匯總、日報表 | 跨多集合彙整 |

### 2.3 已知特性與限制

- 所有模組 state 集中在最外層 `<App />`，子模組透過 props 傳入，**不使用 Redux / Zustand**
- `safeDelete`、`Error Boundary`、匿名登入、`dailySettlement` 等機制已存在
- CRM 與銷售有資料交叉連動（新增銷售會影響客戶歷史）
- 目前**無測試套件、無 CI/CD、無版本控制（Git）**
- 系統未來計劃 SaaS 商業化販售給同業

---

## 三、Claude Code 是什麼？它能做什麼？

### 3.1 功能定義

Claude Code 是 Anthropic 推出的 **終端機 AI Coding Agent**，它不是單純的「程式碼補全」，而是能夠：

- 掃描整個 codebase（多個資料夾、數百個檔案）
- 自動修改多個檔案（跨檔案協同改動）
- 執行終端機指令（npm install、git commit、跑測試）
- 根據錯誤訊息自動修正，反覆迭代直到通過
- 理解「幫我加這個功能」的高層次需求，自己規劃並執行

### 3.2 它最擅長的環境

Claude Code 設計上針對「**現代標準化工程專案**」最有效率：

```
理想環境：
├── src/
│   ├── components/
│   │   ├── Sales.jsx
│   │   ├── CRM.jsx
│   │   └── Stock.jsx
│   ├── hooks/
│   ├── utils/
│   └── App.jsx
├── package.json
└── vite.config.js
```

在這種環境下，它能安全地：
- 找到 `Sales.jsx`，只改銷售模組，不影響其他模組
- 透過 import/export 追蹤依賴關係
- 改完後跑 `npm run build` 驗證沒有編譯錯誤

### 3.3 它面對你目前架構的限制

你目前是**單檔 + CDN + createElement 架構**，Claude Code 若直接接手會遇到：

| 風險點 | 說明 |
|---|---|
| 語法誤判 | 它會傾向把 `createElement` 改成 JSX，CDN 環境跑不起來 |
| 邊界不清 | 單一大檔沒有模組邊界，AI 不知道改一處會影響哪裡 |
| 無法驗證 | 沒有 build 流程，AI 無法自動跑指令確認沒改壞 |
| 拆分失敗 | 叫它「拆出 Sales 模組」，很可能輸出 `import Sales from './Sales.js'`，但你的環境不支援 ES Module |
| 全域污染 | 現有大量函式可能共用全域變數，AI 拆檔時難以追蹤 |

**結論：現在訂閱，投報率不高；但這不等於永遠不能用。**

---

## 四、分階段導入計畫

### 第一階段：整理地基（現在可以做）

> **目標：讓現有單檔系統「可讀性」提高，為後續 AI 接手做準備**  
> **工具：繼續用現有 AI 助手（如我）協助，不需訂閱 Claude Code**  
> **預估時間：2–4 週（依功能穩定程度）**

#### 4.1.1 你需要做的事

**（一）先把系統功能凍結、穩定化**
- 確認所有現有模組的主要功能沒有 Bug
- 先不要加新功能，先讓現有功能穩定運行一段時間
- 記錄「哪些功能最常改」「哪些地方最容易壞」

**（二）建立模組邊界標記（在現有單檔內加註解）**

在 index.html 裡，幫每個模組加清楚的區塊標記，例如：

```javascript
// ============================================================
// MODULE: SalesModule
// 責任：銷售記錄、日結、POS
// 資料集：Firestore > sales
// 依賴：customers（查客戶）、stocks（扣庫存）
// 最後修改：2026-04-xx
// ============================================================
```

**（三）整理全域 state 清單**  
列出所有在 `<App />` 層級的 `useState`，並標記哪個 state 給哪個模組用。

**（四）建立 Git 版本控制**  
這是最重要的基礎建設。每次改動前 commit 一版，這樣不管是你改還是 AI 改，壞掉了都能回頭。

```bash
git init
git add index.html
git commit -m "YESPC OS v初始版本"
```

#### 4.1.2 第一階段完成標誌

- [ ] 所有模組都有清楚的區塊標記與責任說明
- [ ] 全域 state 清單已整理
- [ ] 已建立 Git Repository，每次改動前 commit
- [ ] 已記錄「哪些功能最常改」清單

---

### 第二階段：邏輯分層（中期，可以開始考慮 Claude Code）

> **目標：在不改變現有架構的前提下，讓代碼結構更模組化**  
> **建議工具：Claude Code（有限度使用）或繼續人工主導**  
> **預估時間：4–8 週**

#### 4.2.1 你需要做的事

**（一）抽出共用 helper 函式**  
把所有「不只一個模組會用到的邏輯」抽出集中，例如：

```javascript
// ============================================================
// UTILS: 共用工具函式
// ============================================================
function formatDate(ts) { ... }
function formatPrice(num) { ... }
function generateOrderId() { ... }
```

**（二）抽出 Firestore CRUD 操作**  
把每個模組的資料庫操作集中成函式群組：

```javascript
// ============================================================
// DATA LAYER: Sales 資料操作
// ============================================================
async function createSale(data) { ... }
async function getSales(dateRange) { ... }
async function deleteSale(id) { ... }
```

**（三）把 UI 元件邏輯從業務邏輯分開**  
UI（按鈕長什麼樣）和業務邏輯（按下去發生什麼事）應該分開寫，讓 AI 之後可以只改其中一個。

#### 4.2.2 如何有限度使用 Claude Code（第二階段）

如果這個階段你已訂閱 Claude Code，**不要讓它自由發揮**，而是給它明確、有邊界的任務：

| ✅ 適合交給 Claude Code 的任務 | ❌ 不適合的任務 |
|---|---|
| 「幫我整理這 100 行 Sales 的 CRUD 函式，重新命名並加中文註解」 | 「幫我把系統拆成多檔模組」 |
| 「幫我找出這段 CRM 代碼裡的重複邏輯，告訴我哪裡可以合併」 | 「幫我把 createElement 改成 JSX」 |
| 「幫我在現有格式下新增匯出 CSV 按鈕，不要改其他地方」 | 「幫我升級到 React 19」 |
| 「幫我檢查這段 Stock 的庫存計算有沒有 bug」 | 「幫我加 TypeScript」 |

#### 4.2.3 第二階段完成標誌

- [ ] 共用 helper 函式已集中
- [ ] 每個模組的 CRUD 操作已獨立成函式群組
- [ ] UI 元件與業務邏輯已初步分離
- [ ] 代碼內沒有「複製貼上的重複邏輯」

---

### 第三階段：架構現代化（長期，Claude Code 全力發揮）

> **目標：將 YESPC OS 遷移到 React + Vite + JSX 標準專案結構**  
> **工具：強烈建議訂閱 Claude Code 或 Cursor 協助執行**  
> **預估時間：4–12 週（依複雜度）**

#### 4.3.1 目標架構

```
yespc-os/
├── src/
│   ├── components/
│   │   ├── Sales/
│   │   │   ├── SalesModule.jsx
│   │   │   ├── SalesForm.jsx
│   │   │   └── SalesTable.jsx
│   │   ├── CRM/
│   │   ├── Repair/
│   │   ├── Stock/
│   │   └── Admin/
│   ├── hooks/
│   │   ├── useSales.js
│   │   └── useFirestore.js
│   ├── utils/
│   │   ├── formatters.js
│   │   └── validators.js
│   ├── firebase/
│   │   └── config.js
│   └── App.jsx
├── package.json
└── vite.config.js
```

#### 4.3.2 第三階段後，Claude Code 能幫你做的事

進入這個階段後，你可以對 Claude Code 說：

- **「幫我在 CRM 模組加一個客戶標籤篩選功能」**  
  → 它會自動修改 CRM 相關的多個檔案，測試，完成。

- **「幫我在所有有金額的地方加上千分位格式化」**  
  → 它會掃描整個 codebase，找到所有顯示金額的地方，統一修改。

- **「我想做多店版本，幫我加一個店家選擇器」**  
  → 它會分析影響範圍，提出計畫，逐步實作。

- **「幫我把 Sales 模組的資料層改成支援離線模式」**  
  → 它會只改 Sales 相關的檔案，不動其他模組。

---

## 五、Claude Code vs Cursor 選擇建議

| 比較維度 | Claude Code | Cursor |
|---|---|---|
| 使用方式 | 終端機指令，CLI 操作 | 編輯器內，視覺化操作 |
| 適合你的地方 | 習慣 CLI 工作流、想讓 AI 完全自主跑任務 | 想邊看邊改、審查 diff 再接受 |
| 對非工程師的友善度 | 較低（要在終端機打指令） | 較高（視覺化 diff，較安全） |
| 大規模改動安全性 | 強，但需要有 Git 才能 rollback | 強，每次改動可 Accept/Reject |
| 費用（2026 年參考） | 依 Claude API 使用量計費 | 月費制，含 fast requests |
| **建議** | **適合第三階段、熟悉 CLI 後** | **第二階段就可以開始試用** |

**我的建議：先試 Cursor（第二階段）→ 熟悉後考慮 Claude Code（第三階段）**

Cursor 因為是編輯器介面，你可以逐行審查 AI 改了什麼再決定要不要接受，對現在這個「還沒有測試保護」的架構來說，風險比 Claude Code 小很多。

---

## 六、現在最急迫的行動清單

以下是你今天就可以開始做的事，不需要任何工具訂閱：

### 本週可完成

- [ ] **建立 Git repo**：把現在的 index.html 先 commit 一版，這是所有後續工作的保險
  ```bash
  git init
  git add .
  git commit -m "YESPC OS 初始備份 $(date +%Y-%m-%d)"
  ```

- [ ] **記錄模組清單**：把上方表 2.2 的「模組 + 資料集」對照表貼進你的筆記，有需要補充就補充

- [ ] **記錄「最常改的功能 Top 5」**：哪 5 個功能你最常需要改？把它們列出來

### 近兩週可完成

- [ ] **在 index.html 加模組標記**：參考 4.1.1 範本，幫每個模組加清楚的區塊標記

- [ ] **整理全域 state 清單**：列出所有 `useState` 並標記哪個 state 給哪個模組用

### 一個月後的決策點

完成以上後，回來評估：

- 系統目前有多少行？（如果超過 8,000 行，第一階段完成就應該開始規劃第三階段）
- 每週大概改幾次代碼？（如果超過 3 次，Cursor 已值得訂閱）
- 最常改的功能，現在改一次要花多少時間？（如果超過 30 分鐘，工具化的 ROI 就已經正數）

---

## 七、給 Claude Code / Cursor 的標準 Prompt 模板

當你未來開始用 Claude Code 或 Cursor，每次任務開始都附上以下上下文說明，讓 AI 不會亂改架構：

```
【YESPC OS 架構說明】
這是一個單檔 React 18 + CDN 架構的 ERP/POS 系統
- 語法：React.createElement，非 JSX
- 執行環境：純瀏覽器，無 Node.js / Webpack / Vite 編譯
- 資料庫：Firebase Firestore（主）+ IndexedDB（本地）
- 模組：SalesModule / CrmModule / RepairModule / StockModule / AdminModule / FinanceModule
- state 集中在最外層 <App />，子模組透過 props 接收

【本次任務限制】
- 不要改動任何模組以外的全域 state
- 不要使用 import/export（此環境不支援）
- 不要改成 JSX 語法
- 只改明確說明的區塊，不要「順手改」其他地方
- 完整輸出修改後的代碼，不要只輸出片段

【本次任務】
（這裡填你這次真正想新增或修改的需求）
```

---

## 八、商業化 SaaS 前的必要準備

若你打算將 YESPC OS 包裝成商品販售給其他 3C 門市，以下是進入第三階段後必須處理的項目：

| 項目 | 說明 | 優先度 |
|---|---|---|
| 多租戶資料隔離 | 每家店的資料必須完全隔離 | 🔴 必須 |
| Firebase Security Rules | 防止任意讀寫，每個帳號只能讀自己的資料 | 🔴 必須 |
| 帳號/訂閱管理 | 客戶要能自行申請、付費、取消 | 🟡 重要 |
| 系統管理員介面 | 你要能管理所有客戶的帳號、授權、用量 | 🟡 重要 |
| 版本升級推送 | 系統更新後，所有客戶自動拿到新版本 | 🟡 重要 |
| 文件與操作說明 | 給客戶看的使用手冊 | 🟢 建議 |
| 客製化需求管理 | 不同門市的特殊需求如何處理 | 🟢 建議 |

這些功能，在完成第三階段的架構現代化之後，才能讓 Claude Code 有效協助你逐一實現。

---

## 九、總結建議

| 問題 | 建議答案 |
|---|---|
| 現在要不要訂閱 Claude Code？ | 不急，先做第一階段的地基整理 |
| 什麼時候訂閱 Cursor 最划算？ | 第二階段開始，代碼有基本分層後 |
| 什麼時候訂閱 Claude Code 最划算？ | 完成第三階段（Vite + JSX 架構）後 |
| 現在怎麼讓修改系統更省事？ | 繼續用 AI 助手幫你改單檔，每次改動都 git commit |
| 距離「讓 AI 全自動改代碼」還有多遠？ | 先做第一階段（約 2–4 週），第二階段完成後就能感受到明顯差異 |

**最核心的一句話：工具很強，但要讓工具發揮最大價值，你需要先給它一個它看得懂的家。** 而「整理這個家」這件事，我可以一直陪你做。

---

*本文件由 Perplexity AI 根據 YESPC OS 實際系統架構與楊店長需求生成 ｜ 2026-04-16*
