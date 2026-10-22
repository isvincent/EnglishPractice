# English Bloom｜每日英文學習花園

給高中學生的響應式每日英文教材網站。純 HTML / CSS / JavaScript，無需 npm、無需建置，首頁為 `index.html`。透過 Google Apps Script 公開唯讀端點，讀取既有「教材」工作表，保留原本每日 AI 生成與寄信流程。

## 檔案架構

```text
daily-english/
├── index.html
├── styles.css
├── app.js
├── config.js
├── manifest.webmanifest
├── sw.js
├── demo.json
├── README.md
├── TEST_REPORT.md
├── assets/
│   ├── garden.svg
│   ├── icon-192.png
│   └── icon-512.png
└── apps-script/
    └── WebApi.gs
```

## 先看示範

部署後按「開啟示範教材」，即可測試介面。示範資料明確標示，不會冒充你的試算表資料。尚未連接端點時，首頁會提示設定，不會自動把示範資料當成真實資料。

本機也可在專案目錄執行 `python3 -m http.server 8000`，再開啟 `http://localhost:8000`。請勿直接以 `file://` 開啟，示範讀取與 Service Worker 需要 HTTP(S)。

## 一、準備 Google 試算表

沿用每日寄信程式建立的試算表及 `教材` 工作表。標題順序應為：

|欄|標題|
|---|---|
|A|日期，純文字 YYYY-MM-DD|
|B|主題|
|C|單字|
|D|句型|
|E|閱讀與題目|
|F|翻譯或寫作|
|G|解答與解析|
|H|輸入Token|
|I|輸出Token|
|J|思考Token|
|K|寄送時間|

A～G分區完整呈現；「完整試算表欄位」可展開所有欄位，包括H～K及後續新增欄位，空白儲存格也保留。所有非空白列都會回傳，沒有只讀最新一筆或固定筆數限制。讀取的是儲存格顯示文字，不包含試算表原有圖片、格式、註解或公式本體。

程式以日期與列號辨識教材；請持續往下新增資料，避免任意插入或重新排序既有列，以維持本機收藏與進度的對應。

## 二、新增唯讀 Apps Script 端點

1. 從原試算表開啟「擴充功能 → Apps Script」。
2. 新增指令碼檔案，例如 `WebApi`，貼上 `apps-script/WebApi.gs` 的完整內容。
3. 保留原本 AI 生成、寄信、排程函式，不要整份取代。若原專案已有 `doGet`，必須整合為單一函式，不能重複宣告。
4. 確認「專案設定 → 指令碼屬性」的 `SPREADSHEET_ID` 指向正確試算表。原程式的 `initialize` 已會建立此屬性。
5. 選擇「部署 → 新增部署作業 → 網頁應用程式」。
6. 執行身分：我；誰可以存取：任何人（可匿名）。授權後部署。
7. 複製結尾為 `/exec` 的網址，不要使用 `/dev`。
8. 在登出 Google 或無痕視窗開啟該網址，應看到 `{"ok":true,"headers":...,"rows":...}`。
9. 若看到登入畫面、權限錯誤或 `ok:false`，先修正部署與試算表設定。

學校 Workspace 可能禁止匿名部署；遇到此限制需請管理員允許、改用允許公開部署的帳號並授予它讀取教材權限，或改採具備身分驗證的伺服器架構。不要將私密資料硬放入公開端點。

**公開資料範圍：**此端點讀取並公開「教材」工作表所有顯示值，包括Token、寄送時間與新增欄位。它不回傳指令碼屬性、Gemini API Key、收件人屬性或其他工作表。請確保教材工作表不含姓名、帳號、金鑰或其他不適合公開的內容。試算表本身不必設定為任何人可檢視，但部署身分需有讀取權限。

前端使用Google官方支援的JSONP方式讀取公開內容，避免GitHub Pages與Apps Script跨來源讀取差異；JSONP會執行來源腳本，所以只連接自己信任的Apps Script端點。後端限制callback名稱，教材內容以純文字安全呈現。

修改 Apps Script 後：部署 → 管理部署作業 → 編輯 → 新增版本 → 部署，保持原 `/exec` 網址。

## 三、設定網站資料來源

有兩種方式：

### 建議：在 config.js 預先設定

```javascript
window.APP_CONFIG = {
  endpoint: 'https://script.google.com/macros/s/你的部署ID/exec',
  refreshMinutes: 5
};
```

使用你的實際 `/exec` 網址，不含額外查詢參數。這不是API金鑰，公開教材端點可以放在前端。**不要把 Gemini API Key、寄信程式的指令碼屬性或其他憑證上傳 GitHub。**

### 或：從網頁操作

點「連接資料來源」，貼上網址並儲存。只對目前瀏覽器生效，優先於 `config.js`。如要切換端點，重新儲存即可。

## 四、上傳 GitHub Pages

