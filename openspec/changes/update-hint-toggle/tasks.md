## 1. Implementation
- [x] 1.1 Add UI toggle control for提示/鍵位說明（顯示/隱藏），含手機易點擊樣式。
- [x] 1.2 Persist toggle state（localStorage）並設定合理預設：手機預設隱藏可節省空間。
- [x] 1.3 Adjust layout when提示區隱藏，確保鋼琴/遊戲區在小螢幕放大、不中斷其他面板。
- [x] 1.4 Update accessibility/aria labels與文案，引導使用者如何展開/收起。

## 2. Validation
- [ ] 2.1 Browser manual test：手機尺寸/桌機尺寸切換，隱藏/顯示提示；鋼琴可點擊/按鍵正常；狀態記憶、重新整理後仍生效。
- [ ] 2.2 Regression check：倒數、判定、任務提示、排行榜/設定面板不受影響。
