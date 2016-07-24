Introduction to Pika

简介
====================

Pika is a pure-Python implementation of the AMQP 0-9-1 protocol that tries to stay fairly independent of the underlying network support library.

Pika是纯Python编写对AMQP 0-9-1协议的实现，它独立于底层的网络支持库。

If you have not developed with Pika or RabbitMQ before, the :doc:`intro` documentation is a good place to get started.

如果你以前对 Pika 或者 RabbitMQ没有了解，这里以这里的 :文档:`intro` 作为开始。

Installing Pika

安装Pika
---------------
Pika is available for download via PyPI and may be installed using easy_install or pip::

Pika可以通过PyPI下载，通过pip或者easy_install安装

    pip install pika

or::

    easy_install pika

To install from source, run "python setup.py install" in the root source directory.

通过源代码安装，到源文件的目录下运行“python setup.py install”。

Using Pika
----------
.. toctree::
   :glob:
   :maxdepth: 1

   intro
   modules/index
   examples
   faq
   contributors
   version_history

Indices and tables
------------------

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
