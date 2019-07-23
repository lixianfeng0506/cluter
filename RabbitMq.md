# RabbitMq介绍
发布者（publisher）发布消息时可以给消息指定各种消息属性（message meta-data）。有些属性有可能会被消息代理（brokers）使用，然而其他的属性则是完全不透明的，它们只能被接收消息的应用所使用
消息确认（message acknowledgements）的概念：当一个消息从队列中投递给消费者后（consumer），消费者会通知一下消息代理（broker），这个可以是自动的也可以由处理消息的应用的开发者执行。当“消息确认”被启用的时候，消息代理不会完全将消息从队列中删除，直到它收到来自消费者的确认回执（acknowledgement）
在某些情况下，例如当一个消息无法被成功路由时，消息或许会被返回给发布者并被丢弃。或者，如果消息代理执行了延期操作，消息会被放入一个所谓的死信队列中。此时，消息发布者可以选择某些参数来处理这些特殊情况。
# 交换机和交换机类型

代理提供了四种交换机：
1、Direct exchange（直连交换机）	(Empty string) and amq.direct
2、Fanout exchange（扇型交换机）	amq.fanout
3、Topic exchange（主题交换机）	amq.topic
4、Headers exchange（头交换机）	amq.match (and amq.headers in RabbitMQ)
除交换机类型外，在声明交换机时还可以附带许多其他的属性，其中最重要的几个分别是：
1、Name
2、Durability （消息代理重启后，交换机是否还存在）
3、Auto-delete （当所有与之绑定的消息队列都完成了对此交换机的使用后，删掉它）
4、Arguments（依赖代理本身）
交换机可以有两个状态：持久（durable）、暂存（transient）。持久化的交换机会在消息代理（broker）重启后依旧存在，而暂存的交换机则不会（它们需要在代理再次上线后重新被声明）。然而并不是所有的应用场景都需要持久化的交换机。
# 直连交换机
直连型交换机（direct exchange）是根据消息携带的路由键（routing key）将消息投递给对应队列的

# 扇型交换机
扇型交换机（funout exchange）将消息路由给绑定到它身上的所有队列，而不理会绑定的路由键。

# 主题交换机
主题交换机（topic exchanges）通过对消息的路由键和队列到交换机的绑定模式之间的匹配

# 头交换机
有时消息的路由操作会涉及到多个属性，此时使用消息头就比用路由键更容易表达，头交换机（headers exchange）就是为此而生的。头交换机使用多个消息属性来代替路由键建立路由规则。通过判断消息头的值能否与指定的绑定相匹配来确立路由规则。