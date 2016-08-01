Ensuring message delivery with the mandatory flag
=================================================
用消息标志确保消息正确传输
=================================================

The following example demonstrates how to check if a message is delivered by setting the mandatory flag and checking the return result when using the BlockingConnection::

下面这个例子演示在BlockingConnection:: 中如何通过设置强制性标志进行消息确认和检查返回值。

    import pika

    # Open a connection to RabbitMQ on localhost using all default parameters
    #用默认的方式连接到本机的RabbitMQ
    connection = pika.BlockingConnection()

    # Open the channel
    channel = connection.channel()

    # Declare the queue
    channel.queue_declare(queue="test", durable=True, exclusive=False, auto_delete=False)

    # Enabled delivery confirmations
    channel.confirm_delivery()

    # Send a message
    if channel.basic_publish(exchange='test',
                             routing_key='test',
                             body='Hello World!',
                             properties=pika.BasicProperties(content_type='text/plain',
                                                             delivery_mode=1),
                             mandatory=True):
        print('Message was published')
    else:
        print('Message was returned')
