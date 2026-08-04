# 朋友圈 App — 專案說明

## 專案概述
一個用來記錄親朋好友近況的個人 App，支援手機和電腦使用。

## 技術架構
- **前端**：純 HTML + CSS + JavaScript（單一 index.html 檔案）
- **資料儲存**：Google Sheets（透過 Google Apps Script API）
- **本機快取**：localStorage（離線備援）
- **部署**：GitHub Pages

## 相關連結
- **App 網址**：https://webby123321.github.io/friend-app
- **GitHub Repo**：https://github.com/webby123321/friend-app
- **Google Sheets**：https://docs.google.com/spreadsheets/d/1Dcj4Gm-ZfFAgGKYcjUhqphmVYagGkm9uj_fIflbZRQg
- **Apps Script 網址**：https://script.google.com/macros/s/AKfycbwUF_-DI1y7OngxeYdLYcsjHs2EJKPtqf1g-CKTj8ApA-q5m9Z0gJJHYs2imYNlBxSckQ/exec

## 資料結構（Google Sheets）

### people 工作表（聯絡人）
| 欄位 | 說明 |
|------|------|
| id | 唯一識別碼 |
| name | 姓名 |
| cat | 分類 |
| createdAt | 建立時間（Unix timestamp） |

### notes 工作表（近況記錄）
| 欄位 | 說明 |
|------|------|
| id | 唯一識別碼 |
| personId | 對應的聯絡人 id |
| text | 記錄內容 |
| createdAt | 建立時間（Unix timestamp） |
| order | 排序順序（數字越小越前面） |

### activities 工作表（活動記錄）
| 欄位 | 說明 |
|------|------|
| id | 唯一識別碼 |
| date | 活動日期（Unix timestamp） |
| place | 地點 |
| what | 做了什麼 |
| people | 參與人員 id，逗號分隔 |
| createdAt | 建立時間（Unix timestamp） |

### statuses 工作表（目前狀態）
| 欄位 | 說明 |
|------|------|
| id | 唯一識別碼 |
| personId | 對應的聯絡人 id |
| key | 項目名稱（工作/租屋/住處或自訂） |
| val | 內容 |

## 分類設定
| 分類名稱 | 顏色 |
|----------|------|
| TCSH | #3D7EAA（藍） |
| NTOU EE | #2D6A4F（深綠） |
| NTOU ORCH | #9B5DE5（紫） |
| NYCU | #E07A5F（橘紅） |
| Tronfuture | #B7985A（金黃） |
| 親戚 | #E53935（紅） |

固定三個狀態欄位：**工作、租屋、住處**（可額外自由新增）

## 版本記錄

### v1
- 基本 HTML App，localStorage 儲存

### v2
- 串接 Google Sheets
- 新增匯出/匯入 JSON 功能

### v3
- 目前狀態區塊（工作、租屋、住處 + 自由新增）
- 近況記錄可上下調整順序
- 分類名稱更新（TCSH、NTOU EE 等）
- FAB 按鈕調整（朋友圈＋新增聯絡人、活動＋新增活動）
- 移除右上角新增按鈕
- 活動頁標題改為「活動」
- 近況記錄排序 Bug 修正

### v4
- 目前狀態列點擊整行編輯，移除旁邊按鈕
- 近況記錄點擊整張卡片編輯
- 目前狀態列固定高度 36px
- 聯絡人可刪除
- 近況記錄可刪除

### v5
- 活動可編輯和刪除
- 點擊活動進入編輯介面

### v6
- 首頁卡片改為一排三個
- 深色主題（Dark Mode）
- 配色改為 Okabe-Ito 色盤（色弱友善）
- 分類 Tab 改為整個填滿對應顏色（白字）
- 卡片背景改為各分類專屬深色（非透明疊色）
- 卡片左側加上分類對應亮色邊線
- 卡片時間文字改用 CSS 變數（修正深色模式下不可見的問題）
- 載入畫面背景改為深色

### v7
- 字體改為 Inter + Noto Sans TC（Google Fonts）
- Bug 修正：活動多人儲存至 Google Sheets 時只存第一人的問題（序列化為逗號字串再寫入）
- 活動頁人名 chip 依分類顯示對應顏色
- 分類「親戚」改名為「Relatives」，統一英文格式
- 手機左邊緣右滑返回手勢（swipe back）

### v8
- 新增第三個底部 Tab「更多」（漢堡圖示），三 Tab 等分
- 「更多」頁面：包含匯出／匯入資料、版本號與更新日期
- 匯出／匯入從首頁頂部移至「更多」頁
- 個人頁 section 標題：目前狀態→紀錄、近況記錄→近況、一起的活動→活動
- 活動編輯改為全頁面（不再是底部 sheet），返回正確回到來源頁
- 活動列表改為分隔線樣式（移除浮動卡片感）
- 底部導覽列拉高（iPhone safe area + 10px），避免被 Home indicator 遮住
- body 加 overflow-x: hidden，防止頁面左右滑動

## Apps Script 完整程式碼
```javascript
const SHEET_ID = '1Dcj4Gm-ZfFAgGKYcjUhqphmVYagGkm9uj_fIflbZRQg';

function doGet(e) {
  const sheet = SpreadsheetApp.openById(SHEET_ID);
  const data = {
    people:     getSheet(sheet, 'people'),
    notes:      getSheet(sheet, 'notes'),
    activities: getSheet(sheet, 'activities'),
    statuses:   getSheet(sheet, 'statuses')
  };
  return jsonResponse(data);
}

function doPost(e) {
  const body = JSON.parse(e.postData.contents);
  const sheet = SpreadsheetApp.openById(SHEET_ID);
  if (body.action === 'write') {
    writeSheet(sheet, 'people',     body.people);
    writeSheet(sheet, 'notes',      body.notes);
    writeSheet(sheet, 'activities', body.activities);
    writeSheet(sheet, 'statuses',   body.statuses || []);
    return jsonResponse({ ok: true });
  }
}

function getSheet(ss, name) {
  const sh = ss.getSheetByName(name);
  if (!sh) return [];
  const rows = sh.getDataRange().getValues();
  if (rows.length < 2) return [];
  const headers = rows[0];
  return rows.slice(1).map(row => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = row[i]);
    return obj;
  });
}

function writeSheet(ss, name, data) {
  let sh = ss.getSheetByName(name);
  if (!sh) sh = ss.insertSheet(name);
  sh.clearContents();
  if (!data || !data.length) return;
  const headers = Object.keys(data[0]);
  sh.getRange(1, 1, 1, headers.length).setValues([headers]);
  const rows = data.map(obj => headers.map(h => obj[h] !== undefined ? obj[h] : ''));
  sh.getRange(2, 1, rows.length, headers.length).setValues(rows);
}

function jsonResponse(data) {
  return ContentService.createTextOutput(JSON.stringify(data)).setMimeType(ContentService.MimeType.JSON);
}
```

## 注意事項
- 更新 Apps Script 後必須重新部署（管理部署作業 → 編輯 → 新版本 → 部署）
- 資料的 key 名稱（fcc_people、fcc_notes、fcc_acts、fcc_status）不能改，否則 localStorage 資料會讀不到
- 換手機前記得用「匯出」功能備份 JSON
- Claude 內建預覽無法連線 Google Sheets，需用瀏覽器開 GitHub Pages 網址才能正常使用
