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
