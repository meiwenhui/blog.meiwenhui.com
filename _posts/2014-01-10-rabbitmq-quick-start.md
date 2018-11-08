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

### 相关方法，相关参数
```
/**
 * 声明一个队列, 如果不存在就创建它
 *
 * @param string $queue
 * @param bool $passive
 * @param bool $durable
 * @param bool $exclusive
 * @param bool $auto_delete
 * @param bool $nowait
 * @param array $arguments
 * @param int $ticket
 * @return mixed|null
 */
public function queue_declare(
    $queue = '',
    $passive = false,
    $durable = false,
    $exclusive = false,
    $auto_delete = true,
    $nowait = false,
    $arguments = array(),
    $ticket = null
)


/**
 * Specifies QoS
 *
 * @param int $prefetch_size
 * @param int $prefetch_count
 * @param bool $a_global
 * @return mixed
 */
public function basic_qos(
    $prefetch_size,
    $prefetch_count,
    $a_global
)

/**
 * 声明一个交换机
 *
 * @param string $exchange
 * @param string $type
 * @param bool $passive
 * @param bool $durable
 * @param bool $auto_delete
 * @param bool $internal
 * @param bool $nowait
 * @param array $arguments
 * @param int $ticket
 * @return mixed|null
 */
public function exchange_declare(
    $exchange,
    $type,
    $passive = false,
    $durable = false,
    $auto_delete = true,
    $internal = false,
    $nowait = false,
    $arguments = array(),
    $ticket = null
)

/**
 * Publishes a message
 *
 * @param AMQPMessage $msg
 * @param string $exchange
 * @param string $routing_key
 * @param bool $mandatory
 * @param bool $immediate
 * @param int $ticket
 */
public function basic_publish(
    $msg,
    $exchange = '',
    $routing_key = '',
    $mandatory = false,
    $immediate = false,
    $ticket = null
)

/**
 * Starts a queue consumer
 *
 * @param string $queue
 * @param string $consumer_tag
 * @param bool $no_local
 * @param bool $no_ack
 * @param bool $exclusive
 * @param bool $nowait
 * @param callable|null $callback
 * @param int|null $ticket
 * @param array $arguments
 * @return mixed|string
 */
public function basic_consume(
    $queue = '',
    $consumer_tag = '',
    $no_local = false,
    $no_ack = false,
    $exclusive = false,
    $nowait = false,
    $callback = null,
    $ticket = null,
    $arguments = array()
)
```



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
direct的例子都基本都是1对1的消息发送和接收，即消息只能发送到指定的queue里，但有些时候你想让你的消息被所有的Queue收到，类似广播的效果，这时候就要用到exchange了。 fanout广播模式是实时的，你不在的时候(消费者没有开启)，发消息的时候，就没有收到，这个时候就没有了。如果消费者开启了，生产者发消息时，消费者是收的到的，这个又叫订阅发布，收音机模式。

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
①服务端没有声明queue，为什么客户端要声明一个queue？

答：生产者发消息到exchange上，exchange就会遍历一遍，所有绑定它的哪些queue，然后把消息发到queue里面，它发了queue就不管了，消费者从queue里面去收，所以就收到广播了，而不是说exchange直接就把消息发给消费者，消费者只会从queue里去读消息，且拿着queue去绑定exchange。

②为什么queue要自动生成，而不是自己手动去写？

答：我这个queue只是为了收广播的，所以如果我消费者不收了，这个queue就不需要了，所以就让它自动生成了，不需要的了，就自动销毁


### direct 广播模式(可过虑)
fanout广播模式是1对多以广播的方式，发送给所有的消费者。那如果消费者想对消息过滤，进行有选择的进行接收我想要的消息，那怎么办呢？这就有了第二种广播方式，即direct广播模式。RabbitMQ还支持根据关键字发送(routing_key)，即：队列绑定关键字，发送者将数据根据关键字发送到消息exchange，exchange根据 关键字 判定应该将数据发送至指定队列。

![direct广播模式](https://images2017.cnblogs.com/blog/1147891/201711/1147891-20171122154049493-2024235822.png)


生产者
```
$rabbitmq = new AMQPStreamConnection('127.0.0.1', 5672, 'test', 'test', '/test');
$channel = $rabbitmq->channel();
$channel->exchange_declare('direct_exchange_logs', 'direct', false, false, false);

$severity = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'info';

$data = implode(' ', array_slice($argv, 2));
if (empty($data)) {
    $data = "Hello World!";
}

$msg = new AMQPMessage($data);

$channel->basic_publish($msg, 'direct_exchange_logs', $severity);

echo ' [x] Sent ', $severity, ':', $data, "\n";

$channel->close();
$connection->close();
```
消费者
```
$channel = $rabbitmq->channel();
$channel->exchange_declare('direct_exchange_logs', 'direct', false, false, false);

list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$severities = array_slice($argv, 1);
if (empty($severities)) {
    file_put_contents('php://stderr', "Usage: $argv[0] [info] [warning] [error]\n");
    exit(1);
}

foreach ($severities as $severity) {
    $channel->queue_bind($queue_name, 'direct_exchange_logs', $severity);
}

echo " [*] Waiting for logs. To exit press CTRL+C\n";

$callback = function ($msg) {
    echo ' [x] ', $msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while (count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();
```
#### Cheers!
