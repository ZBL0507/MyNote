## MQ的作用 
- 解耦
- 异步通信
- 流量消峰

<br>

## MQ如何保证消息不丢失

mq原则 数据不能多，也不能少，不能多是说消息不能重复消费，不能少，就是说不能丢失数据。
如果mq传递的是非常核心的消息，支撑核心的业务，那么这种场景是一定不能丢失数据的。

### 丢失数据场景
丢数据一般分为两种，一种是mq把消息丢了，一种就是消费时将消息丢了。
+ rabbitmq:
  <br>
  - 生产者弄丢了数据 <br>
    生产者将数据发送到rabbitmq的时候，可能在传输过程中因为网络等问题而将数据弄丢了。 <br>
    解决方案： <br>
      - 同步模式： 采用rabbitmq的事物机制，同步的，生产者发送一个消息会同步阻塞卡住等待你是成功还是失败
      - 异步模式： 相应的方法重写  消息从producer到exchange会回调一个confirmCallback，在回调方法中做相应的处理
                       相应的方法重写  消息从exchange到queue投递失败会回调一个returnCallback，在回调方法中做相应的处理
        <br>
        
  - rabbitmq自己丢了数据 <br>
    如果没有开启rabbitmq的持久化，那么rabbitmq一旦重启，那么数据就丢了。
    所依必须开启持久化将消息持久化到磁盘，这样就算rabbitmq挂了，恢复之后会自动读取之前存储的数据，一般数据不会丢失。
    除非极其罕见的情况，rabbitmq还没来得及持久化自己就挂了，这样可能导致一部分数据丢失。
    <br>
  - 消费端弄丢了数据 <br>
    主要是因为消费者消费时，刚消费到，还没有处理，结果消费者就挂了，这样你重启之后，rabbitmq就认为你已经消费过了，然后就丢了数据。
    <br>
    解决方案： 将autoAck关闭，使用手动确认机制，消费者消息处理完成之后自己给rabbitmq发送一个ack



## 如何防止MQ消息重复消费
1. 可在内存中维护一个set，只要从消息队列里面获取到一个消息，先查询这个消息在不在set里面，
如果在表示已消费过，直接丢弃；如果不在，则在消费后将其加入set当中。
2. 如果要写数据库，可以拿唯一键(如果有)先去数据库查询一下，如果不存在在写，如果存在直接更新或者丢弃消息。
3. 如果是写redis那没有问题，每次都是set，天然的幂等性。
4. 让生产者发送消息时，每条消息加一个全局的唯一id，然后消费时，将该id保存到redis里面。消费时先去redis里面查一下有么有，没有再消费。
5. 数据库操作可以设置唯一键，防止重复数据的插入，这样插入只会报错而不会插入重复数据。


