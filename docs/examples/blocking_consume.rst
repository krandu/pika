Using the Blocking Connection to consume messages from RabbitMQ
===============================================================
使用阻塞式的消费者处理RabbitMQ发来的消息
===============================================================

.. _example_blocking_basic_consume:

The :py:meth:`BlockingChannel.basic_consume <pika.adapters.blocking_connection.BlockingChannel.basic_consume>`  method assign a callback method to be called every time that RabbitMQ delivers messages to your consuming application.

每当RabbitMQ传递消息到消费者应用程序时 :py:meth:`BlockingChannel.basic_consume <pika.adapters.blocking_connection.BlockingChannel.basic_consume>` 指定的方法将会被调用。

When pika calls your method, it will pass in the channel, a :py:class:`pika.spec.Basic.Deliver` object with the delivery tag, the redelivered flag, the routing key that was used to put the message in the queue, and the exchange the message was published to. The third argument will be a :py:class:`pika.spec.BasicProperties` object and the last will be the message body.

当Pika调用注册的回调方法时，将会给方法传递channel 一个 :py:class:`pika.spec.Basic.Deliver` 对象的实例，这个对象包含消息的标签、重传标志以及路由键 路由键的作用是放消息到队列，推送消息到交换对象。第三个参数是 :py:class:`pika.spec.BasicProperties` 对象、最后一个是消息体。

Example of consuming messages and acknowledging them::

实现消费者处理消息并应答的例子::

    import pika


    def on_message(channel, method_frame, header_frame, body):
        print(method_frame.delivery_tag)
        print(body)
        print()
        channel.basic_ack(delivery_tag=method_frame.delivery_tag)


    connection = pika.BlockingConnection()
    channel = connection.channel()
    channel.basic_consume(on_message, 'test')
    try:
        channel.start_consuming()
    except KeyboardInterrupt:
        channel.stop_consuming()
    connection.close()