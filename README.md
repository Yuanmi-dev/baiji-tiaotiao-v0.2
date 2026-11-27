# 白雞跳跳 v0.2

短局蓄力跳躍的可試玩雛形。公開示範：https://yuanmi-dev.github.io/baiji-tiaotiao-v0.2/

儲存庫：https://github.com/Yuanmi-dev/baiji-tiaotiao-v0.2  
這是 GitHub Pages 示範站，不是正式官網，也不會部署到 yuanmi.tw。v0.1 在上一層目錄的 `index.html`，請不要覆蓋它。

## 架構

以 `index.html` 為主（vanilla JS + Canvas 2D），背景圖在 `assets/bg/`，音效與配樂在 `assets/audio/`。玩法、每日種子、天氣物理、畫面、音效與廣告呼叫點都在同一份遊戲文件裡（Ad Placement API 規定廣告標籤與遊戲畫布要在同一份文件）。

- 邏輯解析度 390×844，實際畫面依高度縮放並橫向延伸。
- 物理以固定 60 tick 模擬；關卡由台北時間（Asia/Taipei）日期種子決定，同一天同一關。
- 廣告採 Google Ad Placement API（AdSense H5 遊戲廣告）預埋 `adBreak` / `adConfig`。尚無 AdSense 帳號時不要填 publisher id；預設 `CFG.ADS_ENABLED` 為關閉，開啟時走 `data-adbreak-test="on"` 測試模式。沒有廣告、被擋或回呼失敗時，遊戲會照常進行。

## 怎麼玩

按住蓄力、放開起跳。角色自動向前跑，跑到台邊仍按著會以當前力道自動起跳。觸控、滑鼠、空白鍵／上鍵都可用。右上角可關震動與聲音（聲音狀態存在 `localStorage` 鍵 `baijiMute.v02`；預設音量較低，第一次點擊後才解鎖 Web Audio）。月球與火星關卡仍用原本的程序地形，其他天氣會在同一組背景上加色調。

死掉後可再點重來；若測試廣告流程有開，也可「看廣告接續目前分數」（每局限一次，從最近安全平台重生並重跑倒數）。

## CFG 重要參數

都寫在 `index.html` 開頭的 `CFG` 物件：

| 參數 | 說明 |
| --- | --- |
| `GRAVITY`、`JUMP_VY_MIN`／`MAX`、`FALL_GRAVITY_MUL` | 重力與起跳；目前不要隨便改滯空手感 |
| `CHARGE_MS`、`RUN_SPEED` | 蓄滿時間、跑速 |
| `RAMP_PLATFORMS`、`REACTION_S`、`MIN_PLATFORM_LEN_FOR_CHARGE` | 前段暖身：前 6 台要夠長，才能反應並蓄滿 |
| `COUNTDOWN_S` | 開始與重開前的倒數 |
| `ADS_ENABLED` | 是否啟用廣告呼叫。正式投放前請保持 `false` |
| `ADS_TEST_MODE` | `true` 時加上 `data-adbreak-test="on"`，只顯示測試廣告 |
| `ADS_CLIENT` | 日後填入 `ca-pub-…`。現在請留空 |
| `ADS_INTERSTITIAL_EVERY` | 每完成 N 局，下一次倒數前插一次 interstitial（預設 3） |

## URL 參數

可組合使用，例如 `?weather=ice&debug=1&fakead=1`。

| 參數 | 說明 |
| --- | --- |
| `weather` | 強制天氣，如 `moon`、`ice`、`tailwind`、`storm` |
| `tod` | 強制時段視覺（`dawn`／`morning`／`noon`／`golden`／`sunset`／`dusk`／`night`），不改物理 |
| `fx` | 畫面品質：`high`／`medium`／`low`，不改物理 |
| `debug` | `1` 時顯示調參資訊（種子、蓄力、跳高、缺口） |
| `fakead` | `1` 時用本機模擬廣告流程，不向 Google 發請求；建議搭配 `debug=1` |
| `ads` | `1` 強制開啟廣告呼叫（仍受測試模式與空的 client 限制）；`0` 強制關閉 |

## 部署

1. 先 `git pull --rebase`，再改檔，一般 `git push`，不要 force push。
2. 把 `index.html`（以及 README、run-log）提交到本儲存庫的 `main`。GitHub Pages 由此分支發布。
3. 不要部署到 yuanmi.tw 或其他正式網域，除非另行決定。
4. 本機預覽可用 `python3 -m http.server 8000`，瀏覽器打開 `http://localhost:8000/`。測試廣告流程請用 `http://localhost:8000/?debug=1&fakead=1`。

要換成正式 AdSense 時：申請並通過 AdSense／H5 遊戲廣告、取得 `ca-pub-` 編號，填入 `CFG.ADS_CLIENT`，將 `ADS_ENABLED` 設為 `true`，並在確定可投放後把 `ADS_TEST_MODE` 改為 `false`（同時不要再輸出 `data-adbreak-test="on"`）。網域要加到 AdSense 網站，且廣告標籤必須與遊戲在同一份文件（若用 iframe 包住遊戲，標籤要放在 iframe 內，並加上 `allow="autoplay"`）。

驗證紀錄見 `run-log.md`。
