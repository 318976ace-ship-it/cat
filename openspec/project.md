# Project Context

## Purpose
- 瀏覽器內的 12 鍵鋼琴下落式節奏小遊戲「Mew Atelier」，仿真鋼琴黑白鍵排列（A/S/D/F/J/K/L 白鍵、W/E/T/Y/U 黑鍵），附 30 首 30 秒 BGM 與自由彈奏模式。
- 除常規歌單模式外，提供任務養成、排行榜與多主題皮膚，支援鍵盤與滑鼠/觸控指彈。

## Tech Stack
- 靜態 HTML + CSS + Vanilla JS（無編譯流程、無框架）。
- Web Audio API（AudioContext + Oscillator）產生鍵聲，`<audio>` 播放 BGM；localStorage 保存高分、排行榜與任務進度。
- Google Fonts `Noto Sans TC`；音檔位於 `assets/bgm/*.mp3`（每首 30s）。

## Project Conventions

### Code Style
- 單一 `index.html` 內嵌 CSS/JS；採 `const/let`、箭頭函式、template literal、camelCase 命名，DOM 以 `querySelector` + `addEventListener` 操作。
- 盡量維持零依賴與直開即玩；新增主題時沿用 `:root` 與 `body[data-theme=…]` 的 CSS 變數模式。
- 不使用行內事件綁定；保持程式邏輯集中於腳本區塊並以小函式拆分（例如判定、任務、排行榜）。

### Architecture Patterns
- 單頁遊戲循環：全域 `game` 狀態（模式/曲目/節拍/判定/分數）＋ `CONFIG`/`SCOREMAP` 常數控制落點與判定視窗。
- 曲目表 `SONGS` 預先定義 30 首 30 秒 BGM（`id/name/bpm/seq/baseNote/diff/tip`），啟動時填入 `#songSel`；`makeSong` + `withRests` 生成節拍序列，`baseNote` 控制音高。
- 模式：`songs`（有 BGM 與下落判定）與 `free`（僅合成器鍵聲，無判定）；自由模式仍追蹤按鍵數與時長以完成相關任務。
- 判定：`draw` 每幀重建 tile，`judge` 依 ms 視窗（perfect 55 / great 95 / good 140）與 lane 比對給分，`autoMiss` 超時自動 Miss，`autoOffset` 以近期樣本平均修正手感。
- 永續化：`pianotiles12_realkeys_v2`（單曲高分/準度/日期）、`mew_leaderboard_v1`（全局排名取前 30）、`mew_quests_v1`（任務完成），另有 `mew_chain_acc75/90`、`mew_multi_8000/10000` 等輔助 key 追蹤連續或多曲條件。
- 任務系統 `QUEST_DEFS` 共 35 項、章節 `CHAPTERS` 5 個；完成後 toast 提示、進度條與徽章點亮。排行榜介面僅顯示前 5 名，實際儲存前 30。
- UI 流程：啟動遮罩（鍵位與玩法提示）→ 倒數 → 遊玩 → 結果（含換曲/再玩/加速）＋高分更新；另有設定面板（主題/速度/音量/鍵聲音色）與判定說明。

### Testing Strategy
- 無自動化測試；以瀏覽器實玩驗證。更動判定、曲目或音檔時需手動覆蓋多首歌、不同速度（含 1.2x/1.4x/1.5x）、高 Combo、0 Miss 等場景。
- 重點檢查：鍵位/黑白鍵排布是否對應、BGM 與鍵聲音量控制、速度拉條對 BGM 與節拍的同步、判定視窗與 Combo 重置、auto miss、任務完成與章節進度、排行榜排序與存取、自由彈奏任務累計、主題切換與響應式排版。

### Git Workflow
- 專案未設 CI/分支策略；建議以 feature branch 開發，小變更可直接 PR 到主線。
- 新功能、行為變更或大型調整需先依 `openspec/AGENTS.md` 建立 change proposal，再進入實作。避免修改/移除現有 localStorage key 以免玩家紀錄遺失。

## Domain Context
- 核心玩法：12 鍵下落式判定，白鍵 A/S/D/F/J/K/L、黑鍵 W/E/T/Y/U，排列對應真實鋼琴。支援鍵盤與點擊方塊/琴鍵操作。
- 設定：主題皮膚、速度 0.8~1.6x、BGM 音量、鍵聲音量與音色（soft/bright/bell/chiptune/saw）；開局可選歌單或自由模式，歌單模式有 3→2→1 倒數。
- BGM 長度假設 30 秒，`CONFIG.LEAD=4` 只渲染當前拍與前 4 拍；Perfect/Great/Good 分別給 300/200/100 分並附帶 Combo 係數（combo *0.6），Miss 歸零 Combo。鼓勵語隨機顯示於結果。
- 結果面板提供高分更新、換曲/再玩/加速 0.2x 快捷；排行榜展示前 5 名、依 score→acc 排序。任務涵蓋首通、分數/準度/Combo/速度/特定曲目、連續表現與自由彈奏時長/按鍵數。

## Important Constraints
- 音訊需使用者互動才能啟動；保留 `ensureAudio`/suspended-resume 流程，避免瀏覽器自動播放阻擋。
- 音檔路徑固定 `assets/bgm/`，檔名需與 `SONGS` 定義一致；長度假設 30s，如更動需同步調整 `beats/seq` 與任務條件。
- localStorage 結構需向後相容，避免改 key 導致既有高分、排行榜與任務進度消失。
- `draw` 每幀重建 tile DOM，避免在迴圈內加入重型計算或多餘節點；保持 `CONFIG.LEAD` 控制節點數以防卡頓。
- 專案為零建置、純前端，確保直接開啟 `index.html`（或使用簡單靜態伺服器）即可運作，避免新增需要打包或網路權限的依賴。

## External Dependencies
- Google Fonts：`https://fonts.googleapis.com/css2?family=Noto+Sans+TC...`
- 瀏覽器內建 API：AudioContext/Audio、localStorage、requestAnimationFrame、Pointer/Keyboard 事件。
- 靜態音檔：`assets/bgm/*.mp3`（30 首 30s BGM）。可搭配 `live-server` 等輕量本地伺服器做預覽，但非必要。
