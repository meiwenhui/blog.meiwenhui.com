---
layout: post
title: "RabbitMQ 快速开始"
comments: true
description: "How to Download or Use This Theme"
keywords: "RabbitMQ"
---

### Use this theme as you main site

- direct  Routing Key==Binding Key
- fanout	把所有发送到该Exchange的消息路由到所有与它绑定的Queue中
- topic   其实就是模糊匹配
- headers Exchange不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。

### direct

生产者
```
$rabbitmq = new AMQPStreamConnection('127.0.0.1', 5672, 'test', 'test', '/test');
$channel = $rabbitmq->channel();
$channel->queue_declare(self::QUEUE_NAME, false, true, false, false);
$msg = new AMQPMessage(
    "数据",
    array('delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT)
);
$channel->basic_publish($msg, '', self::QUEUE_NAME);
$channel->close();
```
消费者
```
$channel = $rabbitmq->channel();
$channel->queue_declare(self::QUEUE_NAME, false, true, false, false);
$callback = function ($msg) {
    $data = $msg->body;
    $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
};
$channel->basic_qos(null, 5, null);
$channel->basic_consume(self::QUEUE_NAME, '', false, false, false, false, $callback);
while (count($channel->callbacks)) {
    $channel->wait();
}
$channel->close();
```
#### Cheers!
