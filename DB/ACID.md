
# Transection 與 ACID

## ACID

操作資料庫裏面有一個很重要的東西叫做 transaction。Transection必須滿足以下條件:

    * Atomicity 原子性
    transaction 裡面的操作必須全部成功或全部失敗。

    * Consistency 一致性
    不同的數據都會有一些基本的約束，而這些約束在交易前跟交易後都必須要遵守，如果沒辦法遵守交易就必須失敗，聽起來很抽象，舉剛剛的匯款的舉例，匯款有一些基本的限制：
    雙方的錢都不能小於 0
    雙方錢的總和不能改變
    上面兩個限制在交易前跟交易後都必須要遵守，這就是一致性。

    * Isolation 隔離性
    多個 transaction 不會互相干擾，不能同時修改到同一個值

    * Durability 持久性
    在 transaction 成功之後。就算 server 當機、斷電，已經修改的數據也不會不見，應該要被寫入能夠永久儲存的裝置中，而不是暫存。

## Transection

Transection 使得一系列DB操作具有 ACID 特徵，目前 mysql 與 mongodb 皆支援 transection