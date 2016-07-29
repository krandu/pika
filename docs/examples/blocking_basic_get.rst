Using the Blocking Connection to get a message from RabbitMQ
============================================================
用阻断式连接处理RabbitMQ发送的消息
============================================================

.. _example_blocking_basic_get:

.. _阻塞式处理的例子:

The :py:meth:`BlockingChannel.basic_get <pika.adapters.blocking_connection.BlockingChannel.basic_get>`  method will return a tuple with the members.

:py:meth:`BlockingChannel.basic_get <pika.adapters.blocking_connection.BlockingChannel.basic_get>` 这个方法返回一个包含各成员的元组。


If the server returns a message, the first item in the tuple will be a :class:`pika.spec.Basic.GetOk` object with the current message count, the redelivered flag, the routing key that was used to put the message in the queue, and the exchange the message was published to. The second item will be a :py:class:`~pika.spec.BasicProperties` object and the third will be the message body.

如果服务器返回消息，元组的第一项是类 :class:`pika.spec. Basic.GetOk` 对象包含当前消息计数、重传标志，路由键被生产者用来将消息放入队列和消息在交换分区中的处理。第二项是 :py:class:`~pika.spec.BasicProperties` 对象 第三项是消息体。


If the server did not return a message a tuple of None, None, None will be returned.

如果服务器没有返回消息，元组将会包含None, None, None返回。

Example of getting a message and acknowledging it::

这个例子处理消息并向服务器应答。

        import pika

        connection = pika.BlockingConnection()
        channel = connection.channel()
        #循环调用basic_get这个方法判断队列状态
        method_frame, header_frame, body = channel.basic_get('test')
        if method_frame:
            print(method_frame, header_frame, body)
            channel.basic_ack(method_frame.delivery_tag)
        else:
            print('No message returned')
