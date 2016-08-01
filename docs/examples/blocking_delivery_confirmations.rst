Using Delivery Confirmations with the BlockingConnection
========================================================
在BlockingConnection中使用消息确认
========================================================

The following code demonstrates how to turn on delivery confirmations with the BlockingConnection and how to check for confirmation from RabbitMQ::

下面的实例代码演示了怎样在BlockingConnection中使用消息确认怎样检查从RabbitMQ发送来的消息。

    import pika

    # Open a connection to RabbitMQ on localhost using all default parameters
    #用默认的方式连接到本机的RabbitMQ
    connection = pika.BlockingConnection()

    # Open the channel
    #打开信道
    channel = connection.channel()

    # Declare the queue
    #申请队列
    channel.queue_declare(queue="test", durable=True, exclusive=False, auto_delete=False)

    # Turn on delivery confirmations
    #打开消息确认
    channel.confirm_delivery()

    # Send a message
    if channel.basic_publish(exchange='test',
                             routing_key='test',
                             body='Hello World!',
                             properties=pika.BasicProperties(content_type='text/plain',
                                                             delivery_mode=1)):
        print('Message publish was confirmed')
    else:
        print('Message could not be confirmed')
