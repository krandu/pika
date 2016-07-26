Core Class and Module Documentation
===================================
核心类和模块文档
===================================
For the end user, Pika is organized into a small set of objects for all communication with RabbitMQ.
对所有用户来说，Pika是一个轻量级的链接RabbitMQ的库。

- A :doc:`connection adapter <adapters/index>` is used to connect to RabbitMQ and manages the connection.

- :doc:`connection adapter <adapters/index>` 是一个链接和管理RabbitMQ服务器的对象

- :doc:`Connection parameters <parameters>` are used to instruct the :class:`~pika.connection.Connection` object how to connect to RabbitMQ.

- :doc:`Connection parameters <parameters>` 对象是 :class:`~pika.connection.Connection` 对象链接RabbitMQ服务器的参数。


- :doc:`credentials` are used to encapsulate all authentication information for the :class:`~pika.connection.ConnectionParameters` class.

- :doc:`credentials` 对象为 :class:`~pika.connection.ConnectionParameters` 对象封装了所有验证信息.


- A :class:`~pika.channel.Channel` object is used to communicate with RabbitMQ via the AMQP RPC methods.

- A :class:`~pika.channel.Channel` 对象是用来经AMQP RPC方法与RabbitMQ的通信。


- :doc:`exceptions` are raised at various points when using Pika when something goes wrong.

- :doc:`exceptions` 用来在Pika发生错误时处理异常。

.. toctree::
   :hidden:
   :maxdepth: 1

   adapters/index
   channel
   connection
   credentials
   exceptions
   parameters
   spec
