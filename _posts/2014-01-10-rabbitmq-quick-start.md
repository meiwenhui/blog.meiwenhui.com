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

### fanout 广播模式
direct的例子都基本都是1对1的消息发送和接收，即消息只能发送到指定的queue里，但有些时候你想让你的消息被所有的Queue收到，类似广播的效果，这时候就要用到exchange了，先来说说exchange的官方说明：

生产者
```
$rabbitmq = new AMQPStreamConnection('127.0.0.1', 5672, 'test', 'test', '/test');
$channel = $rabbitmq->channel();
$channel->exchange_declare('logs_exchange', 'fanout', false, false, false);

$msg = new AMQPMessage(
    "info: Hello World!",
    array('delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT)
);

$channel->basic_publish($msg, 'logs_exchange');

$channel->close();
$connection->close();
```
消费者
```
$channel = $rabbitmq->channel();
$channel->exchange_declare('logs_exchange', 'fanout', false, false, false);

list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$channel->queue_bind($queue_name, 'logs_exchange');

echo " [*] Waiting for logs. To exit press CTRL+C\n";

$callback = function ($msg) {
    echo ' [x] ', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while (count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

#### Cheers!
