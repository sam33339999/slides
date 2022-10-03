# 內部分享 - 1
- 談 trigger
- 談 view

## trigger
- 集中談 `post action`, `for each row` 類型的 trigger
    - 在 MySQL 中，可能看到 
```sql
CREATE TRIGGER XXX ON TABLE ...
AFTER {INSERT/UPDATE/DELETE}
FOR EACH ROW ...
```
- 主要用途
    - Audit log 紀錄log
        - 對於正常的 tableX，我們想要建立 tableX_log ，去紀錄一切改動
        - tableX_log 只能由 audit_log_user 來修改，一般用戶沒有權限；所以 cracker 沒辦法將犯罪記錄刪掉
        - 這個 trigger 是由 tableX_log 的權限是由 audit_log_user 的權限用戶建立的，
        所以觸發時，是由 audit_log_user 的權限來寫入 tableX_log
    - Database refactoring 資料庫重構
        - 商業邏輯修改 db schema 時，必須維持 legacy code 不會掛掉
        - 除非你部署時有停機時間，否則系統一定時間是新舊版本程式並存
            - 你不可能一秒鐘部署完 100 個 docker container

    - Database refactoring 例子：
        - 貓與兔子都在 pets 這個資料表內
        - 新的改動是貓和兔子都要有自己專門的屬性，所以分成 cats, rabbits
        - 為了支援 legacy code ， pets 這個表是暫時需要保留的
        - 所有 cats, rabbits 的改動，都要用 trigger 去重新 pets 的數據，反之亦然。
            - 小心 環狀依賴 問題
- 缺點
    - function programming ，一但系統變複雜了便難以維護。
    - 商業邏輯寫在 應用層，有什麼錯還能直接連進去資料庫直接跑 SQL 改正
    - 一但商業邏輯寫在 RDBMS 內，有問題時可能沒辦法即時做資料修正。
    - 除非要做到上述的兩種情境，否則絕對不要用 trigger
## view

- 除非要做到 database refactoring ，否則絕對別用 updatable view
- 想像成 view 是一個 placeholder ，用一個名字來替代本來的 subquery
- 理論上：用了 view 和原本的 query 對比，其執行計畫應該是相同的，效能應該也是 100% 相同

- 上古時代的用途：
    - 資料庫需要讓第三方直接連入拿資料，但有些敏感欄位不能讓他們資到
    - 寫報表時其 SQL 又臭又長，用 view 把 SQL 拆開來變得容易閱讀


