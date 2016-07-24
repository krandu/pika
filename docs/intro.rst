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

接上部分

--------------------------
Interfacing with Pika asynchronously is done by passing in callback methods you would like to have invoked when a certain event has completed. For example, if you are going to declare a queue, you pass in a method that will be called when the RabbitMQ server returns a `Queue.DeclareOk <http://www.rabbitmq.com/amqp-0-9-1-quickref.html#queue.declare>`_ response.

在异步连接中Pika是通过回调接口完成事件处理。例如，如果声明一个queue，声明的方法将会在 RabbitMQ 服务器返回一个 `Queue.DeclareOk <http://www.rabbitmq.com/amqp-0-9-1-quickref.html#queue.declare>`_ 响应的时候调用.

In our example below we use the following four easy steps:

在我们的例子中我们用了下面的四个简单步骤:
