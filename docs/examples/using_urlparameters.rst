Using URLParameters
===================
使用URL参数
===================
Pika has two methods of encapsulating the data that lets it know how to connect
to RabbitMQ, :py:class:`pika.connection.ConnectionParameters` and :py:class:`pika.connection.URLParameters`.

Pika封装了2个方法可以让我们连接至RabbitMQ，分别是：:py:class:`pika.connection.ConnectionParameters` and :py:class:`pika.connection.URLParameters`.

.. note::
    If you're connecting to RabbitMQ on localhost on port 5672, with the default virtual host of */* and the default username and password of *guest* and *guest*, you do not need to specify connection parameters when connecting.

.. 注意::
    如果你连接的是RabbitMQ的本地5672端口的虚拟主机，用的是默认的用户名和密码 *guest*  *guest*，你不需要给连接对象传递特定的参数。

Using :py:class:`pika.connection.URLParameters` is an easy way to minimize the
variables required to connect to RabbitMQ and supports all of the directives
that :py:class:`pika.connection.ConnectionParameters` supports.

用 :py:class:`pika.connection.URLParameters` 只需要极少的参数就可以连接到RabbitMQ并且起到的作用和 :py:class:`pika.connection.ConnectionParameters` 一样。

The following is the format for the URLParameters connection value::

  scheme://username:password@host:port/virtual_host?key=value&key=value

As you can see, by default, the scheme (amqp, amqps), username, password, host, port and virtual host make up the core of the URL and any other parameter is passed in as query string values.

下面是URLParameters连接对象的参数::

  scheme://username:password@host:port/virtual_host?key=value&key=value

就像你看到的，默认的参数(amqp, amqps),用户名，密码，主机，端口和虚拟主机构成了url的核心其他的参数是一些查询字符串。

Example Connection URLS
-----------------------
连接URL的例子
-----------------------

The default connection URL connects to the / virtual host as guest using the guest password on localhost port 5672. Note the forwardslash in the URL is encoded to %2F::

默认的URL连接是以guest为用户名guest为密码连接至本机5672端口的虚拟主机。注意在URL中反斜杠的编码为 %2F::

  amqp://guest:guest@localhost:5672/%2F

Connect to a host *rabbit1* as the user *www-data* using the password *rabbit_pwd* on the virtual host *web_messages*::

连接到  *rabbit1* 主机的 *web_messages* 虚拟主机上，用户名为 *rabbit1* 密码为
*www-data* ::

  amqp://www-data:rabbit_pwd@rabbit1/web_messages

Connecting via SSL is pretty easy too. To connect via SSL for the previous example, simply change the scheme to *amqps*. If you do not specify a port, Pika will use the default SSL port of 5671::

用SSl连接是简单的。像上面的连接例子一样，只需要改变连接方式为 *amqps*.如果你不指定端口，Pika将要用默认的SSL端口5671::

  amqps://www-data:rabbit_pwd@rabbit1/web_messages

If you're looking to tweak other parameters, such as enabling heartbeats, simply add the key/value pair as a query string value. The following builds upon the SSL connection, enabling heartbeats every 30 seconds::

如果你想设置一些其他的参数，例如想设置心跳间隔，只需要增加一个key/value键值对作为查询参数。下面用SSL建立的连接，设置的心跳连接为30s::

  amqps://www-data:rabbit_pwd@rabbit1/web_messages?heartbeat=30

Options that are available as query string values:

可以作为查询字符串的值：

- backpressure_detection: Pass in a value of *t* to enable backpressure detection, it is disabled by default.

- backpressure_detection: 通过设置 backpressure detection为 *t* 开启，默认为关闭状态

- channel_max: Alter the default channel maximum by passing in a 32-bit integer value here

- channel_max: 改变信道的默认最大值，通过设置32位的int值

- connection_attempts: Alter the default of 1 connection attempt by passing in an integer value here [#f1]_.

- connection_attempts: 通过在这里设置一个int值改变默认的连接尝试次数

- frame_max: Alter the default frame maximum size value by passing in a long integer value [#f2]_.

- frame_max: 通过在这里设置一个int类型的值改变传递消息的最大大小

- heartbeat: Pass a value greater than zero to enable heartbeats between the server and your application. The integer value you pass here will be the number of seconds between heartbeats.

- heartbeat: 通过设置大于0的值规定服务器和你应用程序之间的连接心跳。你在这里传递的数值是两者之间心跳的秒数。

- locale: Set the locale of the client using underscore delimited posix Locale code in ll_CC format (en_US, pt_BR, de_DE).

- locale: 用 underscore delimited posix 设置客户端的 ll_CC 格式 (en_US, pt_BR, de_DE)。

- retry_delay: The number of seconds to wait before attempting to reconnect on a failed connection, if connection_attempts is > 0.

- retry_delay: 如果connection_attempts 大于 0，这个数值是连接之后再次尝试连接的间隔秒数

- socket_timeout: Change the default socket timeout duration from 0.25 seconds to another integer or float value. Adjust with caution.

- socket_timeout: 改变默认的连接超时时间，设置一个int或者float类型的值。谨慎修改

- ssl_options: A url encoded dict of values for the SSL connection. The available keys are:

- ssl_options: 在进行SSL连接时url encoded字典的值，可以使用的值：

   - ca_certs
   - cert_reqs
   - certfile
   - keyfile
   - ssl_version

For an information on what the ssl_options can be set to reference the `official Python documentation <http://docs.python.org/2/library/ssl.html>`_. Here is an example of setting the client certificate and key::

对于什么是ssl_options可以参考  `official Python documentation <http://docs.python.org/2/library/ssl.html>`_ .  这里是一个设置证书和秘钥的例子::

  amqp://www-data:rabbit_pwd@rabbit1/web_messages?heartbeat=30&ssl_options=%7B%27keyfile%27%3A+%27%2Fetc%2Fssl%2Fmykey.pem%27%2C+%27certfile%27%3A+%27%2Fetc%2Fssl%2Fmycert.pem%27%7D

The following example demonstrates how to generate the ssl_options string with `Python's urllib <http://docs.python.org/2/library/urllib.html>`_::

下面例子演示了用  `Python's urllib <http://docs.python.org/2/library/urllib.html>`_:: 生成ssl_options字符串的例子::

    import urllib
    urllib.urlencode({'ssl_options': {'certfile': '/etc/ssl/mycert.pem', 'keyfile': '/etc/ssl/mykey.pem'}})


.. rubric:: Footnotes

.. [#f1] The :py:class:`pika.adapters.blocking_connection.BlockingConnection` adapter does not respect the *connection_attempts* parameter.

.. [#f1] 这个 :py:class:`pika.adapters.blocking_connection.BlockingConnection` 适配器不需要 *connection_attempts* 参数.

.. [#f2] The AMQP specification states that a server can reject a request for a frame size larger than the value it passes during content negotiation.

.. [#f2] 同时AMQP规范规定 一个请求可以拒绝的frame大小大于参数规定的值