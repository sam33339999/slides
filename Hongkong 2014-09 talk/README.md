## 遊戲伺服器的挑戰
- 請求處理時間短
- 大量資料的改變
- 數據壽命低
- 自然的高併發競爭

## 幸運的是
- 遊戲資料通常是容錯的
    - fault-tolerance 容錯的
        - 用戶通常會跟開發商對著罵，繼續玩
- 通常情況下，遊戲數據不大

## Redis 的關鍵概念
- 一個 nosql db
- 記憶體，且持久
- 鍵值對，並且有不同的數據結構 various 不同的
- 允許 Time-to live(TTL)
- 允許阻塞I/O

## Redis 能為我們做到什麼事？
- 儲存遊戲大廳紀錄
- 會話資料庫
- 高性能的鎖定伺服器
- 簡單但足夠用

## 不利條件 (Disadvantages)
- 開發任務的巨大增長 (Huge increase in development mandays) 
    - huge 巨大的
    - increase 增加
    - mandays 進展
        - 對於第一個項目而言，在伺服器端會有 ~50% 的額外任務(cache, ...etc...)
- 在服務器端需要有個額外的組件，在監控、故障轉移方面有待加強
    - effort 盡力
    - monitoring 監控
    - failover 故障轉移
- 需要明確的鎖定
    - explicit 直率的
- 必須為崩潰和恢復後的狀態不一致做好準備
    - inconsistent 不一致

## 但是，我們仍然在使用 redis
- 內存很便宜，而且*之後會更便宜*
- 對於高流量的遊戲服務器，Redis 仍然比一組 RDBMS 服務器簡單
    - traffic 交通
- 在虛擬機的實體中，磁碟I/O 通常是共享的，但內存帶寬卻不是
    - bandwidth 帶寬