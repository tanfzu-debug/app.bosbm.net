# 考古紀錄：www.bosbm.com（B2B2C 分銷電商遺址）

> 調查日期：2026-11-03
> 性質：資產定位與清點。**不是復活計畫。**

## 一句話結論

當年的網站程式碼**還在**，完整躺在 CEO 自己的 Google Drive 裡；
**資料庫資料已永久消失**（主機 2017 年後到期未續約，伺服器端歸零）。

---

## 1. 遺址位置

**Google Drive 資料夾 `www.bosbm.com`**
`https://drive.google.com/drive/folders/1bKaBlrKsJBy2s5uwzBENP9B-gh_VuoSv`
（Drive ID: `1bKaBlrKsJBy2s5uwzBENP9B-gh_VuoSv`，上傳於 2019-06-28）

**第二份拷貝：資料夾 `ShopNum1`**
`https://drive.google.com/drive/folders/1tok5oMNTHUjX9l4PHeWxD3OHvJVS29_2`
檔案時間戳 **2017-11-09** — 即網站最後上線狀態。

---

## 2. 系統身分

**ShopNum1 分銷門戶系統 V8.1.0**（武漢商之翼商用網店系統）

| 項目 | 內容 |
|---|---|
| 語言 / 框架 | C# + ASP.NET **Web Forms** |
| Runtime | .NET Framework 4.x（僅 Windows / IIS） |
| 資料庫 | SQL Server |
| 架構 | 三層（Interface / BusinessLogic / WebControl） |
| 多語系 | 簡繁雙語（`bin/zh-CHS`、`bin/zh-CHT`） |
| 第三方元件 | NeatUpload（上傳）、KindEditor（編輯器） |
| 版權檔 | 根目錄 `ShopNum1Copyright.cer` |

### 物證
CEO 於 2016-02-28 寄給當年合作方（Abe Lin）的信，主旨
「博士寶網站軟件包及跨境物流API」，內文即 `ShopNum1分销门户系统V8.1.0.zip`
的 Drive 連結，附件「跨境貿易文檔.rar」。

### 時間線
```
2016-02  取得 ShopNum1 V8.1.0 授權，開始建站
2017-11  最後一次程式改動（檔案時間戳）
~2018    主機到期未續約，網站下線
2019-06  CEO 將整包上傳 Google Drive 保存  ← 這個動作救了整批資產
2026-11  本次考古定位
```

---

## 3. 目錄結構（部署版）

```
www.bosbm.com/
├── bin/                 ← 編譯後 DLL + **PDB 符號檔**（關鍵）
│   ├── zh-CHS/  zh-CHT/
│   └── ShopNum1.*.pdb  （十餘顆）
├── Admin/               後台
├── SupplierAdmin/       供應商後台（B2B2C 的 2B 端）
├── Api/                 對外介面
├── Template/  Themes/   版型
├── Install/  Install_BAK/
├── BackUp/              （空）
├── images/ upload/ MemberImage/ face/
├── js/ css/ kindeditor/ DatePicker/ NeatUpload/
├── Settings/  log/  temp/
├── ShopNum1.Deploy.sln  ← Visual Studio 方案檔
└── *.aspx               ← 僅 57–96 bytes，純空殼（邏輯在 bin/*.dll）
```

### 可辨識的業務模組
會員註冊/登入、代理商註冊與授權（`AgentRegist`、`AgentValidity`、
`TbMemberAuthorization`）、購物車、訂單、電子卡/禮物卡（`ElcCardOrder`、
`MemberGiftCard`）、餘額與提現（`MemberBalance`、`MemberBalanceCash`）、
積分商城（`ScoreProductList`、`ScoreShoppingCart`）、批發（`Wholesale`）、
退貨退款維修（`MemberReturnGoods`、`MemberRefund`、`MemberReturnRepairs`）、
評價、問卷、留言板、簡訊介面、報表介面、`OrderService.asmx`。

---

## 4. 還原度評估

