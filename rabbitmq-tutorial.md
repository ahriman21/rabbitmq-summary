# rabbitmq intro
RabbitMQ is a message broker: it accepts and forwards messages. You can think about it as a post office.RabbitMQ, and messaging in general, uses some jargon.
A producer is a user application that sends messages.
A queue is a buffer that stores messages.
A consumer is a user application that receives messages.

## connectetion
 The first thing we need to do is to establish a connection with RabbitMQ server.
 > to work with rabbitmq in python you can use pika library.  
```python
import pika
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
```

## create a queue
> to create a queue and send messages through it, we need to create a channel first
```python
channel = connection.channel()
channel.queue_declare(queue='q1')
```

## send a message 
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

## recieve a message
to recieve a message you must implement the connection and also declare the same queue to check if there is a queue to recieve or not.  
then you need a piece of code to configure recieving the message . for example you should set the function that is called when a message recieved.
```python
# reciever.py

def callback(ch, method, properties, body):
    print(f" [x] Received {body}")

channel.basic_consume(queue='q1', # the queue we created .
                      auto_ack=True, # this is a auto ack message. it means when a msg reached to consumer, remove the msg from queue.
                      on_message_callback=callback) # the callback function that will be executed if a message recieves.
channel.start_consuming()

```

## durability 
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

## Message acknowledgment
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

> !!note : it is better to use manual ack. because in auto ack method, it will not check if the message is processed or not, it only checks if the message is recieved and then removes the message from the queue. in the manual ack method we can check if message is recieved and processed, then we remove the message.  

## to configure your workers(consumers) in a way that each resieve messages 'one by one' and after the task done, you can put code below before basic_consume method :
```python
channel.basic_qos(prefetch=1)
```

## exchange
The core idea in the messaging model in RabbitMQ is that the producer never sends any messages directly to a queue.  
Instead, the producer can only send messages to an exchange. An exchange is a very simple thing. On one side it receives messages from producers and on the other side it pushes them to queues.  
There are a few exchange types available: `direct`, `topic`, `headers` and `fanout`.

in `basic_publish` method, the exchange parameter is the name of the exchange. The empty string denotes the default or nameless exchange which is a `direct` exchange. there is another parameter named `exchange_type` for defining the type of exchange.

#### funout
* it just broadcasts all the messages it receives to all the queues it knows.
* when we have a exchange with the type `funout`, we don't need to define the `routing_key` in `basic_publish` method.
* when we are using fanout type, we declare it in both producer and consumer app, but declaring queue and bindings are implemented only in consumer file

> !!! note: Compared to the fanout exchange, the direct exchange allows some filtering based on the message's routing key to determine which queue(s) receive(s) the message. With a fanout exchange, there is no such filtering and all messages go to all bound queues.  
## temporary queue
 Giving a queue a name is important when you want to share the queue between producers and consumers.

But that's not the case for our logger. We want to hear about all log messages, not just a subset of them. We're also interested only in currently flowing messages not in the old ones. To solve that we need two things.

Firstly, to create a tem queue we declare a queue like we always do, in this for a temp queue we can create a random name for it. the server can do it for us by supplying empty `queue` parameter to `queue_declare` method.
```python
q_name = channel.queue_declare(queue='')
print(q_name.method.queue) #output --> amq.gen-JzTY20BRgKO-HjmUJj0wLg
```
Secondly, once the consumer connection is closed, the queue should be deleted. There's an `exclusive` parameter for that whle declaring a queue:  
```python
result = channel.queue_declare(queue='', exclusive=True)

```

## binding 
We've already created a fanout exchange and a queue. Now we need to tell the exchange to send messages to our queue. That relationship between exchange and a queue is called a binding.
```python
channel.queue_bind(exchange='logs', # the name we chose for our exchnage
                   queue=q_name.method.queue) # the name of our random queue

```



