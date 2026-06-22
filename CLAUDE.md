# SLEEPER (codename)

俯視戰術射擊 × 附身式小隊操作 × Slay-the-Spire roguelite。**單一檔 `index.html`**（原生 JS + Canvas 2D、零外部相依，瀏覽器開檔即玩）。

## 接手前先讀
- **`開發進度與脈絡.md`** — 完整進度、架構、設計脈絡、CONFIG 對照、踩雷。**恢復實質工作前先讀這份，別重新摸索程式碼。**
- `SLEEPER_開發計畫.md` — 原始設計規格。

## 速記
- 所有可調數值/內容在 `index.html` 檔頂：`CONFIG` / `WEAPONS` / `ABILITIES` / `EQUIPMENT` / `ITEMS` / `ARCHETYPES` / `ENEMY_TYPES`(敵人類型) / `DESTRUCT`(可破壞地形形態) / `MAPS`(多張戰鬥地圖) / `ROADMAP`。
- **數值平衡面板（§4.21）**：按 `` ` `` 或右上齒輪開的 DOM 側欄，依物件結構自動生成滑桿/勾選、直接寫上列目錄的活值＝即時生效（武器是共用參考即時；彈夾/unitHp/敵人hp,armor 下次生成套用）。資料層(`tune*`/`applyTunerSaved`/`tuneOverrides`/localStorage 鍵 `sleeper_tuning_v1`)與 UI 層(`initTunerUI`，全包 try/catch、無 DOM 即停用)分離。新增 CONFIG 數值不必改面板。
- **獨立地圖編輯器 `editor.html`（§4.22，與 index 分檔）**：自足 DOM 工具，三分頁(地圖物件/waypoint+連線/敵人配置)、拖曳、連線編輯加強、邊穿牆紅標、與 MAPS 同結構雙向 JSON round-trip＋「MAPS 片段」貼回。**踩雷：刪 waypoint 必須重編號**(遊戲 `wp=id=>MAP.waypoints[id]` 以陣列索引當 id，連帶 edges/enemySpawns.nodeId)。editor.html 自帶一份小常數(ENEMY_TYPE_IDS/ENEMY_META/WP_TAGS/DESTRUCT_PRESETS/TEMPLATES)，改 index 對應內容時要手動同步。in-index 編輯器(§4.13)仍並存。
- 地形：牆/門/低矮牆/草叢(MAP.bushes)/**可破壞地形(MAP.destructibles：wall/glass/lowwall/fence/flat 兩形態+HP)**；五型預設(木箱/落地窗/有矮牆窗/全遮蔽/鐵網)。改地形阻擋邏輯看 segBlocked/collideWalls/onLowWall/updateBullets/canSee。詳見 §4.20。
- 敵人＝資料驅動類型（`enemySpawns.type`，預設 grunt）：grunt/dog(警犬)/heavy_lmg/heavy_shot；重裝靠裝甲「機率整發擋下非穿甲」(武器 `pierce`)而非高血量。詳見 §4.16。
- 側翼/突襲（§4.17，敵我皆適用）：繞背(目標視野錐外)＝傷害×2 且無視所有裝甲；閒置敵正面＝×1.5；`damageUnit` 第4參 `fromAng`。
- 潛行/警覺層（§4.19，Stage 1-5 全做完）：IDLE 敵短錐+有限視距(`canSee` 依狀態)+識別空窗(`enemy.detect` 累滿才 ENGAGED，永鎖；槍聲/被擊中瞬交戰)+overwatch 對 IDLE 收火；草叢(`MAP.bushes`/`inConceal`，MOBA 遮蔽，`drawEnemies` 依 `isRevealed`)；消音手槍(`WEAPONS.silpistol` silent，亞音速彈稀少)；望遠(右鍵 `isScoping()`，鏡頭前帶+穩定+禁奔跑，技能移空白鍵)；前哨站 `MAPS.outpost`。`CONFIG.stealth`/`scope`。
- 第0關/cutscene 導演（§4.18）：`initRun` 每局先 `startIntro()`（`intro` 地圖、scene COMBAT + `cutscene` 節拍導演，可 Esc 跳）→ 序章休息 → 路線圖。通用導演 say/move/face/wait/cam/sub/do/shoot/gate 可重用於未來劇情。改 update/draw/initRun 時注意 `cutscene`/`introActive` 分支。
- 場景：`MAP / COMBAT / REST / ARMORY / RUNEND / RUNWIN / EDITOR`（有遊戲內關卡編輯器）。
- 完成的更新都 commit + push 到 GitHub `darkbearlab/sleeper_roguelite`（git 認證走系統 Git Credential Manager）。
- 改完一定要驗證：抽出 `<script>` 丟 Node `vm`、stub 掉 DOM/canvas/pointerlock/**localStorage**，**逐幀跑遍所有場景的 `update()+draw()`**（不要只在戰鬥跑）。
- 必踩雷：① `update()` 必須先判 `scene!=='COMBAT'` 再碰 `cam/players`（非戰鬥場景未建立）。② 迷霧用離屏 canvas 遮罩，**不可在主畫布 `destination-out`**（會擦掉世界）。③ `MAP` 已非單一 const 而是 registry 的作用中指標：衍生表(`N/adj/nodeVis/nextHop/EXTRACT/ENTRY`)由 `buildMapDerived(key)` 重建，別假設它們是載入期常數。
- I/O 接縫（§14，為將來可能搬 Phaser 預留）：模擬層讀輸入只走 `Input.*`(held/fire/locked/aim)、出聲只走 `Sfx.play(id)`；DOM 事件處理＝轉接器。**新增戰鬥邏輯時別直接讀 `keys`/`mouse`/`pointerLocked`/`aimDir`，改用 `Input`**。render 門面/迷霧/相機等昂貴接縫尚未抽（仍直接用 `ctx`）。
- 細節別寫進本檔（每 session 自動載入，保持精簡）；重內容放 `開發進度與脈絡.md`。
