# OCTAVE 卡帶：標籤重貼圖 — 正規流程建議

## 背景與教訓
- 匯入模型的 `SUPERTAPE` 是**烤進 albedo 的像素**，不是圖層也不是字串（見 [MODEL.md](MODEL.md)）。
- 純程式化「蓋色塊」或 PIL「仿製填補」，在**沒有瀏覽器、無法目視 3D 渲染**的情況下盲改，有品質天花板：不是留字腳、就是留輪廓/接縫，**沒完沒了**。
- 結論：**像素級清理該用「人＋修圖工具」一次做乾淨；參數化（顏色/文字）交給程式。** 這是分工，不是方向錯。

## 三種正規做法

### A. 修圖軟體做一次乾淨底圖（★ 推薦，最快到完美）
用 **Photopea（免費網頁版 PS）/ Photoshop / GIMP** 對 `textures/mat_tape_albedo.jpg`，用**內容感知填滿 / 修復筆刷 / 仿製章**把 SUPERTAPE 抹掉——人眼即時看，幾分鐘完美。存成 `textures/mat_tape_blank.jpg`。之後換色/放字/換字型全部由程式處理（乾淨底圖上零鬼影）。

### B. Blender 拆材質（資產級）
把標籤獨立成自己的材質/UV，之後標籤能「換零件式」整張替換，最彈性；需要 Blender、工最重。

### C. three.js decal 疊層（不動原檔）
在標籤位置疊一張**自己畫的完整標籤**（程序化 canvas），原 SUPERTAPE 100% 被蓋 → 無「抹除」問題。全程式控制，但需對齊，且開發者端無瀏覽器不易目視驗證。

## 推薦分工（走 A）
1. **人**：Photopea 對 albedo 抹掉 SUPERTAPE，存 `mat_tape_blank.jpg`。
2. **程式（我接手）**：綠色標籤換色（hue）、畫品牌字（Oswald / 自選字型）、產 6 個 effect 變體，或改成執行時 canvas 即時上色＋畫字。

## SUPERTAPE 座標（4096² 的 albedo）
> 只抹 `SUPERTAPE`（含右接的小字 `®Realistic`）；**保留**右側的 `HIGH OUTPUT / LOW NOISE` 與紅底 `90`。

| 標籤 | x 範圍 | y 範圍 |
|---|---|---|
| Side A | 285 – 1850 | 1450 – 1710 |
| Side B | 300 – 1850 | 3320 – 3545 |

## Photopea 操作步驟（方法 A）
1. 開 https://www.photopea.com → File > Open → `textures/mat_tape_albedo.jpg`。
2. 用矩形/套索圈住上表座標的字樣區。
3. **Edit > Fill > Content-Aware**（或用左側 Spot Healing / Clone Stamp 沿綠底仿製）。
4. 兩個標籤都做；檢查放大無殘影。
5. File > Export as > JPG（品質 90+），命名 `mat_tape_blank.jpg` 放回 `textures/`，告訴我即可。

## 程式端現況（已就緒，等乾淨底圖）
- `octave-cassette.html`：載 glb + 換 baseColor 貼圖 + 用 opacity 當 `alphaMap` 挖窗；貼圖需 `flipY=true`（glb 內嵌 baseColor 是垂直翻轉版）。
- 換色 = 綠色像素 hue 位移；畫字 = canvas/PIL `fillText`。
- **待定**：標籤大字要「固定 OCTAVE」還是「每顆 effect 各自名稱」（一行切換）。

## 備註
- 本環境無瀏覽器，3D 呈現需在 GitHub Pages 上由使用者目視驗證。
- 授權 CC BY-SA（joshtmc），衍生同樣以 SA 釋出、保留 credit。