1. 建立新的 GitHub 儲存庫，例如 `daily-english`。
2. 將解壓縮後資料夾裡的所有檔案及子資料夾上傳到儲存庫根目錄。確保 `index.html` 位於根目錄，不要多包一層資料夾。
3. `apps-script/WebApi.gs` 是部署用參考來源，GitHub Pages 不會執行它。
4. 到 Settings → Pages → Build and deployment。
5. Source 選 Deploy from a branch；分支選 main、資料夾選 / (root)，按 Save。
6. 等候GitHub完成部署，開啟 `https://你的帳號.github.io/daily-english/`。
7. 確認狀態顯示「即時同步」，筆數與試算表一致。若未設定網址，先連接資料來源。
8. 手動在試算表新增測試教材列，按「更新教材」確認出現後再刪除測試列。

CSS、JavaScript、圖片及PWA資源皆使用相對路徑，支援GitHub Pages子目錄。無第三方字型、CDN、外部圖片依賴。書本插畫及圖示為此專案自行製作。

## 五、更新規則

- 每次開站，立即嘗試讀取最新全部教材。
- 網頁在前景時，每5分鐘讀取一次；可在 `config.js` 調整。
- 切回分頁、網路恢復及手動按更新，也會讀取。
- Apps Script 直接讀取工作表，不另做伺服器資料快取。
- 成功後保留本機資料快取；失敗時明確顯示「離線／上次成功資料」與上次讀取時間，不會聲稱是即時資料。
- 關閉網頁時不保證背景同步；下次開啟且有網路時更新。PWA不會自行生成教材，原每日生成寄信排程仍需保留。
- 網頁讀取不呼叫Gemini，不額外產生AI Token費用，但受Apps Script執行與服務配額限制。

## 六、互動與七項設定

- 日期列表、全文搜尋、月份、排序、已收藏／已完成篩選。
- 收藏、完成進度、前後課切換、答案展開、自由作答筆記。
- 自動擷取前1、3、7天的單字複習；無該日期資料時不捏造內容。
- 所有教材原文保留換行與簡單 `**粗體**`。HTML字串不會被執行。
- 全螢幕：依瀏覽器支援；不支援時顯示提示。
- 音效：預設關閉，開啟後由使用者互動觸發短提示音，不自動播放。
- 裝置：自動、手機、平板、電腦；調整內容最大寬度，窄螢幕仍以可閱讀與不橫向溢出為優先。
- 版面：自動、直式、橫式；調整列表與內容排列，不強迫旋轉實體螢幕。
- 語言：繁體中文／英文介面；不自動翻譯試算表教材或使用者筆記，完整欄位標題維持來源原文。
- 字體：縮小16px、標準18px、放大21px，主要閱讀內容依設定縮放。
- 色系：系統、亮色、暗色；系統模式會隨系統變更。
- 尊重系統減少動態效果設定。插畫有輕微浮動動畫。
- 筆記、收藏、完成與顯示設定只存於目前瀏覽器，沒有跨裝置同步。清除網站資料會刪除這些紀錄。

列印只印目前教材；要印解答與完整欄位，請先展開對應區塊。按「列印 / 另存 PDF」使用瀏覽器列印。

## 七、PWA安裝與離線

- GitHub Pages提供HTTPS，符合Service Worker安全來源要求。
- 使用Web App Manifest、192/512 PNG圖示及Service Worker。
- Chrome/Edge符合安裝條件時，顯示「安裝應用程式」按鈕；也可用瀏覽器選單安裝。
- iPhone/iPad Safari：分享 → 加入主畫面。各裝置支援與選單可能不同。
- 至少成功開啟一次後，會快取網站介面與示範資料；成功同步過的教材另存於本機，離線可閱讀。
- 離線不能取得新的試算表資料；會清楚標示快取狀態。
- 更新檔案時可將 `sw.js` 的 CACHE 名稱改成 `english-bloom-v2` 等，以更新靜態快取。快取更新不會刪除筆記與學習紀錄。

## 常見問題

|情況|處理|
|網站顯示更新失敗|檢查/exec網址、匿名存取權限、網路及Apps Script執行紀錄|
|修改後資料沒出現|按更新；確認修改的是SPREADSHEET_ID指向的「教材」工作表|
|頁面出現登入畫面|端點未開放匿名存取，重新部署或詢問管理員|
|開啟網址顯示ok:false|確認SPREADSHEET_ID與工作表標題，至少需要A～G欄|
|進度跨裝置不同|本版為瀏覽器本機紀錄，非帳號同步|
|切英文後教材仍中文|只翻譯介面，保留原教材；若需教材翻譯要另加AI流程|
|部署後未顯示新版|重新整理；必要時更新sw.js快取版本或在開發工具取消註冊Service Worker|

## 官方參考

- Apps Script Content Service / JSONP：https://developers.google.com/apps-script/guides/content
- Apps Script Web Apps：https://developers.google.com/apps-script/guides/web
- PWA離線操作：https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Offline_and_background_operation

實際試算表URL與部署URL尚未提供，本套件不會連接任何猜測的資料來源；請依上述步驟設定並驗收。
