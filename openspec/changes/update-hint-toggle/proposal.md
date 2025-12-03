# Change: Add hint visibility toggle for mobile play area

## Why
- Mobile players want a larger piano area; current on-screen 提示/關鍵字 佔用空間且干擾觸碰。
- Allowing hints to be collapsed/expanded improves可用觸控區與體驗，同時保留需要時再查看的能力。

## What Changes
- Add 一組控制可見性的 UI 按鈕（顯示/隱藏提示與鍵位說明），可在手機版快速切換。
- 將提示區塊預設為可收起，並記住切換狀態以便回到既有佈局。
- 調整小尺寸版面佈局：隱藏後鋼琴/遊戲區域能獲得更大可觸控空間。

## Impact
- Affected specs: ui
- Affected code: `index.html`（UI 佈局、事件邏輯、狀態儲存）
