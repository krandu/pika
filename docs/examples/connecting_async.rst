Connecting to RabbitMQ with Callback-Passing Style
==================================================
连接到RabbitMQ用回调传参的方式
==================================================

When you connect to RabbitMQ with an asynchronous adapter, you are writing event
oriented code. The connection adapter will block on the IOLoop that is watching
to see when pika should read data from and write data to RabbitMQ. Because you're
now blocking on the IOLoop, you will receive callback notifications when specific
events happen.

当你用异步适配器连接到RabbitMQ的时候，你可以在代码中重写一个事件。当Pika在RabbitMQ中读数据或者写数据的时候连接的适配器是被阻断的。因为这时的IOLoop是阻断的，当一个具体的事件发生的时候，注册的回调事件将要接收到一个通知（会被触发）。

Example Code
------------
In the example, there are three steps that take place:

下面这个例子有4步，在其他地方有3步：

1. Setup the connection to RabbitMQ
1. 设置到RabbitMQ的连接
2. Start the IOLoop
2. 开始IOLoop
3. Once connected, the on_open method will be called by Pika with a handle to
   the connection. In this method, a new channel will be opened on the connection.
3. 一旦连接打开，Pika将会用连接对象调用on_open方法。在这个方法中一个新的信道         会在这个连接中打开。
4. Once the channel is opened, you can do your other actions, whether they be
   publishing messages, consuming messages or other RabbitMQ related activities.::
4. 一旦信道打开，就可以做其他一些操作了，推送消息、接收消息或者和RabbitMQ有关的操    作。

    import pika

    # Step #3
    def on_open(connection):
        connection.channel(on_channel_open)

    # Step #4
    def on_channel_open(channel):
        channel.basic_publish('exchange_name',
                              'routing_key',
                              'Test Message',
                              pika.BasicProperties(content_type='text/plain',
                                                   type='example'))

    # Step #1: Connect to RabbitMQ
    connection = pika.SelectConnection(on_open_callback=on_open)

    try:
        # Step #2 - Block on the IOLoop
        connection.ioloop.start()

    # Catch a Keyboard Interrupt to make sure that the connection is closed cleanly
    #当Keyboard  Interrupt 触发的时候确保连接被全部关闭
    except KeyboardInterrupt:

        # Gracefully close the connection
        connection.close()

        # Start the IOLoop again so Pika can communicate, it will stop on its own when the connection is closed
        #当连接被关闭的时候Pika会被停止，再次开始IOLoop Pika会继续运行处理缓存的消息
        connection.ioloop.start()
