# 碳吉 TanJi：低碳社區標章認證系統 🌿

## 1. 專案介紹

### 1.1 系統目的簡介

本系統旨在協助社區管委會、物業管理公司或規劃師，簡化並加速申請「115 年度新北市低碳社區標章認證」與「低碳社區改造補助」的流程。透過「低碳自評快篩」、「照片工作台」與「表四佐證編輯器」三大核心功能，將繁瑣的法規指標轉化為直覺的勾選介面，即時試算落點等級與補助金額，並能一鍵匯出符合官方格式的 Word 申請文件，大幅降低社區參與淨零碳排的門檻。

---

## 2. 系統架構與範圍

### 2.1 系統架構圖

本系統採用 **純前端 (Client-side) 架構** 設計，所有運算與檔案生成皆於使用者瀏覽器端完成，確保隱私與操作流暢度。

```mermaid
graph TD
    %% 定義樣式顏色
    classDef client fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px,color:black
    classDef external fill:#e3f2fd,stroke:#1565c0,stroke-width:2px,color:black
    classDef logic fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:black
    classDef data fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:black

    %% 1. 用戶端環境
    subgraph Client_Zone [用戶端環境 - 前端展示與運算層]
        Browser[("使用者瀏覽器<br>HTML5 / Tailwind CSS")]:::client
        
        %% 業務邏輯層
        subgraph Logic_Layer [核心運算模組 (Vanilla JS)]
            Score_Engine["標章計分與補助運算引擎"]:::logic
            Workspace["照片工作台<br>(Drag & Drop API)"]:::logic
            Docx_Engine["申請書生成引擎"]:::logic
        end

        %% 資料儲存層
        subgraph Data_Layer [本地資料層]
            Local_Storage[("瀏覽器暫存<br>LocalStorage")]:::data
            JSON_IO[("JSON 匯入/匯出模組")]:::data
        end
    end

    %% 2. 外部依賴
    subgraph External_Libs [外部函式庫 (CDN)]
        Tailwind[("Tailwind CSS")]:::external
        DocxJS[("Docx.js")]:::external
        FileSaver[("FileSaver.js")]:::external
    end

    %% 3. 資料流向連線
    Browser -- "1. 填寫指標/參數" --> Score_Engine
    Score_Engine -- "2. 即時計算落點/獎金" --> Browser
    Browser -- "3. 匯入/拖曳照片" --> Workspace
    Browser -- "4. 編輯表四狀態" --> Docx_Engine
    Score_Engine & Workspace -- "5. 狀態同步" --> Local_Storage
    JSON_IO -- "6. 存取備份檔案" --> Local_Storage
    Docx_Engine -- "7. 呼叫產出文件" --> DocxJS
    DocxJS -- "8. 下載 Word 檔" --> FileSaver

```

### 2.2 系統範圍

* **展示層**: 使用 Tailwind CSS 打造現代化響應式介面，支援互動式摺疊面板與照片對照視窗。
* **業務邏輯層**: 包含 115 年度六大指標（綠建築、綠色能源、資源循環等）的配分權重邏輯、補助款天花板計算，以及 Drag & Drop 照片拖曳整合。
* **資料存取層**: 透過 `LocalStorage` 實現進度保留，並整合 `FileSaver.js` 實現 JSON 實體檔案備份與 `docx.js` 文件產出。

### 2.3 交付項目

1. **網頁應用程式**: 單一 `index.html` 檔案（內含樣式與應用邏輯）。
2. **文件匯出模組**: 內建之表四佐證資料轉換腳本。
3. **系統規格文件**: 本 README 說明。

---

## 3. 業務功能需求

