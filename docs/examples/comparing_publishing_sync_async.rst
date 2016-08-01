Comparing Message Publishing with BlockingConnection and SelectConnection
=========================================================================
用 BlockingConnection and SelectConnection作为生产者对比
=========================================================================

For those doing simple, non-asynchronous programing, :py:meth:`pika.adapters.blocking_connection.BlockingConnection` proves to be the easiest way to get up and running with Pika to publish messages. 

对于一些简单同步项目，Pika的 :py:meth:`pika.adapters.blocking_connection.BlockingConnection` 方法提供了一种简单的方式实现生产者模式。

In the following example, a connection is made to RabbitMQ listening to port *5672* on *localhost* using the username *guest* and password *guest* and virtual host */*. Once connected, a channel is opened and a message is published to the *test_exchange* exchange using the *test_routing_key* routing key. The BasicProperties value passed in sets the message to delivery mode *1* (non-persisted) with a content-type of *text/plain*. Once the message is published, the connection is closed::

在下面的例子中，这个连接使用用户名*guest*密码*guest*连接到本地主机中RabbitMQ的虚拟主机 通过监听的*5672*端口，一旦连接信道已经打开消息将会通过*test_routing_key*发送到*test_exchange*交换分区。给BasicProperties设置值规定消息的交付模式为*1*（非持久化）消息的类型为 *text/plain*。一旦消息传递完成 连接将关闭。

  import pika

  parameters = pika.URLParameters('amqp://guest:guest@localhost:5672/%2F')

  connection = pika.BlockingConnection(parameters)

  channel = connection.channel()

  channel.basic_publish('test_exchange',
                        'test_routing_key',
                        'message body value',
                        pika.BasicProperties(content_type='text/plain',
                                             delivery_mode=1))

  connection.close()


In contrast, using :py:meth:`pika.adapters.select_connection.SelectConnection` and the other asynchronous adapters is more complicated and less pythonic, but when used with other asynchronous services can have tremendous performance improvements. In the following code example, all of the same parameters and values are used as were used in the previous example::

相比之下，使用 :py:meth:`pika.adapters.select_connection.SelectConnection`  和其他异步适配器是比较复杂 但是代码更少，但是如果我们用了异步的服务会有性能的巨大提升。在下面的代码中，相同参数和值的作用和上面代码的作用相同。

    import pika

    # Step #3
    def on_open(connection):

        connection.channel(on_channel_open)

    # Step #4
    def on_channel_open(channel):

        channel.basic_publish('test_exchange',
                                'test_routing_key',
                                'message body value',
                                pika.BasicProperties(content_type='text/plain',
                                                     delivery_mode=1))

        connection.close()

    # Step #1: Connect to RabbitMQ
    parameters = pika.URLParameters('amqp://guest:guest@localhost:5672/%2F')

    connection = pika.SelectConnection(parameters=parameters,
                                       on_open_callback=on_open)

    try:

        # Step #2 - Block on the IOLoop
        connection.ioloop.start()

    # Catch a Keyboard Interrupt to make sure that the connection is closed cleanly
    #确保发生Keyboard Interrupt输入的时候连接能优雅的关闭
    except KeyboardInterrupt:

        # Gracefully close the connection
        connection.close()

        # Start the IOLoop again so Pika can communicate, it will stop on its own when the connection is closed
        #当连接关闭的时候IOLoop会停止，再次开始IOLoop Pika 会继续运行（处理缓存的消息）
        connection.ioloop.start()

