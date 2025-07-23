# rabbitMQ 原理

## 簡介

RabbitMQ 是一個開源的訊息佇列中間件，基於高階訊息佇列協定（AMQP）建構。

## 交换器:

+ 相比於較為簡單的主題直連，rabbitiMQ 有額外的 router 功能。

+ Exchange 根據類型 + routing key，選擇性地把訊息發送到一個或多個佇列（Queue）

+ 類型
    + direct 直連交換機	: 根據 routing key 完全匹配佇列綁定	
    + fanout 廣播交換機	: 無視 routing key，發送到所有綁定的佇列
    + topic	主題交換機 : 支援 routing key 模糊匹配（用 *、#）	
    + headers 標頭交換機 : 根據 message 的 headers 做條件篩選

## 支援優先級佇列

+ RabbitMQ 是三者中唯一原生支援優先級佇列的消息中介。 宣告佇列時加上 x-max-priority 屬性（最大優先級）。發送消息時，指定 priority 屬性（0~最大值）。

## 支持死信隊列

+ API 如果消費某消息失敗，可以考慮將其放入此 topic 對應的死信隊列後進行後續處理

## 支持延遲消息

+  支持消息可以過去指定時間後再取用。

## 只支持內部 transection

適用於 MQ 自己的 publish/consume 錯誤處理，不保證與 DB 一致性。可以考慮以補償邏輯/重試機制為主