# Run log：白雞跳跳 v0.2（2026-11-26，bot 實測）

- 基礎：另存自 `/workspace/webgame/index.html`（v0.1），沒有覆蓋原檔。
- 實作：`/workspace/webgame/v0.2/index.html`，仍是單檔 Canvas 2D + vanilla JS。
- 玩法改動：點擊拍翅穿管子 → 長按蓄力、放開起跳的平台跳躍。理由見 README。
- 種子：改為台北時間日期。此次驗證當日種子為 20261126。
- 震動：`navigator.vibrate` 有支援時輕震；iPhone 等不支援的裝置改畫面抖動；右上角有開關。API 只能設時長與節奏，不能調強弱。

## headless 驗證（Chrome，viewport 390×844，deviceScaleFactor 2）

- 語法：`node --check` 通過。
- 短按／最小蓄力與長按／蓄滿的跳躍高度確實不同：
  - `jumpWithCharge(0)`：起跳速度 640，最高點約 88 px
  - `jumpWithCharge(1)`：起跳速度 1200，最高點約 317 px
  - 實際按住 80 ms：蓄力約 0.11，最高點約 106 px
  - 實際按住 800 ms：蓄力約 0.96（接近蓄滿；跑到台邊會提早自動起跳），最高點約 304 px
- 遊戲可從開始玩到結束再重開：自動跑到台邊起跳並得分後進入結束畫面，用 pointer 事件重開，再開始新的一局。console 沒有 error。
- 截圖：`/workspace/baiji-tiaotiao-v0.2-gameplay.png`（遊戲中，已得分、可見平台與缺口）。
- 未做：真機觸控與震動尚未人工實測；手感參數尚未經人工調整。

## 參考技能後的補強（同一日稍後）

只把外部清單當參考，沒有執行裡面的安裝或腳本。實際採用並寫進程式的有：

- `platformer`：土狼時間 0.10 秒、跳躍緩衝 0.12 秒、下落重力 1.8 倍、下落速度上限。蓄力跳高本身仍用「起跳前蓄力」模型，沒有再混用「放開剪跳」。
- `physics-tuning`：固定 60 tick 模擬，單幀最多 5 步。
- `game-feel`：trauma 平方配正弦波的鏡頭抖動（只畫位移，不改物理）；落地擠壓；開關可關閉抖動。沒有加 hit-stop，避免操作像延遲。
- `camera-systems`：向跑向做指數平滑預視；抖動加在跟隨之後。
- `prototype-fast`：`?debug=1` 除錯 HUD、檔案開頭的 CFG、BRIEF.md。
- `game-ui-ux`：震動鈕與底部說明已避開 safe-area；沒有為此改引擎或重做整套 UI。
- `input-systems`：觸控與鍵盤走同一套按住／放開；忽略鍵盤連發。沒有做按鍵重綁或手把。
- 有讀但沒用上：`phaser-core`／`pixijs-rendering`（本專案維持 vanilla Canvas）、`itch-publish`（這次不上 itch，試玩已放 GitHub Pages）。

headless 再驗一次：最小跳最高點約 88 px、蓄滿約 317 px；按住 80 ms 與 800 ms 高度不同；可開始、結束、重開；console 無錯誤。

## 開始倒數（2026-11-26 稍後）

點擊開始與重新開始後，先跑 `CFG.COUNTDOWN_S`（目前為 3）秒倒數，畫面上依序顯示 3、2、1 並帶縮放。倒數期間角色與關卡凍結，按住不會蓄力、也不會起跳。支援震動且開關為開時，每跳一個數字輕震一下。

headless 驗證：開始倒數牆鐘約 3.00 秒、遊戲內計時 3.00 秒，數字為 3→2→1；期間座標與速度不變、無法蓄力或起跳；倒數結束後按住可蓄力。重開同樣倒數約 3 秒。震動開時 3 次 count 回饋，關閉時記錄為未震動。console 無錯誤。

## 放開起跳遲鈍感（2026-11-26 稍後）

真機回饋「按住到放開感覺有一點點遲鈍感」。排查後：輸入已是 `pointerup`（`passive: false`），頁面已設 `touch-action: none`，不是 300 ms 點擊延遲；土狼時間與跳躍緩衝在正常落地放開時不會多等一拍。放開當下 `vy` 已寫入（約 0 ms），但位置要等下一個 rAF 且累積器滿一個固定 tick 才上升。高更新率裝置上還可能先重繪（蓄力蹲下已清掉）再跑物理，看起來像先站直再起跳。

修正：放開若不在物理步內，立刻以 `CFG.JUMP_LAUNCH_DT`（與 `TICK` 同為 1/60 秒）跑完一步並重繪，同時清掉落地擠壓、改為彈起伸展（`JUMP_STRETCH` / `JUMP_STRETCH_S`），並噴起跳粉塵與上升拖影（`JUMP_DUST` / `JUMP_TRAIL`）。`JUMP_VY_MIN`／`JUMP_VY_MAX` 未改。

headless Chrome、viewport 390×844，每組 8 次（放開到位置開始上升）：

- 修正前
  - 短按 80 ms：msToMove 最小 14.70、中位數 15.05、平均 17.14、最大 31.90；幀數最小 1、中位數 1、平均 1.125、最大 2
  - 長按 800 ms：msToMove 最小 14.70、中位數 14.95、平均 14.99、最大 15.30；幀數皆為 1
  - `jumpWithCharge(0)`：msToMove 中位數 14.70，幀數 1
  - `jumpWithCharge(1)`：msToMove 中位數 15.00，幀數 1
- 修正後（放開同一事件內位置已上升，framesToMove 皆為 0）
  - 短按 80 ms：msToMove 最小 0.20、中位數 0.25、平均 0.27、最大 0.50
  - 長按 800 ms：msToMove 最小 0.10、中位數 0.10、平均 0.15、最大 0.30
  - `jumpWithCharge(0)`：msToMove 最小 0.10、中位數 0.10、平均 0.14、最大 0.20
  - `jumpWithCharge(1)`：msToMove 最小 0.20、中位數 0.20、平均 0.22、最大 0.30
- 跳高（修正前後皆同）：蓄力 0 為 87.83 px（vy 640）；蓄力 1 為 317.33 px（vy 1200）

## 左右留白與底部說明被擋（2026-11-27）

真機（iPhone Chrome）畫面以固定 390×844 比例置中，左右出現淺藍色留白，底部說明也被工具列擋住。

做法：改以高度為基準（邏輯高仍為 844），用 `100dvh` 與 `visualViewport` 讓畫布填滿可見區域，並處理 safe-area。天空、山、地面、雲依寬度橫向延伸，不拉伸、不用色塊補邊。玩法公平採用「前方可視距離固定」：`CFG.VIEW_AHEAD` 為 280（與原設計 390-110 相同），多出來的寬度加在雞的後方；超過 `CFG.MAX_ASPECT`（2）的部分只延伸背景，不加看前方。平台生成仍只依每日種子，與螢幕寬度無關。底部說明改放進開始卡與結算卡。

headless 截圖（console 無錯誤；四種尺寸左右皆無留白；四種寬度平台座標相同；前方可視距離約 280）：

- `/workspace/baiji-v02-fill-390x844.png`
- `/workspace/baiji-v02-fill-430x932.png`
- `/workspace/baiji-v02-fill-375x667.png`
- `/workspace/baiji-v02-fill-844x390.png`
