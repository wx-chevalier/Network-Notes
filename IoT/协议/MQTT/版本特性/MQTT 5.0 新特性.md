# MQTT 5.0 新特性

目前支持和使用最广泛的版本是 MQTT 3.1.1 版本。2017 年 8 月，OASIS MQTT Technical Committee 正式发布了用于 Public Review 的 MQTT 5.0 的草案。2018 年，MQTT 5.0 已正式发布，但是目前支持 MQTT 5.0 的 Broker 和 Client 库还比较有限。

MQTT 5.0 在 MQTT 3.1.1 的基础上做了很多改变，并不是向下兼容的。在协议上，就增加了 Property 字段，也正是因为这个字段，使得 MQTT 5.0 可以支持众多的新特性。下面将从以下几个新特性进行介绍，这些新特性能够解决在 3.1.1 版本中较难处理的问题。

## 用户属性（User Properties）

5.0 中可以在 PUBLISH、CONNECT 和带有 Return Code 的数据包中夹带一个和多个用户属性数据：

- 在 PUBLISH 包中携带的用户属性由发送方的应用定义，随消息被 Broker 转发到消息的订阅方；
- CONNECT 和 ACKs 消息里面也可以带发送者自定义的用户属性数据。

在实际的项目中，除了关心收到的消息内容，往往也想知道这个消息来自于谁。例如：ClientA 收到 ClientB 发布的消息后，ClientA 想给 ClientB 发送一个回复，这时 ClientA 必须知道 ClientB 订阅的主题才能将消息传递给 ClientB。在 MQTT 3.1.1 中，我们通常是在消息数据中包含发布方的信息，比如它订阅的主题等。5.0 以后就可以把这些信息放在 User Properties 里面了。

## 共享订阅（Shared Subscriptions）

在 3.1.1 和之前的版本，订阅同一主题的订阅者都会同样收到来自这个主题的所有消息。例如你需要处理一个传感器的数据，假设这个传感器上传的数据量非常大且频率很高，你没有办法通过启动多个 Client 来分担处理的工作，因为多个 Client 都会同样收到所有消息。

针对上述的问题，也许可以启动一个 Client 来接收传感器的数据，并将这些数据分配给后面多个 Worker 来处理。这个用于接收数据的 Client 就是系统的瓶颈和单点故障之一。也可以通过主题分片，如让传感器依次发布到 /topic1…/topicN 来变通地解决这个问题，但仅仅是解决了部分问题，同时也提高了系统的复杂度。

而在 5.0 里面，MQTT 可以实现 Producer/Consumer 模式了。多个 Client（Consumer）可以一起订阅一个共享主题（Producer），来自这个主题的消息会依次均衡地发布给这些 Client，实现订阅者的负载均衡，最终解决了这个问题。

> 这个功能在传统的队列系统中，比如 RabbitMQ 里面很常见。EMQTT 在 3.1.1 上已经支持这个功能了，详情可以查询[官方文档](https://link.zhihu.com/?target=http%3A//emqtt.com/docs/v2/advanced.html%23shared-subscription)

## 消息过期（Publication Expiry Interval）

假设一个基于 MQTT 的共享单车平台，用户通过平台下发一条开锁指令给一辆单车，但是不巧的是，单车的网络信号（比如 GSM）这时恰好断了，用户摇了摇头走开去找其他车了。过了两个小时以后，单车的网络恢复了，它收到了两个小时以前的开锁指令，该怎么做？

为了处理这种情况，在 3.1.1 和之前的版本，我们往往都是在消息数据里带一个消息过期时间，在接收端来判断消息是否过期。但是这要求设备端的时间和 Server 端保持一致，对于一些电量不是很充足的设备，一但断电，之后再启动时间就会变得不准确，会导致异常的出现。

5.0 中终于包含了消息过期的功能，在发布的时候可以指定这个消息在多久之后过期，Broker 不会将已过期的离线消息发送到 Client。

## 重复主题

在 MQTT 3.1.1 和之前的版本里，PUBLISH 数据包每次都需要带上发布的主题名，即使你每次发布的都是同一个主题。

在 5.0 里面，如果你将一条 PUBLISH 的主题名设为长度为 0 的字符串，那么 Broker 会使用你上一次发布的主题。这样降低了多次发布到同一主题（往往都是这样）的额外开销，对网络资源和处理资源都有限的系统非常有用。

## 5. Broker 能力查询

在 5.0 里面，CONNACK 包含了一些预定义的头部数据，用于标识 Broker 支持哪些 MQTT 协议功能。Client 在连接之后就可以知道 Broker 是否支持自己所要用到的功能，这对于一些通用的 MQTT 设备生产商或者 Client 库的开发者很有用。

Pre-defined Header 数据类型描述 Retain AvailableBoolean 是否支持 Retained 消息 Maximum QoSNumberClient 可以用于订阅和发布的最大 QoSWildcard availableBoolean 订阅时是否可以使用通配符主题 Subscription identifiers availableBoolean 是否支持 Subscription Identifier（5.0 特性）Shared Subscriptions availableBoolean 是否支持共享订阅 Maximum Message SizeNumber 可发送的最大消息长度 Server Keep AliveNumberBroker 支持的最大 Keep Alive 值

## 双向 DISCONNECT

在 3.1.1 和之前的版本里，只有 Client 在主动断开时会向 Broker 发送 DISCONNECT 包。如果因为某种错误 Broker 要断开和 Client 的连接，它只能直接断开底层 TCP 连接，Client 则不会知道自己连接断开的原因，也无法解决错误，只是简单地重新连接、被断开、重新连接...

在 5.0 里面，Broker 在主动断开和 Client 的连接时也会发送 DISCONNECT 包。同时，从 Client 到 Broker，以及从 Broker 到 Client 的 DISCONNECT 包里面都会包含一个 Reason Code，来标识断开的原因。

Reason code 发送方描述 0Client 或 Broker 正常断开连接，不发送遗愿消息 4Client 正常断开，但是要求 Broker 发送遗愿消息 129Client 或 BrokerMQTT 数据包格式错误 135Broker 请求未授权 143Broker 主题过滤器格式正确，但是 Broker 不接收 144Client 或 Broker 主题名格式正确，但是 Client 或者 Broker 不接收 153Client 或者 Broker 消息体格式不正确 154Broker 不支持 Retained 消息 155BrokerQoS 等级不支持 158Broker 不支持共享订阅 162Broker 订阅时不支持通配符主题名
