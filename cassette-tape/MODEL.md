# Cassette Tape — 匯入模型：拆解與替換指南

這份文件說明 `cassette-tape/` 這包 3D 檔案的結構，以及**怎麼替換文字、配色、材質**。
搭配載入頁：專案根目錄的 `cassette-gltf.html`（用 three.js `GLTFLoader` 載入）。

---

## 授權 / 出處（務必保留）
> “Cassette Tape” (https://skfb.ly/6QU7Y) by **joshtmc** — licensed **CC BY-SA 4.0**
> http://creativecommons.org/licenses/by-sa/4.0/

**CC BY-SA**：任何衍生（改文字/配色/重烤）都必須（1）標註原作者，（2）以**相同 SA 條款**釋出。
`cassette-gltf.html` 左上 credit 區塊已標註。

---

## 檔案清單
| 檔案 | 用途 | 網頁需要? |
|---|---|---|
| `cassette_tape.glb` | 主模型，**貼圖已內嵌**（單檔可獨立載入） | ✅ 頁面載這個 |
| `textures/mat_tape_albedo.jpg` | **baseColor / 顏色貼圖** → 文字與配色都在這 | 編輯用 |
| `textures/mat_tape_normal.jpeg` | 法線（表面凹凸、刻痕、螺絲） | 編輯用 |
| `textures/mat_tape_roughness.jpeg` | 粗糙度（光澤分佈） | 編輯用 |
| `textures/mat_tape_opacity.jpeg` | 不透明度遮罩（窗口/磁帶挖空） | 編輯用 |
| `textures/internal_ground_ao_texture.jpeg` | AO（陰影烘焙）— glb 材質未引用，可略 | 選用 |
| `source/model.zip` | 原始 **Collada (.dae)** 源檔 | Blender 重烤用 |
| `Cassette_Tape.usdz` | Apple AR 專用（gitignore，未追蹤） | ❌ |

> `.usdz` 被 `.gitignore` 忽略；`textures/` 與 `source/` 有刻意保留追蹤，就是給你改圖用。

---

## 模型結構速覽
- **幾何**：5,760 triangles / 3,837 vertices（很輕）。含 `NORMAL / TANGENT / TEXCOORD_0`（有 UV、有切線 → 法線貼圖正確）。
- **材質**：**只有 1 個** `mat_tape`（PBR）。貼圖對應：baseColor、metallicRoughness、normal。原始 `alphaMode: BLEND` + `doubleSided`。
- **節點**：`Sketchfab_model → visual scene group → Tape → mesh`（單一 mesh、單一 primitive）。
- **尺度**：包圍盒約 `2 × 1.27 × 0.24`（模型單位，寬 = 2）。載入頁會自動置中並縮放到最大邊 ≈ 11 單位（見 `TARGET`）。
- **朝向**：頂層節點已含 Collada(Z-up) → glTF(Y-up) 的轉正矩陣，載入即正立。

---

## 「拆」：什麼藏在哪張貼圖
| 你想改的東西 | 藏在 | 說明 |
|---|---|---|
| 文字：`SUPERTAPE` / `SIDE A/B` / `90` / `EQ-120µS` | **albedo (baseColor)** | 全是**畫上去的點陣**，不是程式字串 |
| 配色：綠標籤 / 紅帶 / 黑殼 / 米色 index | **albedo** | 顏色也畫在圖上 |
| 表面凹凸、螺絲、刻痕、磨損立體感 | **normal** | 改這個改「形」不改「色」 |
| 光澤/霧面分佈 | **roughness** | |
| 窗口、磁帶處的透空 | **opacity** | 載入頁把它從 BLEND 轉成 **alphaTest 硬邊裁切**（見下方 loader 說明） |

> 重點：**文字與配色不是 code，是貼圖點陣**。所以「改字改色」= **修圖**，不是改一行字串。
> （若要像 `mixtape.html` 那樣用字串/hex 秒改，需改走「程序化 canvas 貼圖」路線，見方法 D 之後的備註。）

---

## 「替換」：四種做法

### A. 改文字 / 配色（改圖，不用 3D 軟體）★ 最推薦
1. 用 **[Photopea](https://www.photopea.com)（免費網頁）** 或 GIMP 打開 `textures/mat_tape_albedo.jpg`。
   - 這是 **UV 攤平圖**：卡帶正面標籤在左半、內部帶盤在右側島。對照 `cassette-gltf.html` 畫面找到對應區塊。
2. 直接在標籤島上：蓋掉舊字、打你的新字（找相近的粗襯線/黑體）；用圖層填色換配色（保留磨損質感可用「柔光/色相」圖層）。
3. 存成同名 `mat_tape_albedo.jpg`（或 png）。
4. **重新打包 glb**（把新圖塞回去），用 `gltf-pipeline`：
   ```bash
   npm i -g gltf-pipeline
   # 先把 glb 拆成可讀的 gltf + 分離貼圖
   gltf-pipeline -i cassette_tape.glb -o unpacked/cassette.gltf --separate
   # 覆蓋 unpacked/ 裡對應的 baseColor 圖檔（用你改好的圖，檔名對齊）
   # 再打包回單檔 glb（貼圖內嵌）
   gltf-pipeline -i unpacked/cassette.gltf -o cassette_tape.glb
   ```
   > 載入頁不用改，重整即可看到新文字/配色。

### B. 打包更輕的 glb（減肥）
目前 glb 20MB，瓶頸在內嵌貼圖（幾何才 5.8k 三角形）。把 `textures/` 的 jpg 降尺寸（如 2048→1024）再用上面 `gltf-pipeline` 重打包即可大幅縮小，手機載入更快。

### C. 執行時換整張貼圖（做固定變體）
先做好 N 張不同文字/配色的 albedo（例如每顆 effect 一張），在 loader 載入後把 `material.map` 指向不同圖檔即可切換。**只需一個小勾子**，不動幾何。適合「幾種固定版本」。

### D. 從 Collada 源檔重烤（要 Blender）
解開 `source/model.zip` 取得 `.dae`，在 **Blender** 匯入 → 改材質/貼圖/甚至幾何 → 匯出 `.glb`。彈性最大、工最重。

> **備註（想要「秒改字色」）**：可保留這顆漂亮幾何，但把 baseColor 換成**程序化 canvas 貼圖**（像 `mixtape.html` 那樣用 JS 畫），文字就變字串、配色變 hex。代價是要先對準這顆模型的 UV 佈局畫一次。

---

## loader 端可調的旋鈕（`cassette-gltf.html`）
不改貼圖、只想調呈現時，這幾處：
- `TARGET`：模型置中後縮放到的最大邊尺寸（改大小）。
- 材質覆寫段落：載入後把 `alphaMode:BLEND` 轉成 `alphaTest = 0.5` 的**硬邊裁切**（修掉原本的半透明排序破圖）。
  - 若窗口邊緣太硬/太糊，調 `alphaTest`（約 0.3–0.6）。
  - 目前也設 `side = FrontSide` 停背面 overdraw；若某薄片消失可改回 `DoubleSided`。
- 燈光 / 環境（hemi / key / fill / rim + 程序化 env）：調反射與明暗。

---

## 常見問題
- **改了 `textures/` 的圖但畫面沒變？** 頁面載的是 `cassette_tape.glb`（內嵌貼圖），要**重打包 glb**（方法 A 第 4 步）或在 loader 端改用外部貼圖（方法 C）。
- **出現穿透/鏡像/浮空零件？** 那是原始 `alphaMode:BLEND` 的深度排序問題；loader 已用 alphaTest 修正，別把材質改回 `transparent:true`。
