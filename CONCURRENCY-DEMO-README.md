# Concurrency 範例

舉例案例：
    被 DDOS 了，想要找出來源 ip
        - 100 個 apache access file，每個檔案約 5G
        - 找出每小時 top 10 ip 來源

Notice:
    只是範例，DDOS 不是這樣擋的
    以 ip 來擋，很有可能錯殺用 NAT 的 ISP 正常用戶

    要分析像 apache 這種熱門的 log，市面上有大堆軟體像是 goaccess 之類的。

分案：
    寫一個小的 script 把 access log 掃描一次；
    然後把每小時的 access log 丟入獨立的 file
    每小時的 access log 都跑一個 goaccess 的 process 進行分析。
    這樣做的話，你多少個 CPU 都能吃光光... 100% 真正的「平衡運算」 ...(這會有問題)


Notice!
    不准無腦的開 1000 個 process ，要寫的優美。
    最終版本： 不准讓系統資源閒置浪費掉
        - 就是說： 要卡在系統的 `io bandwidth` 或是 `CPU 總效能`

## v1 版本：
    - 將每小時的 access log 丟入 Redis -> HashSet
    - Warning!!!: 這會會掉大量的 Redis 記憶體
    將全部的 access log 都跑完後，你對每小時 HashSet 做 HGetAll 取回所有 ip ，再跑一次 sort 自然就可以拿到每小時最高的 ip 

    logAnalyer v1 確實是 single thread... 但是你能用 tmux 在 n 個 CPU 跑 >= 2n 個 logAnalyer ... (這是 ... multi process 吧？... 但是... 恩...)

    Redis 的操作是 blocking 的，即是說 HINCRBY 在 Redis 跑完前， application 的 thread 必須等待(blocked)
        - 等待中的 thread 是不吃 CPU 的
    分析了一下，在剛剛的 loop 中，最長的時間應該是 Redis 操作
        - Network 是有 latency 的
            - latency 潛伏的
        - 如果在 n CPU 只跑 n 個 logAnalyer，肯定吃不完 CPU 的
    
    - 缺點：
        - 每一筆資料都是 Redis 的操作，也就是浪費了很多 network latency
        - 每一次的 Redis 操作都會觸發 context switching

## v2 版本：
    - RedisBulkAction
    有個 buffer ，並且將資料從 Redis 取回的邏輯改成 Redis 回傳多筆，這時候可能會用上 `Redis pipelining` 或是 `lua script` 了。

    偽代碼：
```
class RedisBulkAction {
    protected IpBuf[2000] buffer;

    public addLog(ip, accessAt)
    {
        this.addToBuf(ip, accessAt);

        if (this.checkBufFull()) {
            bulk Redis.HINCRBY; ...
            // 別忘記做完要清除 buf 資料
        }
    }
}
```

    現在可能每 2000 筆資料才會做一次 Redis bulk action
        - 所以 2000 次才會有一次 network latency
        - 所以 2000 次才會有一次 context switching

    理想做法： 把 blocking io 的工作以獨立的 thread 群去做工

## v3 版本：
    single process, multi threading

    以此案例，會有兩種 thread 做處理：
        - importer: 將 資料邏輯 做 parsing
        - exporter: 將 處理後的資料 以 bulk action 手段寫入 Redis
    importer 跟 exporter 之間資料流動以 channel 串起來。
    
    在 n 個 CPU 環境下，大約是 n 個 importer + 2, 3 個 exporter


## 淺談 stream
    - 未談 channel 前，先來談 Linux 的 stream
    > 所謂的 stream 就是 FIFO 的將資料從一個 process 流到另外一個 process

    process 的 stdin 就是一個 stream
        - data: 用戶的輸入 (keyboard input)
            - sender: terminal 就像是 bash 這個 process
            - receive: process 自己
        - blocking 會出現在:
            - stream 的 buffer 滿了，sender 要丟資料進 stream (full !)
            - stream 是空的， receive 想要從 stream 拿資料 (waiting !)
    