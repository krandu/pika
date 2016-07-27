Asynchronous consumer example
=============================
异步消费者
=============================
The following example implements a consumer that will respond to RPC commands sent from RabbitMQ. For example, it will reconnect if RabbitMQ closes the connection and will shutdown if RabbitMQ cancels the consumer or closes the channel. While it may look intimidating, each method is very short and represents a individual actions that a consumer can do.

下面的例子实现消费者，它会响应从RabbitMQ服务器发送来的一个RPC命令。在例子中，如果RabbitMQ 关闭连接、RabbitMQ撤销消费者或者关闭通道程序都会退出。看起来代码非常多，但是每个方法都很短并且都表示了一个消费者可以做的一个动作。

consumer.py::

    # -*- coding: utf-8 -*-

    import logging
    import pika

    LOG_FORMAT = ('%(levelname) -10s %(asctime)s %(name) -30s %(funcName) '
                  '-35s %(lineno) -5d: %(message)s')
    LOGGER = logging.getLogger(__name__)


    class ExampleConsumer(object):
        """This is an example consumer that will handle unexpected interactions
        with RabbitMQ such as channel and connection closures.

        在这个例子中如果消费者遇到不可控的意外，RabbitMQ将会关闭缓冲区和连接。

        If RabbitMQ closes the connection, it will reopen it. You should
        look at the output, as there are limited reasons why the connection may
        be closed, which usually are tied to permission related issues or
        socket timeouts.

        如果连接关闭，它会尝试在此打开它。可以在日志中看到一行像这样的输出：与连接关闭的原因是权限问题或者socket连接超时。

        If the channel is closed, it will indicate a problem with one of the
        commands that were issued and that should surface in the output as well.

        如果输出被关闭，它将会在输出的日志前面打印问题发生时的命令。
        """
        EXCHANGE = 'message'
        EXCHANGE_TYPE = 'topic'
        QUEUE = 'text'
        ROUTING_KEY = 'example.text'

        def __init__(self, amqp_url):
            """Create a new instance of the consumer class, passing in the AMQP
            URL used to connect to RabbitMQ.
            创建消费者的一个新实例，传递AMQP地址用于连接的RabbitMQ。

            :param str amqp_url: The AMQP url to connect with

            """
            self._connection = None
            self._channel = None
            self._closing = False
            self._consumer_tag = None
            self._url = amqp_url

        def connect(self):
            """This method connects to RabbitMQ, returning the connection handle.
            When the connection is established, the on_connection_open方法将会被调用。 method
            will be invoked by pika.

            这个方法将会连接到RabbitMQ，返回一个连接对象的地址，当连接建立后on_connection_open方法将会被调用。

            :rtype: pika.SelectConnection

            """
            LOGGER.info('Connecting to %s', self._url)
            return pika.SelectConnection(pika.URLParameters(self._url),
                                         self.on_connection_open,
                                         stop_ioloop_on_close=False)

        def on_connection_open(self, unused_connection):
            """This method is called by pika once the connection to RabbitMQ has
            been established. It passes the handle to the connection object in
            case we need it, but in this case, we'll just mark it unused.

            这个方法只在连接建立的时候调用一次。传入了一个连接对象 以防我们使用 但是在这个例子中，只是把它作为了一个标记。

            :type unused_connection: pika.SelectConnection

            """
            LOGGER.info('Connection opened')
            self.add_on_connection_close_callback()
            self.open_channel()

        def add_on_connection_close_callback(self):
            """This method adds an on close callback that will be invoked by pika
            when RabbitMQ closes the connection to the publisher unexpectedly.

            这个方法增加一个在连接被意外关闭时候的调用。（先在第一次连接的时候调用 然后在这个里面注册意外关闭处理方法）

            """
            LOGGER.info('Adding connection close callback')
            self._connection.add_on_close_callback(self.on_connection_closed)

        def on_connection_closed(self, connection, reply_code, reply_text):
            """This method is invoked by pika when the connection to RabbitMQ is
            closed unexpectedly. Since it is unexpected, we will reconnect to
            RabbitMQ if it disconnects.

            此方法在RabbitMQ连接意外关闭时由Pika调用。因为是意外，我们将重新连接到
            RabbitMQ，如果它断开

            :param pika.connection.Connection connection: The closed connection obj
            :param int reply_code: The server provided reply_code if given 服务器的回复代码
            :param str reply_text: The server provided reply_text if given
            服务器的回复文本

            """
            self._channel = None
            if self._closing:
                self._connection.ioloop.stop()
            else:
                LOGGER.warning('Connection closed, reopening in 5 seconds: (%s) %s',
                               reply_code, reply_text)
                self._connection.add_timeout(5, self.reconnect)

        def reconnect(self):
            """Will be invoked by the IOLoop timer if the connection is
            closed. See the on_connection_closed method.
            如果连接被关闭后，经过超时时间的话 将会被调用 可以看上一个方法

            """
            # This is the old connection IOLoop instance, stop its ioloop
            # 处理上一个IOLoop 如果上一个连接没有被关闭（可以重用）  则停止上一个的IOLoop
            self._connection.ioloop.stop()

            if not self._closing:
                #如果连接被关闭 创建一个新的连接
                # Create a new connection
                self._connection = self.connect()

                # There is now a new connection, needs a new ioloop to run
                #这个新的连接需要一个新的ioloop
                self._connection.ioloop.start()

        def open_channel(self):
            """Open a new channel with RabbitMQ by issuing the Channel.Open RPC
            command. When RabbitMQ responds that the channel is open, the
            on_channel_open callback will be invoked by pika.

            发出一个Channel.Open命令和RabbitMQ之间打开一个通道。当RabbitMQ响应通道已经打开的时候传递的回调被调用。

            """
            LOGGER.info('Creating a new channel')
            self._connection.channel(on_open_callback=self.on_channel_open)

        def on_channel_open(self, channel):
            """This method is invoked by pika when the channel has been opened.
            The channel object is passed in so we can make use of it.
            Since the channel is now open, we'll declare the exchange to use.

            这个方法会在通道已经被打开的时候调用，这时候可依申请交换分区来使用。

            :param pika.channel.Channel channel: The channel object

            """
            LOGGER.info('Channel opened')
            self._channel = channel
            #注册信道被关闭时的调用方法
            self.add_on_channel_close_callback()
            self.setup_exchange(self.EXCHANGE)

        def add_on_channel_close_callback(self):
            """This method tells pika to call the on_channel_closed method if
            RabbitMQ unexpectedly closes the channel.
            如果RabbitMQ信道被意外关闭，给Pika增加一个调用方法
            """
            LOGGER.info('Adding channel close callback')
            self._channel.add_on_close_callback(self.on_channel_closed)

        def on_channel_closed(self, channel, reply_code, reply_text):
            """Invoked by pika when RabbitMQ unexpectedly closes the channel.
            Channels are usually closed if you attempt to do something that
            violates the protocol, such as re-declare an exchange or queue with
            different parameters. In this case, we'll close the connection
            to shutdown the object.

            当信道被关闭的时候Pika会调用这个方法，关闭连接（现在回调用打开连接时注册的方法）。当你试图做一些违反协议的事时 信道将会被关闭 ：如生命的队列和交换分区是不同的参数。在这种情况下，竟会关闭对象里面的连接。
            :param pika.channel.Channel: The closed channel
            :param int reply_code: The numeric reason the channel was closed
            :param str reply_text: The text reason the channel was closed

            """
            LOGGER.warning('Channel %i was closed: (%s) %s',
                           channel, reply_code, reply_text)
            #递归调用 精妙绝伦
            self._connection.close()

        def setup_exchange(self, exchange_name):
            """Setup the exchange on RabbitMQ by invoking the Exchange.Declare RPC
            command. When it is complete, the on_exchange_declareok method will
            be invoked by pika.

            发送RPC命令 申请交换分区的时候调用，如果申请成功on_exchange_declareok 将会调用。
            :param str|unicode exchange_name: The name of the exchange to declare

            """
            LOGGER.info('Declaring exchange %s', exchange_name)
            self._channel.exchange_declare(self.on_exchange_declareok,
                                           exchange_name,
                                           self.EXCHANGE_TYPE)

        def on_exchange_declareok(self, unused_frame):
            """Invoked by pika when RabbitMQ has finished the Exchange.Declare RPC
            command.
            当RabbitMQ完成Exchange.Declare RPC命令的时候调用
            :param pika.Frame.Method unused_frame: Exchange.DeclareOk response frame

            """
            LOGGER.info('Exchange declared')
            self.setup_queue(self.QUEUE)

        def setup_queue(self, queue_name):
            """Setup the queue on RabbitMQ by invoking the Queue.Declare RPC
            command. When it is complete, the on_queue_declareok method will
            be invoked by pika.

            发送队列初始化命令的时候调用，如果初始化成功调用on_queue_declareok

            :param str|unicode queue_name: The name of the queue to declare.

            """
            LOGGER.info('Declaring queue %s', queue_name)
            self._channel.queue_declare(self.on_queue_declareok, queue_name)

        def on_queue_declareok(self, method_frame):
            """Method invoked by pika when the Queue.Declare RPC call made in
            setup_queue has completed. In this method we will bind the queue
            and exchange together with the routing key by issuing the Queue.Bind
            RPC command. When this command is complete, the on_bindok method will
            be invoked by pika.

            当RabbitMQ完成了 Queue.Declare RPC命令的时候将会被调用。在这个方法中通过Queue.Bind
            RPC命令根据路由键绑定交换分区和队列。当这个命令完成的时候，on_bindok方法将会被调用。

            :param pika.frame.Method method_frame: The Queue.DeclareOk frame

            """
            LOGGER.info('Binding %s to %s with %s',
                        self.EXCHANGE, self.QUEUE, self.ROUTING_KEY)
            self._channel.queue_bind(self.on_bindok, self.QUEUE,
                                     self.EXCHANGE, self.ROUTING_KEY)

        def on_bindok(self, unused_frame):
            """Invoked by pika when the Queue.Bind method has completed. At this
            point we will start consuming messages by calling start_consuming
            which will invoke the needed RPC commands to start the process.

            当完成Queue.Bind命令的时候被调用。在这里将会开始消费者调用start_consuming，同时这也将会调用RPC命令来启用一个新的进程。

            :param pika.frame.Method unused_frame: The Queue.BindOk response frame

            """
            LOGGER.info('Queue bound')
            self.start_consuming()

        def start_consuming(self):
            """This method sets up the consumer by first calling
            add_on_cancel_callback so that the object is notified if RabbitMQ
            cancels the consumer. It then issues the Basic.Consume RPC command
            which returns the consumer tag that is used to uniquely identify the
            consumer with RabbitMQ. We keep the value to use it when we want to
            cancel consuming. The on_message method is passed in as a callback pika
            will invoke when a message is fully received.

            这个方法设置了消费者，先调用add_on_cancel_callback 这样当RabbitMQ取消消费者的时候用关闭连接这个状态来通知对象消费者被取消。
            然后发出Basic.Consume RPC命令 RabbitMQ返回一个唯一标识一个消费者的标签。保存这个值，当我们想取消消费者的时候。on_message方法作为回调来处理标准消息。
            """
            LOGGER.info('Issuing consumer related RPC commands')
            self.add_on_cancel_callback()
            self._consumer_tag = self._channel.basic_consume(self.on_message,
                                                             self.QUEUE)

        def add_on_cancel_callback(self):
            """Add a callback that will be invoked if RabbitMQ cancels the consumer
            for some reason. If RabbitMQ does cancel the consumer,
            on_consumer_cancelled will be invoked by pika.
            注册一个回调，当RabbitMQ因为某些原因取消消费者时调用on_consumer_cancelled

            """
            LOGGER.info('Adding consumer cancellation callback')
            self._channel.add_on_cancel_callback(self.on_consumer_cancelled)

        def on_consumer_cancelled(self, method_frame):
            """Invoked by pika when RabbitMQ sends a Basic.Cancel时将会被调用 for a consumer
            receiving messages.
            当消费者接收到Basic.Cancel时将会被调用
            :param pika.frame.Method method_frame: The Basic.Cancel frame

            """
            LOGGER.info('Consumer was cancelled remotely, shutting down: %r',
                        method_frame)
            if self._channel:
                self._channel.close()

        def on_message(self, unused_channel, basic_deliver, properties, body):
            """Invoked by pika when a message is delivered from RabbitMQ. The
            channel is passed for your convenience. The basic_deliver object that
            is passed in carries the exchange, routing key, delivery tag and
            a redelivered flag for the message. The properties passed in is an
            instance of BasicProperties with the message properties and the body
            is the message that was sent.

            当接收到RabbitMQ消息的时候被调用，channel是为了需要的时候使用。basic_deliver保存了交换分区 路由 数据标签 重写传递消息的标志。properties是一个实例保存了消息的类型 body就是我们发送的消息。

            :param pika.channel.Channel unused_channel: The channel object
            :param pika.Spec.Basic.Deliver: basic_deliver method
            :param pika.Spec.BasicProperties: properties
            :param str|unicode body: The message body

            """
            LOGGER.info('Received message # %s from %s: %s',
                        basic_deliver.delivery_tag, properties.app_id, body)
            self.acknowledge_message(basic_deliver.delivery_tag)

        def acknowledge_message(self, delivery_tag):
            """Acknowledge the message delivery from RabbitMQ by sending a
            Basic.Ack RPC method for the delivery tag.
            用Basic.Ack RPC命令传递delivery tag来告诉RabbitMQ已经正确的接收到消息。
   
            :param int delivery_tag: The delivery tag from the Basic.Deliver frame

            """
            LOGGER.info('Acknowledging message %s', delivery_tag)
            self._channel.basic_ack(delivery_tag)

        def stop_consuming(self):
            """Tell RabbitMQ that you would like to stop consuming by sending the
            Basic.Cancel RPC command.
            发送Basic.Cancel RPC命令告诉RabbitMQ想停止消费者
            """
            if self._channel:
                LOGGER.info('Sending a Basic.Cancel RPC command to RabbitMQ')
                self._channel.basic_cancel(self.on_cancelok, self._consumer_tag)

        def on_cancelok(self, unused_frame):
            """This method is invoked by pika when RabbitMQ acknowledges the
            cancellation of a consumer. At this point we will close the channel.
            This will invoke the on_channel_closed method once the channel has been
            closed, which will in-turn close the connection.

            当RabbitMQ发送取消的确认消息时将会被调用。这个操作将会调用close_channel关闭信道，同时转关闭连接。

            :param pika.frame.Method unused_frame: The Basic.CancelOk frame

            """
            LOGGER.info('RabbitMQ acknowledged the cancellation of the consumer')
            self.close_channel()

        def close_channel(self):
            """Call to close the channel with RabbitMQ cleanly by issuing the
            Channel.Close RPC command.

            通过发出Channel.Close RPC命令优雅的关闭信道。

            """
            LOGGER.info('Closing the channel')
            self._channel.close()

        def run(self):
            """Run the example consumer by connecting to RabbitMQ and then
            starting the IOLoop to block and allow the SelectConnection to operate.
            block、开始 SelectConnection操作，运行example消费者连接到RabbitMQ然后开始IOLoop循环监听。
            """
            self._connection = self.connect()
            self._connection.ioloop.start()

        def stop(self):
            """Cleanly shutdown the connection to RabbitMQ by stopping the consumer
            with RabbitMQ. When RabbitMQ confirms the cancellation, on_cancelok
            will be invoked by pika, which will then closing the channel and
            connection. The IOLoop is started again because this method is invoked
            when CTRL-C is pressed raising a KeyboardInterrupt exception. This
            exception stops the IOLoop which needs to be running for pika to
            communicate with RabbitMQ. All of the commands issued prior to starting
            the IOLoop will be buffered but not processed.

            清除关闭到RabbitMQ的连接停止消费者和RabbitMQ的通信。当RabbitMQ发回一个确认消息的时候on_cancelok将会被调用。当CTRL-C触发的时候将会引发一个KeyboardInterrupt异常 IOLoop将会再次调用。 这个
            异常停止IOLoop，所有在之前发出的命令将在IOLoop里被缓冲但不处理，所以需要再次            运行self._connection.ioloop.start()维持通信  处理以前的命令。

            """
            LOGGER.info('Stopping')
            self._closing = True
            self.stop_consuming()
            self._connection.ioloop.start()
            LOGGER.info('Stopped')

        def close_connection(self):
            """This method closes the connection to RabbitMQ.
                关闭和RabbitMQ的链接
            """
            LOGGER.info('Closing connection')
            self._connection.close()


    def main():
        logging.basicConfig(level=logging.INFO, format=LOG_FORMAT)
        example = ExampleConsumer('amqp://guest:guest@localhost:5672/%2F')
        try:
            example.run()
        except KeyboardInterrupt:
            example.stop()


    if __name__ == '__main__':
        main()

