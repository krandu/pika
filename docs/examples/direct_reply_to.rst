Direct reply-to example
==============================
直接回复的例子
==============================
The following example demonstrates the use of the RabbitMQ "Direct reply-to" feature via `pika.BlockingConnection`. See https://www.rabbitmq.com/direct-reply-to.html for more info about this feature.

这里例子演示了 `pika.BlockingConnection` 直接回复的功能，更多的关于这方面的例子可以看 https://www.rabbitmq.com/direct-reply-to.html 。

direct_reply_to.py::

    # -*- coding: utf-8 -*-

    """
    This example demonstrates the RabbitMQ "Direct reply-to" usage via
    `pika.BlockingConnection`. See https://www.rabbitmq.com/direct-reply-to.html
    for more info about this feature.

    这里例子演示了 `pika.BlockingConnection` 直接回复的功能，更多的关于这方面的例子可以看 https://www.rabbitmq.com/direct-reply-to.html 。
    """
    import pika


    SERVER_QUEUE = 'rpc.server.queue'


    def main():
        """ Here, Client sends "Marco" to RPC Server, and RPC Server replies with
        "Polo".
        这里客户端发送“Marco”到RPC服务器，然后RPC服务器回复“Polo”。

        NOTE Normally, the server would be running separately from the client, but
        in this very simple example both are running in the same thread and sharing
        connection and channel.
        说明在通常情况下，客户端连接的服务端要运行在其他的地方，这个是一个非常简单的例子两者运行在同一个线程中分享同一个连接和信道。

        """
        with pika.BlockingConnection() as conn:
            channel = conn.channel()

            # Set up server
            #设置服务端

            channel.queue_declare(queue=SERVER_QUEUE,
                                  exclusive=True,
                                  auto_delete=True)
            channel.basic_consume(on_server_rx_rpc_request, queue=SERVER_QUEUE)


            # Set up client

            # NOTE Client must create its consumer and publish RPC requests on the
            # same channel to enable the RabbitMQ broker to make the necessary
            # associations.
            #
            # Also, client must create the consumer *before* starting to publish the
            # RPC requests.
            #
            # Client must create its consumer with no_ack=True, because the reply-to
            # queue isn't real.
            # 注意：客户端必须使用和生产者相同的信道，方便RabbitMQ关联他们
            # 当然了在创建消费者之前，应该开始推送消息
            # 客户端必须设置no_asc=True，因为这个回复不是消息确认

            channel.basic_consume(on_client_rx_reply_from_server,
                                  queue='amq.rabbitmq.reply-to',
                                  no_ack=True)
            channel.basic_publish(
                exchange='',
                routing_key=SERVER_QUEUE,
                body='Marco',
                properties=pika.BasicProperties(reply_to='amq.rabbitmq.reply-to'))

            channel.start_consuming()


    def on_server_rx_rpc_request(ch, method_frame, properties, body):
        print 'RPC Server got request:', body

        ch.basic_publish('', routing_key=properties.reply_to, body='Polo')

        ch.basic_ack(delivery_tag=method_frame.delivery_tag)

        print 'RPC Server says good bye'


    def on_client_rx_reply_from_server(ch, method_frame, properties, body):
        print 'RPC Client got reply:', body

        # NOTE A real client might want to make additional RPC requests, but in this
        # simple example we're closing the channel after getting our first reply
        # to force control to return from channel.start_consuming()
        #一个真正的客户端可能需要作出更多的RPC请求，但在这个简单的例子中，我们在第一个答复之#后关闭了该通道，并强制控制从channel.start_consuming()
        print 'RPC Client says bye'
        ch.close()
