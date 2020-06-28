#### RabbitMQ是基于AMQP协议的，通过使用通用协议就可以做到在不同语言之间传递

### AMQP协议

**核心概念**

1. server：又称broker，接受客户端连接，实现AMQP实体服务。
2. connection：连接和具体broker网络连接。
3. channel：网络信道，几乎所有操作都在channel中进行，channel是消息读写的通道。客户端可以建立多个channel，每个channel表示一个会话任务。
4. message：消息，服务器和应用程序之间传递的数据，由properties和body组成。properties可以对消息进行修饰，比如消息的优先级，延迟等高级特性；body是消息实体内容。
5. Virtual host：虚拟主机，用于逻辑隔离，最上层消息的路由。一个virtual host可以若干个Exchange和Queue，同一个Virtual host不能有同名的Exchange或Queue。
6. Exchange：交换机，接受消息，根据路由键转发消息到绑定的队列上。
7. banding：Exchange和Queue之间虚拟连接，binding中可以包括routing key
8. routing key：一个路由规则，虚拟机根据它来确定如何路由一条消息。
9. Queue：消息队列，用来存放消息的队列。

**Exchange**

**定义：**接受消息，并根据路由键转发消息所绑定的队列

1. Direct Exchange，所有发送到Direct Exchange的消息被转发到RouteKey中指定的Queue，Direct Exchange可以使用默认的Exchange，默认的Exchange会绑定所有的队列，所以Direct可以直接使用Queue名（作为routing key）绑定。或者消费者和生产者的routing key完全匹配。





 

