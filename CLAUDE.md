# SLEEPER (codename)

俯視戰術射擊 × 附身式小隊操作 × Slay-the-Spire roguelite。**單一檔 `index.html`**（原生 JS + Canvas 2D、零外部相依，瀏覽器開檔即玩）。

## 接手前先讀
- **`開發進度與脈絡.md`** — 完整進度、架構、設計脈絡、CONFIG 對照、踩雷。**恢復實質工作前先讀這份，別重新摸索程式碼。**
- `SLEEPER_開發計畫.md` — 原始設計規格。

## 速記
- 所有可調數值/內容在 `index.html` 檔頂：`CONFIG` / `WEAPONS` / `ABILITIES` / `EQUIPMENT` / `ITEMS` / `ARCHETYPES` / `ENEMY_TYPES`(敵人類型) / `MAPS`(多張戰鬥地圖) / `ROADMAP`。
- 敵人＝資料驅動類型（`enemySpawns.type`，預設 grunt）：grunt/dog(警犬)/heavy_lmg/heavy_shot；重裝靠裝甲「機率整發擋下非穿甲」(武器 `pierce`)而非高血量。詳見 §4.16。
- 場景：`MAP / COMBAT / REST / ARMORY / RUNEND / RUNWIN / EDITOR`（有遊戲內關卡編輯器）。
- 完成的更新都 commit + push 到 GitHub `darkbearlab/sleeper_roguelite`（git 認證走系統 Git Credential Manager）。
- 改完一定要驗證：抽出 `<script>` 丟 Node `vm`、stub 掉 DOM/canvas/pointerlock/**localStorage**，**逐幀跑遍所有場景的 `update()+draw()`**（不要只在戰鬥跑）。
- 必踩雷：① `update()` 必須先判 `scene!=='COMBAT'` 再碰 `cam/players`（非戰鬥場景未建立）。② 迷霧用離屏 canvas 遮罩，**不可在主畫布 `destination-out`**（會擦掉世界）。③ `MAP` 已非單一 const 而是 registry 的作用中指標：衍生表(`N/adj/nodeVis/nextHop/EXTRACT/ENTRY`)由 `buildMapDerived(key)` 重建，別假設它們是載入期常數。
- 細節別寫進本檔（每 session 自動載入，保持精簡）；重內容放 `開發進度與脈絡.md`。
