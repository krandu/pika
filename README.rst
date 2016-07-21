Pika
====
Pika is a RabbitMQ (AMQP-0-9-1) client library for Python.


|Version| |Downloads| |Status| |Coverage| |License| |Docs|

Introduction
-------------
Pika is a pure-Python implementation of the AMQP 0-9-1 protocol including RabbitMQ's
extensions.

- Python 2.6+ and 3.3+ are supported.

- Since threads aren't appropriate to every situation, it doesn't
  require threads. It takes care not to forbid them, either. The same
  goes for greenlets, callbacks, continuations and generators. It is
  not necessarily thread-safe however, and your mileage will vary.
- 每个解决方案引入线程的方式并不一定合适，因此Pika不要求线程，也不禁止使用线程。同时，其它的方法包括并发, 回调,长连接和协程.它不保证线程安全，因此需要选择适合自己的。

- People may be using direct sockets, plain old `select()`,
  or any of the wide variety of ways of getting network events to and from a
  python application. Pika tries to stay compatible with all of these, and to
  make adapting it to a new environment as simple as possible.
- 对于在老版本的Python应用程序中使用的sockets，select或者各种各样的网络事件，Pika尽可能的兼容他们，并且在新的环境中使用尽量简单。

Documentation
-------------
Pika's documentation can be found at `https://pika.readthedocs.org <https://pika.readthedocs.org>`_

Example
-------
Here is the most simple example of use, sending a message with the BlockingConnection adapter:
这里是一个非常简单的例子，使用BlockingConnection适配器发送了一个消息(生产者)：

.. code :: python

    import pika
    connection = pika.BlockingConnection()
    channel = connection.channel()
    channel.basic_publish(exchange='example',
                          routing_key='test',
                          body='Test Message')
    connection.close()

And an example of writing a blocking consumer:

.. code :: python

    import pika
    connection = pika.BlockingConnection()
    channel = connection.channel()

    for method_frame, properties, body in channel.consume('test'):

        # Display the message parts and ack the message
        print(method_frame, properties, body)
        channel.basic_ack(method_frame.delivery_tag)

        # Escape out of the loop after 10 messages
        if method_frame.delivery_tag == 10:
            break

    # Cancel the consumer and return any pending messages
    requeued_messages = channel.cancel()
    print('Requeued %i messages' % requeued_messages)
    connection.close()

Pika provides the following adapters
------------------------------------

- BlockingConnection - enables blocking, synchronous operation on top of library for simple uses
- LibevConnection    - adapter for use with the libev event loop http://libev.schmorp.de
- SelectConnection   - fast asynchronous adapter
- TornadoConnection  - adapter for use with the Tornado IO Loop http://tornadoweb.org
- TwistedConnection  - adapter for use with the Twisted asynchronous package http://twistedmatrix.com/

Contributing
------------
To contribute to pika, please make sure that any new features or changes
to existing functionality **include test coverage**.

*Pull requests that add or change code without coverage will most likely be rejected.*

Additionally, please format your code using `yapf <http://pypi.python.org/pypi/yapf>`_
with ``google`` style prior to issuing your pull request.

.. |Version| image:: https://img.shields.io/pypi/v/pika.svg?
   :target: http://badge.fury.io/py/pika

.. |Status| image:: https://img.shields.io/travis/pika/pika.svg?
   :target: https://travis-ci.org/pika/pika

.. |Coverage| image:: https://img.shields.io/codecov/c/github/pika/pika.svg?
   :target: https://codecov.io/github/pika/pika?branch=master

.. |Downloads| image:: https://img.shields.io/pypi/dm/pika.svg?
   :target: https://pypi.python.org/pypi/pika

.. |License| image:: https://img.shields.io/pypi/l/pika.svg?
   :target: https://pika.readthedocs.org

.. |Docs| image:: https://readthedocs.org/projects/pika/badge/?version=stable
   :target: https://pika.readthedocs.org
   :alt: Documentation Status
