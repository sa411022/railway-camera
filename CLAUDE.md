# SA Railway — 監視器牆

用來同時播放 1～8 個即時路況／鐵道沿線監視器的網頁，部署在 GitHub Pages。
使用者以繁體中文溝通，回覆也請用繁體中文。

## 工作方式

- **直接在 `main` 上 commit 並 push，不要開分支，也不要開 PR**（除非使用者明確要求）。
- **每次修改前先 `git pull origin main`**：使用者會自己在 GitHub 上手動修改檔案（尤其是 `camera.jsonc`）。
- 修改 `camera.jsonc` 時，**只動被要求的那幾行**，其他行（包含註解掉的監視器、順序、名稱）保持原樣。
- 「啟用」某支監視器 = 拿掉該行開頭的 `// `；「停用」= 在行首加上 `// `。
  同名的監視器可能有好幾支（例如「談文=大山」有兩支），使用者會用「第幾支」指定，依檔案中出現的順序計算。
- **「啟用預設的監視器」= 只啟用下列八支，其餘全部停用**（同名的有好幾支，請以網址辨認）：
  1. 成功(追分)=彰化 — `https://cctvc.freeway.gov.tw/abs2mjpg/bmjpg?camera=269&t=`
  2. 通霄=苑裡 — `https://cctv-ss04.thb.gov.tw/T1-139K+850?resolution=CIF_352X288&t=`
  3. 談文=大山 — `https://cctvc.freeway.gov.tw/abs2mjpg/bmjpg?camera=582&t=`（第二支）
  4. 竹南=談文(造橋) — `https://cctv-ss03.thb.gov.tw/T1F-003K+650?resolution=CIF_352X288&t=`
  5. 桃園=內壢 — `https://cctvtraffic.tycg.gov.tw/hls/nvr2/camera067/live.m3u8`（HLS 那支）
  6. 南樹林=樹林 — `https://cctvatis5.ntpc.gov.tw/hls/C000217/live.m3u8`
  7. 大甲=台中港 — `https://cctv-ss03.thb.gov.tw/T1-156K+650?resolution=CIF_352X288&t=`
  8. 埔心=平鎮 — `https://cctvtraffic.tycg.gov.tw/hls/nvr3/camera135/live.m3u8`
- 名稱找不到完全相符時（例如名稱前後顛倒），先找最可能的那支並在回覆中說明，不要自行改名。
- 密集 push 時 Pages 部署會排隊，部署由 `.github/workflows/pages.yml` 負責（進行中的部署不會被取消）。

## 檔案結構

- `index.html` — 整個網頁（純 HTML + JavaScript，不需建置）。從 jsdelivr 載入 dayjs 與 hls.js。
- `camera.jsonc` — 監視器設定檔，使用者隨時會修改。
- `.github/workflows/pages.yml` — GitHub Pages 部署（Source 需設為 GitHub Actions）。
- `CNAME` — 自訂網域；`.nojekyll` — 跳過 Jekyll。

## camera.jsonc 格式

```jsonc
[
  { "name": "成功(追分)=彰化", "url": "https://.../bmjpg?camera=269&t=", "type": "img" },
  // { "name": "桃園=內壢", "url": "https://.../live.m3u8", "type": "hls" },
]
```

- 允許 `//`、`/* */` 註解、結尾逗號、未加引號的 key、單引號字串（由 `index.html` 的 `parseLooseJson()` 解析）。
- `type`：`"img"` 用 `<img>` 播放，每 6 秒把網址中的 `t=` 設為目前 timestamp 重新連線；
  `"hls"` 用 `<video>` + hls.js 播放。沒寫 `type` 時，網址是 `.m3u8` 就當 hls，其餘當 img。
- 只播放前 8 支未註解的監視器（`index.html` 的 `MAX_CAMS`）。

## 網頁行為（修改時請維持）

- 每 6 秒重新讀取 `camera.jsonc`；`url` 與 `type` 都沒變的監視器沿用，不中斷播放。JSON 壞掉時保留目前畫面。
- 依螢幕比例自動選擇最佳的行列排版，旋轉／縮放時重新排版。
- 每列有一條標題列：上半部的列放在上方、下半部的列放在下方（靠近螢幕邊緣）；名稱之間顯示目前時間，欄位窄時只顯示 `HH:mm:ss`。
- 畫面以 `object-fit: fill` 填滿格子；網頁標題為「SA Railway」。

## 測試

本機測試：在 repo 根目錄執行 `python3 -m http.server`，再開 `http://localhost:8000/`（不能用 `file://` 開啟，因為要 fetch 設定檔）。
