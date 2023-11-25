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

```


