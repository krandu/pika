Ensuring well-behaved connection with heartbeat and blocked-connection timeouts
===============================================================================
用连接心跳和阻塞超时确保正确的连接行为
===============================================================================

This example demonstrates explicit setting of heartbeat and blocked connection timeouts.

这个例子明确演示了设置心跳和阻塞超时

Starting with RabbitMQ 3.5.5, the broker's default hearbeat timeout decreased from 580 seconds to 60 seconds. As a result, applications that perform lengthy processing in the same thread that also runs their Pika connection may experience unexpected dropped connections due to heartbeat timeout. Here, we specify an explicit lower bound for heartbeat timeout.

从RabbitMQ 3.5.5开始，官方默认的心跳超时从580秒减少到60秒。其结果是，运行Pika的线程可能要处理繁重的任务从而断掉连接由于心跳超时。在这里，我们指定一个明确的心跳超时下限。

When RabbitMQ broker is running out of certain resources, such as memory and disk space, it may block connections that are performing resource-consuming operations, such as publishing messages. Once a connection is blocked, RabbiMQ stops reading from that connection's socket, so no commands from the client will get through to te broker on that connection until the broker unblocks it. A blocked connection may last for an indefinite period of time, stalling the connection and possibly resulting in a hang (e.g., in BlockingConnection) until the connection is unblocked. Blocked Connectin Timeout is intended to interrupt (i.e., drop) a connection that has been blocked longer than the given timeout value.

当RabbitMQ中间件运行的时候需要一些资源，像内存和磁盘空间，它可能会因为资源的耗尽而阻塞一些操作，像消息推送。一旦连接被阻塞，RabbiMQ停止从这个socket连接读取信息，因此就没有了从中间件发往客户端的消息，直到连接被释放。连接被阻塞的时间是不确定的，阻塞的连接有可能被挂起，直到连接被释放。给定阻塞超时是为了防止连接被阻塞的时间太长而断掉。

Example of configuring hertbeat and blocked-connection timeouts::

一个关于配置超时和心跳的例子::

    import pika


    def main():

        # NOTE: These paramerers work with all Pika connection types
        #注意这些参数在Pika的任何连接方式中都是适用的
        params = pika.ConnectionParameters(heartbeat_interval=600,
                                           blocked_connection_timeout=300)

        conn = pika.BlockingConnection(params)

        chan = conn.channel()

        chan.basic_publish('', 'my-alphabet-queue', "abc")

        # If publish causes the connection to become blocked, then this conn.close()
        # would hang until the connection is unblocked, if ever. However, the
        # blocked_connection_timeout connection parameter would interrupt the wait,
        # resulting in ConnectionClosed exception from BlockingConnection (or the
        # on_connection_closed callback call in an asynchronous adapter)
        # 如果推送的连接被阻塞 conn.close() 将会挂起连接 直到连接被释放
        # 然而，blocked_connection_timeout 参数有可能中断等待，导致BlockingConnection发生ConnectionClosed异常（在异步的适配器中on_connection_closed回调函数将会被调用）
        conn.close()


    if __name__ == '__main__':
        main()
