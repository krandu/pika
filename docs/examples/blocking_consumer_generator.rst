Using the BlockingChannel.consume generator to consume messages
===============================================================
使用BlockingChannel.consume协程作为消费者来处理消息
===============================================================

.. _example_blocking_basic_get:

The :py:meth:`BlockingChannel.consume <pika.adapters.blocking_connection.BlockingChannel.consume>` method is a generator that will return a tuple of method, properties and body.

:py:meth:`BlockingChannel.consume <pika.adapters.blocking_connection.BlockingChannel.consume>`  方法是一个协程将要返回一个元组包含method, properties and body.

When you escape out of the loop, be sure to call consumer.cancel() to return any unprocessed messages.

当你跳出循环，一定要调用consumer.cancel()返回所有未被处理的消息。

Example of consuming messages and acknowledging them::

消费者例子，并发送应答::

        import pika

        connection = pika.BlockingConnection()
        channel = connection.channel()

        # Get ten messages and break out
        #得到10个消息然后退出
        for method_frame, properties, body in channel.consume('test'):

            # Display the message parts
            #显示该消息部分
            print(method_frame)
            print(properties)
            print(body)

            # Acknowledge the message
            #回复消息
            channel.basic_ack(method_frame.delivery_tag)

            # Escape out of the loop after 10 messages
            #接收10条消息后跳出循环
            if method_frame.delivery_tag == 10:
                break

        # Cancel the consumer and return any pending messages
        #取消消费者 返回所有未被处理的消息
        requeued_messages = channel.cancel()
        print('Requeued %i messages' % requeued_messages)

        # Close the channel and the connection
        #关闭信道和连接
        channel.close()
        connection.close()

If you have pending messages in the test queue, your output should look something like::

如果你在队列中有未被处理的消息，会得到这样的输出。

        (pika)gmr-0x02:pika gmr$ python blocking_nack.py
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=1', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=2', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=3', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=4', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=5', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=6', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=7', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=8', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=9', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        <Basic.Deliver(['consumer_tag=ctag1.0', 'redelivered=True', 'routing_key=test', 'delivery_tag=10', 'exchange=test'])>
        <BasicProperties(['delivery_mode=1', 'content_type=text/plain'])>
        Hello World!
        Requeued 1894 messages
