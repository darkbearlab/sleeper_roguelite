# SLEEPER (codename)

俯視戰術射擊 × 附身式小隊操作 × Slay-the-Spire roguelite。

**單一檔 `index.html`**：原生 JS + Canvas 2D，零外部相依、零資產，瀏覽器開檔即玩。

## 玩法概念
- 路線圖（Slay-the-Spire 式節點）→ 戰鬥 / 休整 / 軍械庫節點。
- 附身式操作：控制小隊單位，未控制者依個性（朝向×紀律×射擊模式）自動行動。
- 撤離型局內目標：到達撤離點、營救俘虜（→額外戰力）、限時。
- meta 庫存層：共用倉庫（容量上限）、背包、戰利品、技能/裝備/消耗品。
- 多張戰鬥地圖（門禁設施有門/矮牆/按鈕、敵人預設閒置）。
- 內建視覺化關卡編輯器（場景 `EDITOR`）。

## 開始遊玩
直接用瀏覽器開啟 [`index.html`](index.html) 即可，無需安裝或建置。

## 開發者文件
- [`開發進度與脈絡.md`](開發進度與脈絡.md) — 完整進度、架構、設計脈絡、CONFIG 對照、踩雷。**接手前先讀這份。**
- [`SLEEPER_開發計畫.md`](SLEEPER_開發計畫.md) — 原始設計規格。
- [`CLAUDE.md`](CLAUDE.md) — 精簡接手指標。

所有可調數值/內容集中在 `index.html` 檔頂：`CONFIG` / `WEAPONS` / `ABILITIES` / `EQUIPMENT` / `ITEMS` / `ARCHETYPES` / `MAPS` / `ROADMAP`。
