# SLEEPER (codename)

俯視戰術射擊 × 附身式小隊操作 × Slay-the-Spire roguelite。**單一檔 `index.html`**（原生 JS + Canvas 2D、零外部相依，瀏覽器開檔即玩）。

## 變體（規劃中、尚未動工）
- `extraction_arcade/` — 2D「搜打撤」arcade 變體，將由本遊戲**複製分家**獨立維護（非共用引擎）。**目前只有設計文件 `extraction_arcade/設計規劃.md`，尚未動工**；非經使用者明確要求別開工。本目錄與 SLEEPER 主程式無關，改 index.html 時可忽略。

## 接手前先讀
- **`開發進度與脈絡.md`** — 現況架構、系統、CONFIG 速查、踩雷（2026-06-24 重整精簡版）。**恢復實質工作前先讀這份，別重新摸索程式碼。**
- `封存/` — 舊版完整逐功能紀錄/時間線/舊 §4.x 編號（需考古再翻；現況以上面那份為準）。
- `SLEEPER_開發計畫.md` — 原始設計規格。

## 速記
- 所有可調數值/內容在 `index.html` 檔頂：`CONFIG` / `WEAPONS` / `ABILITIES` / `EQUIPMENT` / `ITEMS` / `ARCHETYPES` / `ENEMY_TYPES`(敵人類型) / `DESTRUCT`(可破壞地形形態) / `MAPS`(多張戰鬥地圖) / `ROADMAP`。
- **數值平衡面板（§4.21）**：按 `` ` `` 或右上齒輪開的 DOM 側欄，依物件結構自動生成滑桿/勾選、直接寫上列目錄的活值＝即時生效（武器是共用參考即時；彈夾/unitHp/敵人hp,armor 下次生成套用）。資料層(`tune*`/`applyTunerSaved`/`tuneOverrides`/localStorage 鍵 `sleeper_tuning_v1`)與 UI 層(`initTunerUI`，全包 try/catch、無 DOM 即停用)分離。新增 CONFIG 數值不必改面板。
- **獨立地圖編輯器 `editor.html`（§4.22，與 index 分檔）**：自足 DOM 工具，三分頁(地圖物件/waypoint+連線/敵人配置)、拖曳、連線編輯加強、邊穿牆紅標、與 MAPS 同結構雙向 JSON round-trip＋「MAPS 片段」貼回。**v2**：★特徵庫(`FEATURES` 柱子/房間/長廊/轉角/窗牆/掩體叢/十字路口，拖框 `stampFeature` 放置成扁平物件)、**框選群組** `selRefs`(marquee+8 控制點整組縮放/位移，**不存持久 `_gid`**、格式仍與遊戲一致)、重疊放置+點擊循環 `hitObjectsCycle`、**門切牆缺口** `cutWallsForDoor`、**參考底圖** (第4分頁「底圖」，`underlay` 墊圖描繪可旋轉，editor-only 不匯出，存 `sleeper_editor_underlay`)、**地圖尺寸可調** (`map.width/height` 真欄位+標題列輸入+「外牆」重建；遊戲相機不夾 WORLD 故本就支援大地圖)、**新增 waypoint 自動連線** (`autoLink` 開關+`autoLinkWaypoint` 連所有中間無阻隔點)。**踩雷：刪 waypoint 必須重編號**(遊戲 `wp=id=>MAP.waypoints[id]` 以陣列索引當 id，連帶 edges/enemySpawns.nodeId)。editor.html 自帶一份小常數(ENEMY_TYPE_IDS/ENEMY_META/WP_TAGS/DESTRUCT_PRESETS/FEATURES/TEMPLATES)，改 index 對應內容時要手動同步。in-index 編輯器(§4.13)仍並存。**工具列分群**(地圖庫/本圖/檢視/單張＋「？」`EDITOR_HELP` 說明)；**地圖左下角 1 公尺比例尺**(`PX_PER_M=40`/`drawScaleRef`)；**鏡像 ⇄/⇅**(`mirrorMap`/`mirrorToNew`：剛性翻轉成庫裡新的一張、保留原圖＝做地圖變體；敵朝向 H`π−θ`/V`−θ`)。**v3 多地圖庫(maps.json)**：editor 變成多地圖管理器＝以 `maps.json`(`{鍵:地圖}`)為單位「讀入整份→下拉切換/＋新增/🗑刪除/改鍵→下載整份」。model 層 `mapLib/curKey`＋`libImportObj/libExportObj/selectMap/addMap/deleteMap/renameCurKey/syncCurrentToLib`(DOM 分離、headless 可測)；通用清理抽成 `cleanOf`。**index.html 完全不動**——maps.json 只裝自訂地圖、不含內建圖，遊戲端怎麼吃 maps.json 之後再做。
- 地形：牆/門/低矮牆/草叢(MAP.bushes)/**可破壞地形(MAP.destructibles：wall/glass/lowwall/fence/flat 兩形態+HP)**；五型預設(木箱/落地窗/有矮牆窗/全遮蔽/鐵網)。改地形阻擋邏輯看 segBlocked/collideWalls/onLowWall/updateBullets/canSee。詳見 §4.20。
- 敵人＝資料驅動類型（`enemySpawns.type`，預設 grunt）：**grunt(步兵·衝鋒/rusher)・grunt_flank(步兵·包抄/flanker)**・dog(警犬)・heavy_lmg・heavy_shot；重裝靠裝甲「機率整發擋下非穿甲」(武器 `pierce`)而非高血量。詳見 §4.16。
- **敵人近戰收尾**（§4.6）：`enemyTick` 移動＝若 `seen && weapon.range ≤ CONFIG.enemy.meleeApproach(60)` 則直奔「目標本人」(f.pos) 而非節點，補上節點導航到不了咬擊距離的最後一段（只有 fangs=34 觸發，長射程武器永不）。性格(rusher/flanker/holder)目前只驅動**選點走位**，不改開火/距離管理（待重設計，§4.6 末）。
- **IDLE 巡邏**（§4.6）：`type.patrols` 的單位(只有 grunt)IDLE 會巡邏；重裝/狗守原地。`enemyIdleTick`→`patrolStep`：認領制 `patrolClaims`(隨機認領未佔點→`bfsPath` 走→`startDwell` 停留掃視→放掉再認領，點>兵=輪換不可預測)；巡邏池 `computePatrolPool`＝有 patrol 標籤點(≥2)用之否則自動取距入口/撤離口都遠的點。轉 ENGAGED/死亡 `releasePatrol`。`CONFIG.patrol{dwell,speedMul,autoBandFromEnds}`。
- **開關門（§4.6）**：`openDoorsNear(e,track)` 走到關門就開（**狗 opensDoors:false 不開**）；IDLE 巡邏開路上門＋`closeDoorsBehind`(走遠且門口無人才隨手關回)；**交戰後永不關門、只開**(進 ENGAGED 清 openedDoors)；玩家自走撤離 extractTick 也開門→門＝減速帶非死牆、**softlock 解除**(免額外 waypoint)。
- **被擊中來向局部脅威（§4.6，修側翼魚貫被側射全滅）**：`registerHitThreat`＝被擊中(連被甲擋下)時，在「被擊中者→射手」方位(`fromAng`)距 `hunt.threatDist` 放脅威點，傳給**本人＋看得到他的敵＋`hunt.threatShareRadius` 內近隣**(皆 ENGAGED)，存 per-enemy `threatPos/threatT`(只給方向、非全體 lastSeen＝保潛行)。enemyTick：無全體已知目標但有新鮮 threatPos(≤`hunt.threatGrace`)→當合成目標 `threatOnly`＝朝來向回頭/推進但**不對空地開火**；親眼看到真玩家才正常開火。持續開火持續刷新方位、停火移動可甩。`CONFIG.hunt{threatGrace,threatShareRadius,threatDist}`。
- **交戰後共享情報+搜索（§4.6，修視野穿牆）**：`updateThreatKnowledge()`(每幀，enemyTick 前)算我方「被任一敵看到否」＝共享。**KNOWN**(有人看到→全體共享實位追擊、更新 `player.lastSeenPos`；ENGAGED 用 `nearestKnownPlayer`，選點吃已知位置不穿牆、開火吃實位且需本隻 canSee)→**SEARCH**(全員失視線→`searchFocusNode`=最後已知位置最近節點、`searchSet`=bfsWithin hops、走到/視線掃過清格)→**PATROL**(清光沒找到→全場 patrolStep 含重裝、待新目擊再建焦點)。`CONFIG.hunt{searchHops3,knownGrace0.5}`。
- 側翼/突襲（§4.17，敵我皆適用）：繞背(目標視野錐外)＝傷害×2 且無視所有裝甲；閒置敵正面＝×1.5；`damageUnit` 第4參 `fromAng`。
- **手榴彈道具（§4.24b，不用新系統）**：`throwGrenade` 朝瞄準方向**丟出一顆會飛的手榴彈**(`grenades` 陣列，`rayHit` 算落點不穿牆、`throwRange/throwSpeed`)→`updateGrenades` 飛抵落點才 `detonateGrenade` 360° 散射一批 `bullets`(faction 玩家)。威力衰減＝被幾顆打中(中心密/遠處稀)、超 range 過期。pierce0 對重裝弱。值在 `ITEMS.grenade`(面板可調)。`drawGrenades` 畫飛行中的小手榴彈。
- **視覺化聲音（§4.25，無音效替代）**：`Sfx.play(id,pos)` 給 pos→`pingSound`(`SOUND_PINGS` 表)→`soundPings`；`drawSoundEdges` 對控制角色 `!canSee` 的 ping 在畫面外框該方向畫發光（你眼前/自己的聲音不提示，背後/牆後才亮）。`CONFIG.soundEdge`。引擎級、arcade 沿用。
- **開火現形（§4.24，修草叢隱形射手）**：敵人非消音開火 → `e.revealT=CONFIG.stealth.fireReveal`(0.6s)；`isRevealed`＝`revealT>0 || 任一我方 canSee`，`drawFog` 對 revealT>0 敵人挖洞→草叢/霧中正在開火的敵人對玩家現形（槍口火光暴露，與友軍 overwatch 一致）。
- 潛行/警覺層（§4.19，Stage 1-5 全做完）：IDLE 敵短錐+有限視距(`canSee` 依狀態)+識別空窗(`enemy.detect` 累滿才 ENGAGED，永鎖；槍聲/被擊中瞬交戰)+overwatch 對 IDLE 收火；草叢(`MAP.bushes`/`inConceal`，MOBA 遮蔽，`drawEnemies` 依 `isRevealed`)；消音手槍(`WEAPONS.silpistol` silent，亞音速彈稀少)；望遠(右鍵 `isScoping()`，鏡頭前帶+穩定+禁奔跑，技能移空白鍵)；前哨站 `MAPS.outpost`。`CONFIG.stealth`/`scope`。
- **每單位交戰姿態（§4.19）**：我方 `player.stance`（makePlayer 預設 `'defend'`）：`defend`＝overwatch 只打已 ENGAGED 敵(原行為)／`engage`＝打視野內任何敵(主動開戰)。`idleTick` 篩選＝`u.stance!=='engage' && e.state!=='ENGAGED' → skip`；`setStance(u,s)`。**鍵位變更：E＝engage、Q＝defend（對 controlledId 設、不奪回護送）；互動 E→F、道具 Q→C**。engage 單位頭頂紅「⚔」。
- **機密檔案（任務目標物，§4.26）**：run 級唯一物件，獨立攜帶格、休息關配裝卡單選指派攜帶者（`roster.carriesFile`/`player.hasFile`，`ensureFileCarrier` 恆一名、跨關隨撤出者）。攜帶者陣亡→`fileDrop` 掉地（穿迷霧永遠可見＋HUD 警示＋邊框箭頭），任一活動我方走過即背（含自走撤離，`updateFile`）。**失敗條件**：戰鬥結算無「撤出成功者帶著檔案」→ RUN FAILED（`loseReason`，與全滅同級）。第0關/沙盒/編輯試玩不啟用。改名：發射密碼→機密檔案。
- **開發測試架（`CONFIG.devLootRack`，預設 true、發佈前設 false）**：第0關地上陳列所有武器/道具(一般掉落)＝撿進背包→序章休息配裝；`enterRest` 背包→倉庫已支援 item/equip。`spawnDevRack`。
- 第0關/cutscene 導演（§4.18）：`initRun` 每局先 `startIntro()`（`intro` 地圖、scene COMBAT + `cutscene` 節拍導演，可 Esc 跳）→ 序章休息 → 路線圖。通用導演 say/move/face/wait/cam/sub/do/shoot/gate 可重用於未來劇情。改 update/draw/initRun 時注意 `cutscene`/`introActive` 分支。
- 場景：`MENU / MAP / COMBAT / REST / ARMORY / RUNEND / RUNWIN / EDITOR / SANDBOX`（有遊戲內關卡編輯器）。
- **主選單（進入點，§4.27）**：開機預設 `goMenu()`＝canvas 畫的 `MENU` 場景三選項（戰役→`initRun`／自由戰鬥→`enterSandbox`／地圖編輯器→`location.href='editor.html'`，`drawMenu`/`menuRects`/`menuClick`）。`?sandbox` 仍直接沙盒；`?playtest` 走 `bootPlaytest`。結算(RUNEND/RUNWIN)點擊/R＝回選單。
- **編輯器測試戰鬥（跨頁往返，§4.27）**：editor.html「▶ 測試戰鬥」→`playtestCurrent`(目前地圖存 `localStorage.sleeper_playtest`＋整庫存 `sleeper_editor_session`)→`index.html?playtest=1`→index `bootPlaytest` 讀圖、`enterSandbox` 預填地圖 JSON、`playtestMode=true`→打完/離開(`endMission`/`exitSandbox`/Backspace) `location.href='editor.html'`。回到 editor 由 `restoreSession` 還原整個工作區（不丟稿）。
- **自訂戰鬥 SANDBOX（§4.23）**：左上「⚔ 自訂戰鬥」鈕或 `?sandbox=1`。無路線圖/演出，自選小隊(個性/武器/技能/裝備＋🎲隨機)＋貼地圖JSON或內建下拉→打一場(敵人沿用地圖 enemySpawns)。重用 startMission/endMission，靠 `fromSandbox` 路由(打完/Backspace 回設定)。`initSandboxUI`(try/catch)；設定存 `sleeper_sandbox_v1`。
- **打擊感 juice（§4.24，`CONFIG.juice`）**：近 hitscan 子彈(`bulletSpeedMul`，碰撞本就掃描式不穿透)+曳光(`drawBullets` 'lighter')、頓幀(`hitstop`/`hitstopT`，主迴圈凍 update，僅擊殺)、螢幕震動(`shakeAmt`/`kickY`/`addShakeAt` 依距相機衰減，套在 `applyCam`+`w2s` 的 `shakeX/shakeY`)、開火後座(`u.recoil`)+槍口閃焰；**Batch 2**：受擊閃白(`u.hitFlash`)+後仰(`u.flinchX/Y`)+血霧(`spawnBlood`)+地上血漬(`decals`/`pushDecal`/`drawDecals` 上限160)。**我方死亡鏡頭已取消**(使用者：激戰中相機轉過去干擾操作)＝陣亡只保留短頓幀 `hitstop(deathHitstop)`，無相機帶位/慢動作/暈影。音效(WebAudio)暫不做。
- 完成的更新都 commit + push 到 GitHub `darkbearlab/sleeper_roguelite`（git 認證走系統 Git Credential Manager）。
- 改完一定要驗證：抽出 `<script>` 丟 Node `vm`、stub 掉 DOM/canvas/pointerlock/**localStorage**，**逐幀跑遍所有場景的 `update()+draw()`**（不要只在戰鬥跑）。
- 必踩雷：① `update()` 必須先判 `scene!=='COMBAT'` 再碰 `cam/players`（非戰鬥場景未建立）。② 迷霧用離屏 canvas 遮罩，**不可在主畫布 `destination-out`**（會擦掉世界）。③ `MAP` 已非單一 const 而是 registry 的作用中指標：衍生表(`N/adj/nodeVis/nextHop/EXTRACT/ENTRY`)由 `buildMapDerived(key)` 重建，別假設它們是載入期常數。
- I/O 接縫（§14，為將來可能搬 Phaser 預留）：模擬層讀輸入只走 `Input.*`(held/fire/locked/aim)、出聲只走 `Sfx.play(id)`；DOM 事件處理＝轉接器。**新增戰鬥邏輯時別直接讀 `keys`/`mouse`/`pointerLocked`/`aimDir`，改用 `Input`**。render 門面/迷霧/相機等昂貴接縫尚未抽（仍直接用 `ctx`）。
- 細節別寫進本檔（每 session 自動載入，保持精簡）；重內容放 `開發進度與脈絡.md`。
