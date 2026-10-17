# Veldara 落地頁（ccdocs）

## 專案簡介

這是「Veldara」的產品落地頁。首屏用捲動驅動的影片當背景，影片還沒載入或載不到時，改用 WebGL 程序化星雲。整個網站只有一個 `index.html`，由 GitHub Actions 部署到 GitHub Pages。

頁面上的安裝指令 `npm i @veldara/core` 目前還沒有發布到 npm，頁面上已標示「Concept preview — not yet published on npm」。

預期網址（GitHub Pages 專案站）：https://wenbin2005.github.io/ccdocs/

## 檔案結構

| 路徑 | 說明 |
|---|---|
| `index.html` | 整個頁面，HTML、CSS、JavaScript 都在這個檔案 |
| `assets/favicon.svg` | 網站圖示 |
| `assets/og-image.png` | 分享卡圖（1200×630，由程式產生） |
| `.github/workflows/deploy-pages.yml` | 推送到 `main` 時部署到 GitHub Pages |

## 本機預覽

需要 Python 3，也可以用任何靜態檔案伺服器：

```bash
python3 -m http.server 8000
```

然後用瀏覽器開 http://localhost:8000/ 。

直接用 `file://` 開檔案也看得到版面，但瀏覽器可能擋掉字型與影片的網路請求，建議用本機伺服器預覽。

## 部署流程

- 觸發條件：推送到 `main`，或在 Actions 頁面手動執行（workflow_dispatch）。
- 步驟：checkout → 用白名單把網站檔案複製到 `_site/` → 上傳 Pages artifact → 部署。
- 白名單只有 `index.html` 和 `assets/`。要上線的新檔案請放進 `assets/`，放在其他地方的檔案不會被部署；`README.md` 和 `.github/` 也不會上線。

## 外部資源與授權

| 資源 | 來源 | 說明 |
|---|---|---|
| Inter 字型 | Google Fonts（fonts.googleapis.com、fonts.gstatic.com） | SIL Open Font License 1.1。用不擋首次繪製的方式載入，載入前先用調整過字寬的系統字型 |
| 捲動背景影片 | 第三方 CloudFront 網址（`index.html` 的 `VIDEO_URL`） | 不是本 repo 擁有的素材，授權與保存期限都未確認。連不到時頁面改用 WebGL 背景 |
| WebGL 星雲背景、分享卡圖、網站圖示 | 本 repo 用程式產生 | 沒有使用第三方圖片 |

每次瀏覽都會對 Google Fonts 和影片 CDN 發出請求，訪客的 IP 會送到這些服務。

## 背景與動態怎麼運作

1. 頁面一載入先顯示 CSS 漸層，WebGL 星雲畫好第一幀後淡入。
2. 同時用 `fetch` 下載影片一次（25 秒逾時），抽成最多 48 張、寬 720 的影格。
3. 抽格完成後由影片影格接手，隨捲動切換；WebGL 背景淡出並停止渲染。
4. 影片失敗（403、CORS、逾時或解碼錯誤）時維持 WebGL 背景；瀏覽器沒有 WebGL 時維持 CSS 漸層。

## 無障礙與效能原則

- 系統開啟「減少動態」時：
  - 不下載影片，背景是一張靜止畫面。
  - 不做位移、縮放、模糊動畫，只保留淡入淡出。
  - 頁內錨點瞬間跳轉。
- 分頁隱藏時，粒子和 WebGL 都停止渲染；WebGL 會依幀時間自動降低內部解析度。
- 停用 JavaScript 時，功能卡和結尾標題直接顯示。
- 鍵盤焦點環是 2px 亮藍；hero 淡出時，只要焦點在 hero 裡就會強制顯示。
- 地標用 `header`、`main`、`footer`；標題層級是 h1 → h2 → h3。

## 修改指南

| 想改的東西 | 在 `index.html` 的位置 |
|---|---|
| 文案（標題、卡片、頁尾） | `<main id="content">` 和 `<footer>` 裡的文字 |
| 品牌色 | CSS 裡的 `#2C5C88`（按鈕、底線）和 `#8cc4ff`（焦點環、亮色） |
| 動效曲線與時長 | CSS `:root` 的 `--ease-out`、`--dur-*`、`--stagger` |
| 背景影片網址 | JavaScript 的 `VIDEO_URL` 常數 |
| WebGL 星雲外觀 | JavaScript 的 `GL_FRAG` 著色器 |
| 分享卡文字與網址 | `<head>` 的 `og:*`、`description`、`canonical` |

改完後至少確認三件事：
- 桌機與手機寬度的首屏。
- 系統開啟「減少動態」時的畫面。
- 用鍵盤 Tab 走一輪，焦點都看得到。