| 資產 | 狀態 | 備註 |
|---|---|---|
| 程式碼邏輯 | **90–95% 可還原** | 有 PDB → 反編譯帶回真實變數名與原始路徑 |
| DB schema | **可從 DLL 反推** | BusinessLogic / Interface 層含實體類別與 SQL 語句 |
| 部分原始碼 | **直接可用** | `ShopNum1OrderCommon.aspx.cs`(32KB)、`ShopNum1HttpModule.aspx.cs`(43KB) |
| `.bak` 原始碼備份 | **直接可用** | `ShoppingCart2.ashx.bak`(193KB)、`Fastbuy.ashx.bak`(208KB) 等，為未編譯原始碼 |
| 商品圖片 | **在** | `images/`、`upload/`、`MemberImage/` |
| 版型 / CSS / JS | **在** | |
| **DB 實際資料** | **永久消失** | 商品、訂單、會員資料 |

### 為什麼還原度這麼高
一般部署版只剩 DLL。這包額外帶了 **PDB 符號檔**、**`.sln` 方案檔**、
**部分未編譯 `.cs`**，以及大量工程師改檔前的 **`.bak` 備份** —— 遠超正常部署版。

### 還原方法
在 Windows 上用 **ILSpy**（免費開源）：
1. 把 `bin/` 整個資料夾取下（DLL 與 PDB 必須放在一起，ILSpy 會自動讀 PDB）
2. 載入後右鍵 → Save Code → 產出可用 Visual Studio 開啟的 `.csproj` 專案

---

## 5. 資料庫：已確認消失

查過且**全部為空或無相關檔案**：

- Drive 資料夾 `sql`（ID `116OKxPZRtkneVv6MINIB8cr50p1Gm7P9`）→ 空
- `www.bosbm.com/BackUp/` → 空
- `www.bosbm.com/Install/Scripts/` → 空
- `Install_BAK/` → 僅含 `Control/`
- 全 Drive 搜尋 `.sql` / `.bak` / `.mdf` → 唯一命中的 `.bak` 全為 js / ashx 小檔備份，無任何 SQL Server 備份

伺服器端已隨主機到期歸零，無可回溯。
**Schema 仍可從 DLL 反推，但資料本身無法復原。**

---

## 6. 技術判斷

此技術棧（ASP.NET Web Forms + .NET Framework 4.x）已停止演進：
不跨平台、無法在 BOSBM 現行的 LXD / Ubuntu 容器上運行、微軟僅提供安全維護。

**不建議原樣復活。** 真正可帶走的是設計知識，不是程式本身。

### 對 BOSBM 現行產品可能有參考價值者
1. **B2B2C 分銷商業邏輯** — 代理商分級與授權、傭金計算、積分商城、
   餘額提現、退貨退款流程。這是當年付費取得的 domain know-how。
2. **供應商端設計**（`SupplierAdmin/`）— 多賣家架構的欄位與權限劃分。
3. **DB schema** — 可作為新系統設計的參考底稿。
4. **圖片資產**。

---

## 7. 未追完的線索

| 線索 | 說明 |
|---|---|
| **舊硬碟** | CEO 另有舊實體硬碟待檢。優先找 SQL Server `.bak` / `.mdf`、2016–2018 的 zip/rar、`.sln` / `.csproj` 原始碼專案、`web.config` |
| 2016 信件中的舊 Drive 連結 | `0Bxmf7IF_C8qGYjlzVGV4WEI2LVk` — 原始安裝包，通常帶建表腳本，可能仍存活 |
| `Install_BAK/Control/` | 未展開 |
| `bin/` 完整 DLL 清單 | 未逐一列出，反編譯時再盤點 |

### 已排除
- Internet Archive / Wayback Machine — 本 session 網路政策封鎖，且該網域在公開搜尋引擎已無任何索引
- `bosbm.com` 本體 — DNS 無回應，主機早已釋出

---

## 8. 注意事項

- `Settings/` 與 `web.config` 內含 SQL Server 連線帳密。**不得 commit 進任何 repo。**
- ShopNum1 為商之翼商用產品。研究自有系統、取回當年客製部分屬合理範圍；
  若涉及整套重新發行或轉售，須先確認 2016 年授權合約條款。
