---
name: hedgehog-pet-maintainer
description: 維護、除錯與擴充「刺蝟養成記」單檔網頁遊戲（index.html）：調整狀態數值平衡、新增照顧動作、修改手繪刺蝟造型與姿勢、場景與日夜、存檔相容性，以及發版流程。使用者回報遊戲問題或要求改玩法、改外觀、發新版本時使用。
---

# 刺蝟養成記 維護指南

整個遊戲就是一個檔案 `index.html`（HTML + CSS + Canvas + 原生 JS），沒有任何依賴、沒有建置步驟。用瀏覽器直接開啟就能測試。

## 程式結構（index.html 由上到下）

| 區塊 | 重點 |
|---|---|
| `<style>` | 顏色用 `:root` 變數；按鈕、狀態條、頁尾樣式 |
| HTML | `#cv` 畫布（760×380）、`#gbar` 成長條、`#stats` 狀態條、`.actions` 九個按鈕、`#log` 紀錄、`#modal` 取名視窗 |
| 常數 | `W,H,FLOOR`、`SAVE`（存檔 key）、`SEC_PER_DAY=240`（真實 4 分鐘 = 遊戲 1 天）、`STAGES`（成長門檻與體型）、`PAL`（各階段身體／刺的顏色） |
| 狀態 | `DEFAULT` 物件；`S` 從 localStorage 讀入後 `Object.assign({},DEFAULT,…)` 合併 |
| `tick(dt, offline)` | 每幀的數值變化：睡眠回體力、飢餓／心情／清潔衰減、便便、健康增減、成長累積與升級 |
| 動作 | `FOODS`、`feed / play / pet / bath / cleanPoop / toggleSleep / doctor`，冷卻用 `ready(k)` / `setCool(k,s)`，`busy()` 時停用按鈕 |
| `updateUI()` | 狀態條、成長條、按鈕停用、時鐘（每 0.2 秒呼叫） |
| 繪圖 | `drawHog(x,y,s,dir,o)` 手繪線條風刺蝟（原點在腳底），三種姿勢：側面走路、正面發呆／被摸、縮成球；`drawScene` 背景與日夜；`draw` 合成 |
| `updateHog(dt)` | 刺蝟走動 AI、模式計時（`walk / eat / wheel / curl`）、自言自語 |
| 主迴圈 | `frame` → `tick` + `updateHog` + 粒子 + `draw`；每 5 秒存檔；`catchUp` 補算離線時間（最多 6 小時，離線衰減 ×0.5） |

## 常見修改

- **調數值平衡**：衰減速度在 `tick()`；動作效果在各動作函式與 `FOODS`；成長速度是 `S.growth+=.45*f*dt`，門檻在 `STAGES[].need`。改完要用不同情境實際試玩（白天／晚上、離線回來）。
- **新增照顧動作**：HTML 加按鈕 → 寫動作函式（改 `S`、設定 `hog.mode/modeT`、`burst()` 粒子、`say()`、`log()`、`setCool()`）→ 綁 `onclick` → 在 `updateUI()` 處理停用條件。同步更新 README 的動作表。
- **新增狀態欄位**：加進 `DEFAULT`，舊存檔會自動補上預設值；若要改變既有欄位的意義，才需要把 `SAVE` 版本號往上加（例如 `hedgehog-save-v2`），否則玩家進度會不見。
- **改刺蝟外觀**：只動 `drawHog` 與 `PAL`；保持「原點在腳底、`dir` 控制左右翻轉」的約定，三種姿勢都要檢查，四個成長階段（`STAGES[].scale`）也都要看。
- **場景**：`drawScene(t)`，日夜亮度由 `daylight()` 決定（8–17 點白天、20–5 點夜晚，中間漸變）。

## 測試

1. 瀏覽器開 `index.html`，取名後操作每個按鈕，確認數值、動畫、紀錄文字。
2. 要快速看成長或夜晚：在 DevTools Console 直接改 `S.growth`、`S.gameMin`（分鐘，0–1439）、`S.stage`。
3. 測離線：關掉分頁，在 Console 把存檔的 `last` 改早，或等幾分鐘再開。
4. 「重新開始」會清除 localStorage 存檔。

## 發版流程

1. 更新 `index.html` 頁尾的版本號（搜尋 `· v`）。
2. 在 `CHANGELOG.md` 最上方新增版本段落；玩法有變就同步 `README.md`。
3. `git commit`，再打附註標籤：`git tag -a vX.Y.Z -m "刺蝟養成記 vX.Y.Z"`。
4. 若有設定遠端：`git push origin master --tags`，並用 CHANGELOG 內容建立 GitHub Release。
5. 更新接管筆記的進度與下一步。

## 風格約定

- 介面文字、紀錄、刺蝟台詞一律繁體中文，語氣可愛口語。
- 維持單檔、零依賴；不要引入框架或建置工具。
- 程式碼風格是緊湊的單行寫法，修改時沿用既有風格。
