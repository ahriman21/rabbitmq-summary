# rabbitmq intro
RabbitMQ is a message broker: it accepts and forwards messages. You can think about it as a post office.RabbitMQ, and messaging in general, uses some jargon.
producing : Producing means nothing more than sending. A program that sends messages is a producer .
queue : queue stores the messages.
consumer : consumer recieves messages from producer through queue.

# connectetion
 The first thing we need to do is to establish a connection with RabbitMQ server.
```python
import pika
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
```

# create a queue
> to create a queue and send messages through it, we need to create a channel first
```python
channel = connection.channel()
channel.queue_declare(queue='q1')
```

# send a message 
> notice that a message always needs to go through an exchange in order to store in a queue.
> when you fill `exchange` attribute with empty strings it will set it as default value which is `direct`.  
> notice that after sending message you need to close the connection.  
```python
# sender.py

channel.basic_publish(exchange='',
                      routing_key='q1',
                      body='Hello World!')
connection.close()
```

# recieve a message
to recieve a message you must implement the connection and also declare the same queue to check if there is a queue to recieve or not.  
then you need a piece of code to configure recieving the message . for example you should set the function that is called when a message recieved.
```python
# reciever.py

def callback(ch, method, properties, body):
    print(f" [x] Received {body}")

channel.basic_consume(queue='q1', # the queue we created .
                      auto_ack=True, # 
                      on_message_callback=callback) # the callback function that will be executed if a message recieves.
channel.start_consuming()

```

# durability 
to make messages and queue durable, you can add attributes while creating queue and sending messages.it means if server restarted or something like that, messages wouldn't lost.  
```python
# sender.py

# make queue durable ==> durable=True
channel.queue_declare(queue='q1', durable=True)

# make messages durable ==> 'properties=pika.BasicProperties(delivery_mode=2,)' 2=persistent
channel.basic_publish(...,
                       properties=pika.BasicProperties(delivery_mode=2,)
                     ,...)

```

# Message acknowledgment
when consumer gets a message, it will send an acknowledgment to producer to delete the message from queue.  
> if you want to do this automatically you need to set `auto_ack=True`.  
```python

channel.basic_consume(...,
                      auto_ack=True,
                      ...)
```

> if you don't wanna send acknowledgment automatically you need to put code below in your callback function:
```python
def callback(ch, method, properties,body):
    ...
    ch.basic_ack(delivery_tag = method.delivery_tag)
    ...

```


# to configure your workers(consumers) in a way that each resieve messages 'one by one' and after the task done, you can put code below before basic_consume method :
```python
channel.basic_qos(prefetch=1)
```

