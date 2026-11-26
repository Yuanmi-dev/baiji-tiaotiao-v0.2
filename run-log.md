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
