Asynchronous publisher example
==============================
异步的生产者例子
==============================

The following example implements a publisher that will respond to RPC commands sent from RabbitMQ and uses delivery confirmations. It will reconnect if RabbitMQ closes the connection and will shutdown if RabbitMQ closes the channel. While it may look intimidating, each method is very short and represents a individual actions that a publisher can do.

这个例子实现了一个生产者例子 RabbitMQ响应接收到的RPC命令、生产者进行接收确认。如果RabbitMQ关闭了连接应用程序将退出、如果RabbitMQ关闭了信道他将会重新连接。看起来代码非常多，但是每个方法都非常短 都代表一个生产者独立的行为。
publisher.py::

    # -*- coding: utf-8 -*-

    import logging
    import pika
    import json

    LOG_FORMAT = ('%(levelname) -10s %(asctime)s %(name) -30s %(funcName) '
                    '-35s %(lineno) -5d: %(message)s')
    LOGGER = logging.getLogger(__name__)


    class ExamplePublisher(object):
        """This is an example publisher that will handle unexpected interactions
        with RabbitMQ such as channel and connection closures.

        这个生产者例子将和RabbitMQ交互并处理异常像信道和连接的关闭。

        If RabbitMQ closes the connection, it will reopen it. You should
        look at the output, as there are limited reasons why the connection may
        be closed, which usually are tied to permission related issues or
        socket timeouts.

        如果RabbitMQ的连接关闭，将会重新打开它。在日志输出中可以看到原因 连接为什么关闭，这通常是权限问题或者是socket超时。

        It uses delivery confirmations and illustrates one way to keep track of
        messages that have been sent and if they've been confirmed by RabbitMQ.

        如果消息被发送到RabbitMQ，它将用消息确认或者illustrates来保证消息被正常接收。

        """
        EXCHANGE = 'message'
        EXCHANGE_TYPE = 'topic'
        PUBLISH_INTERVAL = 1
        QUEUE = 'text'
        ROUTING_KEY = 'example.text'

        def __init__(self, amqp_url):
            """Setup the example publisher object, passing in the URL we will use
            to connect to RabbitMQ.

            初始化一个生产者的对象，通过URL连接到RabbitMQ消息服务器。

            :param str amqp_url: The URL for connecting to RabbitMQ

            """
            self._connection = None
            self._channel = None

            self._deliveries = None
            self._acked = None
            self._nacked = None
            self._message_number = None

            self._stopping = False
            self._url = amqp_url

        def connect(self):
            """This method connects to RabbitMQ, returning the connection handle.
            When the connection is established, the on_connection_open method
            will be invoked by pika. If you want the reconnection to work, make
            sure you set stop_ioloop_on_close to False, which is not the default
            behavior of this adapter.

            这个方法连接到RabbitMQ服务器，返回一个连接句柄。当连接建立的时候on_connection_open方法将要被Pika调用，当连接关闭的时候on_connection_closed方法将要被调用。如果想要连接工作，请确保设置stop_ioloop_on_close为False，这不是适配器的默认值。

            :rtype: pika.SelectConnection

            """
            LOGGER.info('Connecting to %s', self._url)
            return pika.SelectConnection(pika.URLParameters(self._url),
                                         on_open_callback=self.on_connection_open,
                                         on_close_callback=self.on_connection_closed,
                                         stop_ioloop_on_close=False)

        def on_connection_open(self, unused_connection):
            """This method is called by pika once the connection to RabbitMQ has
            been established. It passes the handle to the connection object in
            case we need it, but in this case, we'll just mark it unused.

            当Pika连接到RabbitMQ，连接已经建立的时候这个方法将要被Pika调用。这个方法接收一个连接的句柄对象，我们有可能需要它，但是在实际中，我们仅仅把它作为标志位。

            :type unused_connection: pika.SelectConnection

            """
            LOGGER.info('Connection opened')
            self.open_channel()

        def on_connection_closed(self, connection, reply_code, reply_text):
            """This method is invoked by pika when the connection to RabbitMQ is
            closed unexpectedly. Since it is unexpected, we will reconnect to
            RabbitMQ if it disconnects.

            这个方法在RabbitMQ连接被异常关闭的时候被调用。既然是意外，如果断开连接 我们将会重新连接到RabbitMQ。

            :param pika.connection.Connection connection: The closed connection obj
            :param int reply_code: The server provided reply_code if given
            :param str reply_text: The server provided reply_text if given

            """
            self._channel = None
            if self._stopping:
                self._connection.ioloop.stop()
            else:
                LOGGER.warning('Connection closed, reopening in 5 seconds: (%s) %s',
                               reply_code, reply_text)
                self._connection.add_timeout(5, self._connection.ioloop.stop)

        def open_channel(self):
            """This method will open a new channel with RabbitMQ by issuing the
            Channel.Open RPC command. When RabbitMQ confirms the channel is open
            by sending the Channel.OpenOK RPC reply, the on_channel_open method
            will be invoked.

            这个方法向RabbitMQ发出一个Channel.Open RPC命令来打开一个新的信道。当RabbitMQ确认信道已经打开会发送一个Channel.OpenOK RPC确认，这个on_channel_open方法将要被调用。

            """
            LOGGER.info('Creating a new channel')
            self._connection.channel(on_open_callback=self.on_channel_open)

        def on_channel_open(self, channel):
            """This method is invoked by pika when the channel has been opened.
            The channel object is passed in so we can make use of it.

            Since the channel is now open, we'll declare the exchange to use.

            :param pika.channel.Channel channel: The channel object

            当channel已经打开的时候这个方法将要被Pika调用。这个信道对象可以被传递所以可以使用它。由于信道是开放的，我们可以申请exchange使用。

            """
            LOGGER.info('Channel opened')
            self._channel = channel
            self.add_on_channel_close_callback()
            self.setup_exchange(self.EXCHANGE)

        def add_on_channel_close_callback(self):
            """This method tells pika to call the on_channel_closed method if
            RabbitMQ unexpectedly closes the channel.

            这个方法告诉Pika调用on_channel_closed方法，当RabbitMQ异常关闭信道的时候。

            """
            LOGGER.info('Adding channel close callback')
            self._channel.add_on_close_callback(self.on_channel_closed)

        def on_channel_closed(self, channel, reply_code, reply_text):
            """Invoked by pika when RabbitMQ unexpectedly closes the channel.
            Channels are usually closed if you attempt to do something that
            violates the protocol, such as re-declare an exchange or queue with
            different parameters. In this case, we'll close the connection
            to shutdown the object.

            当RabbitMQ异常关闭信道的时候被调用，如果你试图做一些违反协议的事情信道会被关闭，像再次申请的交换分区和队列参数不一致。在这种情况下，我们将会关闭连接来回收对象。

            :param pika.channel.Channel channel: The closed channel
            :param int reply_code: The numeric reason the channel was closed
            :param str reply_text: The text reason the channel was closed

            """
            LOGGER.warning('Channel was closed: (%s) %s', reply_code, reply_text)
            self._channel = None
            if not self._stopping:
                self._connection.close()

        def setup_exchange(self, exchange_name):
            """Setup the exchange on RabbitMQ by invoking the Exchange.Declare RPC
            command. When it is complete, the on_exchange_declareok method will
            be invoked by pika.

            调用Exchange.Declare RPC命令在RabbitMQ中设置交换分区。当完成的时候on_exchange_declareok方法会被Pika调用。

            :param str|unicode exchange_name: The name of the exchange to declare

            """
            LOGGER.info('Declaring exchange %s', exchange_name)
            self._channel.exchange_declare(self.on_exchange_declareok,
                                           exchange_name,
                                           self.EXCHANGE_TYPE)

        def on_exchange_declareok(self, unused_frame):
            """Invoked by pika when RabbitMQ has finished the Exchange.Declare RPC
            command.

            当RabbitMQ完成Exchange.Declare RPC命令的时候这个方法将要被调用。

            :param pika.Frame.Method unused_frame: Exchange.DeclareOk response frame

            """
            LOGGER.info('Exchange declared')
            self.setup_queue(self.QUEUE)

        def setup_queue(self, queue_name):
            """Setup the queue on RabbitMQ by invoking the Queue.Declare RPC
            command. When it is complete, the on_queue_declareok method will
            be invoked by pika.

            通过一个Queue.Declare RPC命令在RabbitMQ中设置队列。当它完成的时候on_queue_declareok方法将要被Pika调用。

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

            当 Queue.Declare RPC命令设置的队列已经完成的时候这个方法将要被调用。在这个方法中将会发送Queue.Bind RPC 命令通过routing key来绑定队列和exchange。当这个命令完成的时候，on_bindok方法将要被Pika调用。

            :param pika.frame.Method method_frame: The Queue.DeclareOk frame

            """
            LOGGER.info('Binding %s to %s with %s',
                        self.EXCHANGE, self.QUEUE, self.ROUTING_KEY)
            self._channel.queue_bind(self.on_bindok, self.QUEUE,
                                     self.EXCHANGE, self.ROUTING_KEY)

        def on_bindok(self, unused_frame):
            """This method is invoked by pika when it receives the Queue.BindOk
            response from RabbitMQ. Since we know we're now setup and bound, it's
            time to start publishing.

            当pika接收到RabbitMQ Queue.BindOk 响应的时候这个方法将会被调用。这个时候我们已经建立了交换分区和队列，可以开始推送消息。

            """
            LOGGER.info('Queue bound')
            self.start_publishing()

        def start_publishing(self):
            """This method will enable delivery confirmations and schedule the
            first message to be sent to RabbitMQ

            这种方法将发送传送确认并安排第一个消息发送到RabbitMQ

            """
            LOGGER.info('Issuing consumer related RPC commands')
            self.enable_delivery_confirmations()
            self.schedule_next_message()

        def enable_delivery_confirmations(self):
            """Send the Confirm.Select RPC method to RabbitMQ to enable delivery
            confirmations on the channel. The only way to turn this off is to close
            the channel and create a new one.

            发送Confirm.Select RPC命令到RabbitMQ 在channel上进行接收确认。这个方法唯一的作用是设置一个关闭的开关 当关闭信道的时候创建一个新的。

            When the message is confirmed from RabbitMQ, the
            on_delivery_confirmation method will be invoked passing in a Basic.Ack
            or Basic.Nack method from RabbitMQ that will indicate which messages it
            is confirming or rejecting.

            当消息从RabbitMQ确认的时候，on_delivery_confirmation方法将要被调用，从RabbitMQ接收一个Basic.Ack或者Basic.Nack参数 它将标识这个消息是接收还是拒绝。

            """
            LOGGER.info('Issuing Confirm.Select RPC command')
            self._channel.confirm_delivery(self.on_delivery_confirmation)

        def on_delivery_confirmation(self, method_frame):
            """Invoked by pika when RabbitMQ responds to a Basic.Publish RPC
            command, passing in either a Basic.Ack or Basic.Nack frame with
            the delivery tag of the message that was published. The delivery tag
            is an integer counter indicating the message number that was sent
            on the channel via Basic.Publish. Here we're just doing house keeping
            to keep track of stats and remove message numbers that we expect
            a delivery confirmation of from the list used to keep track of messages
            that are pending confirmation.

            当RabbitMQ响应一个Basic.Publish RPC命令的时候被调用。传递一个Basic.Ack或者Basic.Nack参数作为标签标识消息是否被接收。这个标签是一个整数计数器标识出有多少个消息通过Basic.Publish已经被发送到channel。这里我们仅仅做一个标识仓库保存统计消息，我们希望从列表的递送确认中跟踪消息的状态。

            :param pika.frame.Method method_frame: Basic.Ack or Basic.Nack frame

            """
            confirmation_type = method_frame.method.NAME.split('.')[1].lower()
            LOGGER.info('Received %s for delivery tag: %i',
                        confirmation_type,
                        method_frame.method.delivery_tag)
            if confirmation_type == 'ack':
                self._acked += 1
            elif confirmation_type == 'nack':
                self._nacked += 1
            self._deliveries.remove(method_frame.method.delivery_tag)
            LOGGER.info('Published %i messages, %i have yet to be confirmed, '
                        '%i were acked and %i were nacked',
                        self._message_number, len(self._deliveries),
                        self._acked, self._nacked)

        def schedule_next_message(self):
            """If we are not closing our connection to RabbitMQ, schedule another
            message to be delivered in PUBLISH_INTERVAL seconds.

            如果我们不关闭RabbitMQ连接，可以在间隔秒数后发送另一条消息

            """
            LOGGER.info('Scheduling next message for %0.1f seconds',
                        self.PUBLISH_INTERVAL)
            self._connection.add_timeout(self.PUBLISH_INTERVAL,
                                         self.publish_message)

        def publish_message(self):
            """If the class is not stopping, publish a message to RabbitMQ,
            appending a list of deliveries with the message number that was sent.
            This list will be used to check for delivery confirmations in the
            on_delivery_confirmations method.

            如果这个对象没有停止，推送一个消息到RabbitMQ，追加一个我们发送的消息标签到list中，这个list被用来在on_delivery_confirmations方法中检查消息确认。

            Once the message has been sent, schedule another message to be sent.
            The main reason I put scheduling in was just so you can get a good idea
            of how the process is flowing by slowing down and speeding up the
            delivery intervals by changing the PUBLISH_INTERVAL constant in the
            class.

            一旦消息已经发送，安排发送另一条消息。这么做的主要的原因是：你可以在类中设置PUBLISH_INTERVAL的值来控制进程推送消息的进度和速度。

            """
            if self._channel is None or not self._channel.is_open:
                return

            message = {u'مفتاح': u' قيمة',
                       u'键': u'值',
                       u'キー': u'値'}
            properties = pika.BasicProperties(app_id='example-publisher',
                                              content_type='application/json',
                                              headers=message)

            self._channel.basic_publish(self.EXCHANGE, self.ROUTING_KEY,
                                        json.dumps(message, ensure_ascii=False),
                                        properties)
            self._message_number += 1
            self._deliveries.append(self._message_number)
            LOGGER.info('Published message # %i', self._message_number)
            self.schedule_next_message()

        def run(self):
            """Run the example code by connecting and then starting the IOLoop.

              通过连接开始IOLoop运行示例代码。
            """
            while not self._stopping:
                self._connection = None
                self._deliveries = []
                self._acked = 0
                self._nacked = 0
                self._message_number = 0

                try:
                    self._connection = self.connect()
                    self._connection.ioloop.start()
                except KeyboardInterrupt:
                    self.stop()
                    if (self._connection is not None and
                            not self._connection.is_closed):
                        # Finish closing
                        self._connection.ioloop.start()

            LOGGER.info('Stopped')

        def stop(self):
            """Stop the example by closing the channel and connection. We
            set a flag here so that we stop scheduling new messages to be
            published. The IOLoop is started because this method is
            invoked by the Try/Catch below when KeyboardInterrupt is caught.
            Starting the IOLoop again will allow the publisher to cleanly
            disconnect from RabbitMQ.

            停止这个例子的信道和连接。我们在这里设置一个标志，让我们判断是否推送新的消息。IOLoop开始的时候，当KeyboardInterrupt异常发生的时候将会执行Try/Catch代码。再次启动IOLoop允许生产者推送消息时 应断开RabbitMQ的所有连接。
            """
            LOGGER.info('Stopping')
            self._stopping = True
            self.close_channel()
            self.close_connection()

        def close_channel(self):
            """Invoke this command to close the channel with RabbitMQ by sending
            the Channel.Close RPC command.
            当RabbitMQ发送一个 Channel.Close RPC命令的时候这个方法被调用
            """
            if self._channel is not None:
                LOGGER.info('Closing the channel')
                self._channel.close()

        def close_connection(self):
            """This method closes the connection to RabbitMQ.
            	这个方法关闭RabbitMQ的连接。
            """
            if self._connection is not None:
                LOGGER.info('Closing connection')
                self._connection.close()


    def main():
        logging.basicConfig(level=logging.DEBUG, format=LOG_FORMAT)

        # Connect to localhost:5672 as guest with the password guest and virtual host "/" (%2F)

        #以guest用户guest密码连接到 localhost:5672 地址的虚拟主机 "/" (%2F)

        example = ExamplePublisher('amqp://guest:guest@localhost:5672/%2F?connection_attempts=3&heartbeat_interval=3600')
        example.run()


    if __name__ == '__main__':
        main()
