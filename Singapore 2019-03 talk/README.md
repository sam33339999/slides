# 使用Redis高流量限速

- 面臨到的問題
    - 非預期的事件導致流量異常爆炸
        - Unpredictable 非預期的
        - sudden 意外的
        - burst 爆炸
    - 自動擴展不夠及時
        - Auto-scaling 自動擴展
    - 資料庫的擴展是痛苦的
        - 特別是在高峰期
            - Especially 特別的
            - peak hours 高峰期
        - 特別是使用 m***odb
    - 解決: HTTP 429, Too Many Requests
- 解法一
- 解法二