| 需求編號 | 功能名稱 | 參與者 | 功能描述 | 業務邏輯/備註 |
| --- | --- | --- | --- | --- |
| **FR-01** | **低碳指標自評** | 管委會/規劃師 | 輸入戶數與屋齡，並勾選六大類別指標現況，系統即時試算總分與標章等級（白金/金熊/銀鵝）。 | 自動判斷最高補助金額與適用規模（小型/中型/大型社區）。 |
| **FR-02** | **照片工作台** | 管委會/規劃師 | 提供隱藏式側邊欄，支援多圖匯入、滾輪縮放、平移檢視，並可直接拖曳照片至佐證編輯器。 | 使用 HTML5 Drag and Drop API 與 Canvas 變換矩陣。 |
| **FR-03** | **佐證資料編輯器** | 規劃師 | 針對各得分項目，設定「已有成果」、「擬改造」或「不適合」，並動態展開對應的照片拖曳區與文字說明框。 | 針對不同狀態動態切換必填提示與欄位（如擬改造示意圖）。 |
| **FR-04** | **申請書自動生成** | 規劃師 | 點擊「生成 Word 檔」，系統自動將勾選項目、說明文字與拖曳的照片打包，轉出為政府規定格式的 `.docx`。 | 採用 `docx.js`，照片會自動計算長寬比例縮放至適合 A4 版面。 |
| **FR-05** | **進度儲存與匯入** | 使用者 | 提供「儲存進度」與「匯入紀錄」功能，將表單狀態打包成 JSON 檔下載或載入。 | 避免瀏覽器關閉造成資料遺失，方便跨裝置接續編輯。 |

---

## 4. 非業務功能需求

### 4.1 安全性要求

* **無伺服器架構**: 所有社區資料（包含照片與地址）僅保留於使用者的瀏覽器與本機記憶體，**完全不回傳雲端**，無個資外洩風險。
* **離線作業支援**: 頁面載入 CDN 資源後，核心計算與 Word 產生邏輯皆可於斷網環境下繼續執行。

### 4.2 系統效能

* **輕量化設計**: 採用原生 JavaScript (Vanilla JS) 搭配 CDN，免去複雜的打包編譯，開啟速度極快。
* **圖片處理**: 生成 Word 前，系統會自動將拖曳的照片轉換為 Base64 Buffer 並按比例縮減尺寸，避免產出的文件檔案過大造成當機。

### 4.3 可用性與準確性

* **響應式與操作體驗**: 支援手機、平板的漢堡選單，並在桌機版提供「對照模式（Compare Mode）」並排顯示工作台與編輯器。
* **評分精準度**: 嚴格對應新北市環保局 115 年度低碳社區認證之最新計分權重與補助級距。

---

## 5. 系統介面設計

### 5.1 本地資料結構 (Local Storage Schema)

系統主要依賴瀏覽器 `LocalStorage` (`tanji_data`) 進行資料暫存與匯入/匯出：

```json
{
  "households": 150,
  "age": 10,
  "checkboxStates": [true, false, true, ...], 
  "radioStates": {
    "green_coverage": "2",
    "lighting_gen": "3",
    "e_home": "4",
    "ev_scooter": "1"
  },
  "plan_budget": ""
}

```

### 5.2 Word 產出規格 (Docx.js Mapping)

* **版面設定**: A4 橫向 (Landscape)。
* **表格結構**: 固定的表四格式，包含「項目」、「社區現況」、「自評得分」與「初審得分」欄位。
* **圖片轉換**: 若狀態為「已有成果」或「擬改造」，系統讀取拖曳圖片的 Base64，限制最大寬高為 `290px` 寫入 Word Run。

---

## 6. 專案安裝與部署

### 前置需求

* 現代瀏覽器（Google Chrome, Edge, Safari）。
* 無須安裝 Node.js 或任何後端環境。

### 部署步驟

1. **取得原始碼**: 下載專案的 `index.html`。
2. **本地執行**: 直接雙擊 `index.html` 於瀏覽器開啟即可完全正常使用。
3. **線上託管 (選擇性)**:
* 可將此單一檔案上傳至 **GitHub Pages**、**Vercel** 或任何靜態網頁伺服器 (Static Server)。
* 請確保網域允許載入外部 CDN（`cdn.tailwindcss.com`, `unpkg.com`, `cdnjs.cloudflare.com`）。
