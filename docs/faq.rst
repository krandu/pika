Frequently Asked Questions

FAQ
--------------------------

- Is Pika thread safe?

- Pika保证线程安全吗？

    Pika does not have any notion of threading in the code. If you want to use Pika with threading, make sure you have a Pika connection per thread, created in that thread. It is not safe to share one Pika connection across threads.

  	Pika没有在线程安全上做任何保证。如果你想用线程的方式使用Pika，请在每个线程里面单独创建一个连接通道。多个线程共享一个连接是不安全的。

- How do I report a bug with Pika?

- 我怎么样为Pika报告Bug？

    The `main Pika repository <https://github.com/pika/pika>`_ is hosted on `Github <https://github.com>`_ and we use the Issue tracker at `https://github.com/pika/pika/issues <https://github.com/pika/pika/issues>`_.

    `Pika的主页 <https://github.com/pika/pika>`_ 在 `Github <https://github.com>`_ 上。 我们的issuse页面在 `https://github.com/pika/pika/issues <https://github.com/pika/pika/issues>`_.


- Is there a mailing list for Pika?

- Pika有邮件列表吗？

    Yes, Pika's mailing list is available `on Google Groups <https://groups.google.com/forum/?fromgroups#!forum/pika-python>`_ and the email address is pika-python@googlegroups.com, though traditionally questions about Pika have been asked on the `RabbitMQ-Discuss mailing list <http://lists.rabbitmq.com/cgi-bin/mailman/listinfo/rabbitmq-discuss>`_.

    有的，在`谷歌论坛 <https://groups.google.com/forum/?fromgroups#!forum/pika-python>`上我们有一个活跃的邮件列表，地址是 pika-python@googlegroups.com,一些常见的关于Pika的问题已经在 `RabbitMQ-Discuss mailing list <http://lists.rabbitmq.com/cgi-bin/mailman/listinfo/rabbitmq-discuss>`_ 上讨论过。

- How can I contribute to Pika?

- 我怎样完善Pika？

    You can `fork the project on Github <http://help.github.com/forking/>`_ and issue `Pull Requests <http://help.github.com/pull-requests/>`_ when you believe you have something solid to be added to the main repository.

    你可以在`Github fork Pika <http://help.github.com/forking/>`_  然后 `Pull Requests <http://help.github.com/pull-requests/>`_ , 当你相信你已经解决了一个问题的时候，你可以发起一个合并请求将你的更改合并到主分支。