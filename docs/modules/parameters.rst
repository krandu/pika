Connection Parameters
=====================
链接参数
=====================
To maintain flexibility in how you specify the connection information required for your applications to properly connect to RabbitMQ, pika implements two classes for encapsulating the information, :class:`~pika.connection.ConnectionParameters` and :class:`~pika.connection.URLParameters`.
为了让应用程序可以灵活的连接到RabbitMQ服务器，pika封装了2个类来接收链接信息：:class:`~pika.connection.ConnectionParameters` and :class:`~pika.connection.URLParameters`. 

ConnectionParameters
--------------------
常规参数
--------------------
The classic object for specifying all of the connection parameters required to connect to RabbitMQ, :class:`~pika.connection.ConnectionParameters` provides attributes for tweaking every possible connection option.
:class:`~pika.connection.ConnectionParameters` 类对象要求你提供一些常规的链接参数来作为类属性。

Example::

    import pika

    # Set the connection parameters to connect to rabbit-server1 on port 5672
    # on the / virtual host using the username "guest" and password "guest"
    credentials = pika.PlainCredentials('guest', 'guest')
    parameters = pika.ConnectionParameters('rabbit-server1',
                                           5672,
                                           '/',
                                           credentials)

.. autoclass:: pika.connection.ConnectionParameters
   :members:
   :inherited-members:
   :member-order: bysource

URLParameters
-------------
URL参数
-------------
The :class:`~pika.connection.URLParameters` class allows you to pass in an AMQP URL when creating the object and supports the host, port, virtual host, ssl, username and password in the base URL and other options are passed in via query parameters.
:class:`~pika.connection.URLParameters` 类创建的时候要传递一个URL作为链接参数，它包含主机 端口 虚拟记住 ssl 用户名和密码基本参数和一些查询参数。

Example::

    import pika

    # Set the connection parameters to connect to rabbit-server1 on port 5672
    # on the / virtual host using the username "guest" and password "guest"
    parameters = pika.URLParameters('amqp://guest:guest@rabbit-server1:5672/%2F')

.. autoclass:: pika.connection.URLParameters
   :members:
   :inherited-members:
   :member-order: bysource

