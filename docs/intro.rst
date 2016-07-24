Introduction to Pika

Pika 简介
====================

IO and Event Looping

IO和事件循环
--------------------
As AMQP is a two-way RPC protocol where the client can send requests to the server and the server can send requests to a client, Pika implements or extends IO loops in each of its asynchronous connection adapters. These IO loops are blocking methods which loop and listen for events. Each asynchronous adapters follows the same standard for invoking the IO loop. The IO loop is created when the connection adapter is created. To start an IO loop for any given adapter, call the ``connection.ioloop.start()`` method.

AMQP是一个双向的RPC协议，客户机可以发送请求到服务器，服务器可以发送请求到客户端，Pika实现和扩展了在每个异步连接适配器中的IO循环。这些IO循环和和同步方法会循环和监听事件。每个异步适配器在调用IO循环中都遵循相同的标准。当连接适配器创建的时候IO循环也会创建。任何一个适配器调用 ``connection.ioloop.start()`` 方法将会开始IO循环。


If you are using an external IO loop such as Tornado's :class:`~tornado.ioloop.IOLoop`, you invoke it as you normally would and then add the adapter to it.

如果你调用的是外部的IO循环像：Tornado's :class:`~tornado.ioloop.IOLoop`,你可以将它作为适配器增加到Pika然后调用它。

Example::

    import pika

    def on_open(connection):
        # Invoked when the connection is open
        #当连接打开的时候将会调用
        pass

    # Create our connection object, passing in the on_open method
    # 创建我们的链接对象，将我们的on_open方法作为回调传入。
    connection = pika.SelectConnection(on_open_callback=on_open)

    try:
        # Loop so we can communicate with RabbitMQ
        #开始循环 这样我们就可以链接到RabbitMQ服务器
        connection.ioloop.start()
    except KeyboardInterrupt:
        # Gracefully close the connection
        #安全的关闭链接
        connection.close()
        # Loop until we're fully closed, will stop on its own
        # 直到我们完全关闭链接，循环将停止工作
        connection.ioloop.start()

.. _intro_to_cps:

Continuation-Passing Style
--------------------------
接上部分
--------------------------
Interfacing with Pika asynchronously is done by passing in callback methods you would like to have invoked when a certain event has completed. For example, if you are going to declare a queue, you pass in a method that will be called when the RabbitMQ server returns a `Queue.DeclareOk <http://www.rabbitmq.com/amqp-0-9-1-quickref.html#queue.declare>`_ response.

在异步连接中Pika是通过回调接口完成事件处理。例如，如果声明一个queue，声明的方法将会在 RabbitMQ 服务器返回一个 `Queue.DeclareOk <http://www.rabbitmq.com/amqp-0-9-1-quickref.html#queue.declare>`_ 响应的时候调用.


In our example below we use the following four easy steps:

在我们的例子中我们用了下面的四个简单步骤:

#. We start by creating our connection object, then starting our event loop.

#. 首先，我们创建连接对象，然后开始事件循环

#. When we are connected, the *on_connected* method is called. In that method we create a channel.

#. 当我们连接时，* onconnected*方法被调用。在这种方法中，我们创建一个通道。

#. When the channel is created, the *on_channel_open* method is called. In that method we declare a queue.

#. 当创建管道时，* on_channel_open*方法被调用。在这种方法中，我们初始化一个队列。

#. When the queue is declared successfully, *on_queue_declared* is called. In that method we call :py:meth:`channel.basic_consume <channel.Channel.basic_consume>` telling it to call the handle_delivery for each message RabbitMQ delivers to us.

#. 当队列初始化成功，* on_queue_declared*被调用。在这个方法中，我们用 :py:meth:`channel.basic_consume <channel.Channel.basic_consume>` 注册handle_delivery方法处理RabbitMQ传递给我们的每个消息。

#. When RabbitMQ has a message to send us, it call the handle_delivery method passing the AMQP Method frame, Header frame and Body.

#. 当RabbitMQ有一个信息发送给我们，调用handle_delivery方法，将 AMQP Method frame, Header frame and Body传递给handle_delivery。

.. NOTE::

.. 注意::
    Step #1 is on line #28 and Step #2 is on line #6. This is so that Python knows about the functions we'll call in Steps #2 through #5.

    Step #1 is on line #28 and Step #2 is on line #6. This is so that Python knows about the functions we'll call in Steps #2 through #5.

.. _cps_example:

Example::

    import pika

    # Create a global channel variable to hold our channel object in
    #创建一个global变量来保存管道对象
    channel = None

    # Step #2
    def on_connected(connection):
        """
            Called when we are fully connected to RabbitMQ
            我们连接到的RabbitMQ的时候调用
        """
        # Open a channel
        connection.channel(on_channel_open)

    # Step #3
    def on_channel_open(new_channel):
        """
            Called when our channel has opened
            当管道打开的时候调用 (初始化队列)
        """
        global channel
        channel = new_channel
        channel.queue_declare(queue="test", durable=True, exclusive=False, auto_delete=False, callback=on_queue_declared)

    # Step #4
    def on_queue_declared(frame):
        """
            Called when RabbitMQ has told us our Queue has been declared, frame is the response from RabbitMQ
            当RabbitMQ告诉我们我们的队列已经初始化成功调用，frame是RabbitMQ的响应数据。
        """
        channel.basic_consume(handle_delivery, queue='test')

    # Step #5
    def handle_delivery(channel, method, header, body):
        """ 
            Called when we receive a message from RabbitMQ
            接受到RabbitMQ消息的时候调用
        """
        print(body)

    # Step #1: Connect to RabbitMQ using the default parameters
    #          连接到RabbitMQ使用默认参数
    parameters = pika.ConnectionParameters()
    connection = pika.SelectConnection(parameters, on_connected)

    try:
        # Loop so we can communicate with RabbitMQ
        connection.ioloop.start()
    except KeyboardInterrupt:
        # Gracefully close the connection
        #关闭链接
        connection.close()
        # Loop until we're fully closed, will stop on its own
        #  直到我们完全关闭链接，循环将停止工作
        connection.ioloop.start()

Credentials

认证
-----------
:mod:`pika.credentials` module provides the mechanism by which you pass the username and password to the :py:class:`ConnectionParameters <pika.connection.ConnectionParameters>` class when it is created.

:mod:`pika.credentials` 模块提供用户名和密码的链接机制。当创建 :py:class:`ConnectionParameters <pika.connection.ConnectionParameters>` 时会创建:mod:`pika.credentials`。

Example::

    import pika
    credentials = pika.PlainCredentials('username', 'password')
    parameters = pika.ConnectionParameters(credentials=credentials)

.. _connection_parameters:

Connection Parameters

连接参数
---------------------
There are two types of connection parameter classes in Pika to allow you to pass the connection information into a connection adapter, :class:`ConnectionParameters <pika.connection.ConnectionParameters>` and :class:`URLParameters <pika.connection.URLParameters>`. Both classes share the same default connection values.

有两种类型的Pika连接参数类允许你传递连接信息到适配器，类：`ConnectionParameters<pika.connection.ConnectionParameters>`和类：`URLParameters<pika.connection.URLParameters>`。这两个类都共享相同的默认连接值。

.. _intro_to_backpressure:

TCP Backpressure
----------------

As of RabbitMQ 2.0, client side `Channel.Flow <http://www.rabbitmq.com/amqp-0-9-1-quickref.html#channel.flow>`_ has been removed [#f1]_. Instead, the RabbitMQ broker uses TCP Backpressure to slow your client if it is delivering messages too fast. If you pass in backpressure_detection into your connection parameters, Pika attempts to help you handle this situation by providing a mechanism by which you may be notified if Pika has noticed too many frames have yet to be delivered. By registering a callback function with the :py:meth:`add_backpressure_callback <pika.connection.Connection.add_backpressure_callback>` method of any connection adapter, your function will be called when Pika sees that a backlog of 10 times the average frame size you have been sending has been exceeded. You may tweak the notification multiplier value by calling the :py:meth:`set_backpressure_multiplier <pika.connection.Connection.set_backpressure_multiplier>` method passing any integer value.

RabbitMQ 2.0的客户端中的 `Channel.Flow <http://www.rabbitmq.com/amqp-0-9-1-quickref.html#channel.flow>`_ 已经删除 [#f1]_. 相反RabbitMQ broker 使用 TCP Backpressure 如果传递消息太快而你的客户端处理消息太慢。假设你传递了 backpressure_detection到你的连接参数中，如果Pika已经注意到太多frames没有被处理，Pika试图帮助你处理这种情况通过提供一种框架给你。通过注册一个回调函数： :py:meth:`add_backpressure_callback <pika.connection.Connection.add_backpressure_callback>`  这个方法使用于任何适配器，当Pika发现你发送的消息平均积压的frame超过10个这个方法将要被调用。你可以通过调用 :py:meth:`set_backpressure_multiplier <pika.connection.Connection.set_backpressure_multiplier>` 来传递任何整数值。

Example::

    import pika

    parameters = pika.URLParameters('amqp://guest:guest@rabbit-server1:5672/%2F?backpressure_detection=t')

.. rubric:: Footnotes
.. rubric:: 脚注

.. [#f1] "more effective flow control mechanism that does not require cooperation from clients and reacts quickly to prevent the broker from exhausing memory - see http://www.rabbitmq.com/extensions.html#memsup" from http://lists.rabbitmq.com/pipermail/rabbitmq-announce/attachments/20100825/2c672695/attachment.txt

.. [#f1] "更有效的内存管理，不需要依赖客户端控制，有更有效的流动控制机制防止数据从内存溢出。请看 - see http://www.rabbitmq.com/extensions.html#memsup" from http://lists.rabbitmq.com/pipermail/rabbitmq-announce/attachments/20100825/2c672695/attachment.txt

