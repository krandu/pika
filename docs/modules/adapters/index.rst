Connection Adapters
===================
链接适配器
===================
Pika uses connection adapters to provide a flexible method for adapting pika's
core communication to different IOLoop implementations. In addition to asynchronous adapters, there is the :class:`BlockingConnection <pika.adapters.blocking_connection.BlockingConnection>` adapter that provides a more idomatic procedural approach to using Pika.
Pika 的核心适配器不同于IOLoop的方式，它有灵活的方法。Pika除了异步适配器, :class:`BlockingConnection <pika.adapters.blocking_connection.BlockingConnection>` 适配器，提供了更符合习惯的使用方式。

Adapters
--------
.. toctree::
   :glob:
   :maxdepth: 1

   blocking
   select
   tornado
   twisted
