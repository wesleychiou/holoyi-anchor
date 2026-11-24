# holoyi-anchor

HoloYi 台股預測研究專案的**前瞻預測帳本錨點**。本儲存庫只有一個檔案 `anchors.jsonl`，每行記錄某一時刻帳本兩條雜湊鏈的鏈尾與筆數：

```json
{"anchoredAt":"2026-11-24T15:30:00.000+08:00","outcomes":{"chain":"…","count":0},"predictions":{"chain":"…","count":11}}
```

**不包含**任何預測內容、標的、行情或報酬。

## 這是做什麼的

專案每個交易日盤後把「下一期」的預測寫進只增不改的帳本，目標期結束後再對帳（協議：前瞻對帳協議，預先登記）。帳本用雜湊鏈串接：每筆的 `chain = SHA-256(前一筆 chain ‖ 該筆內容雜湊)`。

鏈尾雜湊會「承諾」整本帳本的全部歷史：事後改動、刪除或插入任何一筆預測，之後的鏈尾都會不同。把鏈尾公開推送到這裡，推送時間由 GitHub 伺服器端記錄，就能讓第三方檢查：**某筆預測在某個時間點以前就已經存在**，且之後沒有被改寫。

## 怎麼驗證

1. 取得帳本的 `predictions.jsonl`（作者公開時）。每行 JSON 含 `seq`、`chain`。
2. 對 `anchors.jsonl` 的每一筆錨點，帳本第 `predictions.count` 筆的 `chain` 必須等於該錨點的 `predictions.chain`（`outcomes` 同理）。
3. 重算每筆的內容雜湊與鏈，確認帳本內部一致（專案的 `npm run verify` 已做這件事）。
4. 查 GitHub 上對應 commit 的**推送時間**（不是 commit 自述的時間，後者可由作者本機任意設定）。

## OpenTimestamps（獨立的長期證明）

`ots/anchors-NNNNN.jsonl.ots` 是 [OpenTimestamps](https://opentimestamps.org) 證明：`anchors.jsonl` **前 NNNNN 行**的 SHA-256 經公開日曆伺服器匯集後寫入比特幣區塊。這一層不依賴 GitHub 或任何公司，也不受 GitHub 事件保留期限影響。

- 剛蓋章的證明是「等待中」（PendingAttestation），通常數小時內由排程升級為「已入區塊」（BitcoinBlockHeaderAttestation，附區塊高度）。
- 驗證：`head -n NNNNN anchors.jsonl > anchors-NNNNN.jsonl`，再用 `ots verify anchors-NNNNN.jsonl.ots`（需要比特幣節點），或把兩個檔案拖進 [opentimestamps.org](https://opentimestamps.org) 的網頁驗證。
- 較新的證明涵蓋之前所有行（檔案只增不改），所以排程每個台北日期最多蓋一次章。

## 限制（誠實聲明）

- 錨點只證明「這個鏈尾在推送時已存在」。它不證明預測有預測力，也不證明帳本內容是對的。
- GitHub 推送時間信任 GitHub，且事件 API 只保留較近期的紀錄；長期證明靠上面的 OpenTimestamps（比特幣）。
- 若某天漏推，之後補推的錨點只能證明「補推時」之前就存在。
