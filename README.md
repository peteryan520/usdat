# usdat

USDat 套利指標儀表板是一個可直接部署到 GitHub Pages 的純前端靜態網站，用來追蹤 ETH 鏈 USDat/USDC、BNB 鏈 USDat/USDT，以及 Binance `USDCUSDT` 匯差，並估算扣除滑點、手續費、固定成本與安全門檻後的套利空間。

## 功能介紹

- 顯示 Binance `USDCUSDT` bid、ask、mid、24H high / low、24H volume 與更新時間。
- 顯示 ETH 鏈 `1 USDat = ? USDC`。
- 顯示 BNB 鏈 `1 USDat = ? USDT`。
- 計算 USDC/USDT 匯差、ETH USDat 脫錨、BNB USDat 對 Binance 公允價偏離、ETH 到 BNB 跨鏈價差與淨套利空間。
- 提供快速計算機，可調整投入金額、起始幣種、套利路徑、DEX 滑點與手續費、Binance 手續費、固定成本與安全門檻。
- 提供資料源設定 Modal，設定會儲存在瀏覽器 `localStorage`。
- 記錄本頁開啟後最近 30 筆淨套利空間，並畫成簡單折線圖。
- 不會自動下單，不需要後端，不需要 API Key。

## 資料源說明

### Binance Spot API

預設交易對：

```text
USDCUSDT
```

程式會依序嘗試以下 Binance base URL：

```text
https://api.binance.com
https://api1.binance.com
https://api2.binance.com
https://data-api.binance.vision
```

使用端點：

```text
/api/v3/ticker/bookTicker?symbol=USDCUSDT
/api/v3/ticker/24hr?symbol=USDCUSDT
```

### DEX Screener API

支援端點：

```text
/latest/dex/pairs/{chainId}/{pairId}
/token-pairs/v1/{chainId}/{tokenAddress}
/latest/dex/search?q=
```

ETH 鏈固定 pair 查不到時，會 fallback：

1. `token-pairs/v1/{chainId}/{tokenAddress}`
2. `search?q=USDAT USDC`
3. 挑選 Ethereum 鏈、quote/base 包含 USDC 與 USDat、流動性最高的池子

BNB 鏈固定 pair 查不到時，會 fallback：

1. `search?q=USDAT USDT`
2. 挑選 BSC 鏈、quote/base 包含 USDT 與 USDat、流動性最高的池子

程式會檢查 `baseToken` / `quoteToken` 方向：

- 若 `baseToken` 是 USDat 且 `quoteToken` 是 USDC 或 USDT，會把 `priceNative` 視為 `1 USDat` 的 quote 價格。
- 若 `quoteToken` 是 USDat 且 `baseToken` 是 USDC 或 USDT，會反向換算。
- 若方向無法判斷，會顯示資料不足，不會硬算。

## 預設池子地址

### ETH 鏈 USDat/USDC

```text
DEX：Curve Ethereum
Chain ID：ethereum
Pool Address：0xf4d0cf32908b2c7f1021339c43df0f77f06896d7
ETH USDat Token Address：0x23238f20b894f29041f48d88ee91131c395aaa71
交易對邏輯：USDat / USDC
```

### BNB 鏈 USDat/USDT

```text
DEX：PancakeSwap BSC
Chain ID：bsc
Pool Address：0xF80Ab3Cc041d8Ccc1c51AcC295AFdba26AD70Aa9
交易對邏輯：USDat / USDT
```

## GitHub Pages 部署方式

請先建立新的 GitHub repo：

```text
usdat
```

不要把這些檔案放進其他既有 repo。建立 repo 後，將本資料夾的檔案推上去：

```bash
git init
git add index.html README.md .nojekyll
git commit -m "Add USDat arbitrage dashboard"
git branch -M main
git remote add origin https://github.com/peteryan520/usdat.git
git push -u origin main
```

接著到 GitHub：

1. 進入 `usdat` repo。
2. 開啟 `Settings`。
3. 進入 `Pages`。
4. `Build and deployment` 選擇 `Deploy from a branch`。
5. Branch 選 `main`，資料夾選 `/root`。
6. 儲存後等待部署完成。

預期網址：

```text
https://peteryan520.github.io/usdat/
```

## 測試方式

本專案是純靜態網站，可直接用瀏覽器開啟：

```text
index.html
```

也可以用任一靜態伺服器測試：

```bash
python -m http.server 8080
```

開啟：

```text
http://localhost:8080/
```

測試重點：

- 點擊「立即刷新」後，三個資料源狀態是否更新。
- 點擊「資料源設定」是否可修改並儲存到 `localStorage`。
- 調整投入金額、bps、固定成本與路徑後，快速計算機是否即時更新。
- 若 API 無法讀取，畫面是否顯示中文錯誤訊息，而不是原始錯誤。

## 可能的 API 限制與 fallback

- Binance API 可能因地區、CORS、頻率限制或服務狀態暫時失敗。程式會依序嘗試 `api.binance.com`、`api1.binance.com`、`api2.binance.com`、`data-api.binance.vision`。
- DEX Screener 固定 pair 可能查不到或資料延遲。ETH 會 fallback 到 token-pairs 與 search；BNB 會 fallback 到 search。
- DEX Screener pair 方向若無法判斷，程式會顯示「Pair 方向無法判斷」，並暫停相關計算。
- GitHub Pages 是靜態頁面，無後端代理；若第三方 API 封鎖瀏覽器 CORS，前端無法繞過，只能等待服務恢復或改用自建代理。

## 如何修改資料源

在網站右上角點擊「資料源設定」，可修改：

- Binance symbol
- ETH chainId
- ETH USDat Token Address
- ETH USDat/USDC Pair Address
- BNB chainId
- BNB USDat/USDT Pair Address
- ETH 搜尋關鍵字
- BNB 搜尋關鍵字

按「儲存設定」後會寫入瀏覽器 `localStorage`，重新整理仍會保留。按「重設成預設值」可回復預設地址與搜尋條件。

## 風險提醒

本工具僅作為價格監控與套利試算，不構成投資建議。實際交易仍需考量滑點、MEV、跨鏈橋延遲、Gas、CEX/DEX 手續費、池子深度、合約風險、穩定幣脫錨、USDat 兌回限制與官方規則變動。

## 未來可擴充項目

- Telegram / LINE 通知
- 自訂套利門檻
- 歷史資料儲存
- 多池比較
- 加入 Curve / Pancake 官方 router quote
- 加入真實換匯與橋費
