Kafka备忘

# 关键词

两种模式：
- 点对点（P2P）：生产消费一对一
- 发布订阅（Pub/Sub）：生产消费一对多广播

- producer
    - acks: 多少个ISR中的副本写入才认为消息写入成功，0 - 不需要确认， 1 - leader写入即可， all 全部副本都写入（但当ISR中只有leader时仍然可能丢消息，需要配合）。
- consumer
    - consumer group:
        - 每一个分区只能被一个消费者组的一个消费者消费
        - 可以通过增加消费者组中的消费者数量来提高并发量，但是消费者数量不能大于分区数量，否则就会有消费者分配不到任何分区
- broker
- zookeeper

- topic
- partition
- replica
    - Leader
    - AR(Assigned Replicas): 分区中的所有副本
    - ISR(In-Sync Replicas)：同步中的副本，仍然可能会有一定程度的滞后
    - OSR(Out-Sync Replicas): 滞后过多的副本

- HW: High Watermark
- LEO: Log End Offset LEO是Log End Offset的缩写，它标识当前日志文件中下一条待写入消息的offset，LEO的大小相当于当前日志分区中最后一条消息的offset值加1。分区ISR集合中的每个副本都会维护自身的LEO，而ISR集合中最小的LEO即为分区的HW，对消费者而言只能消费HW之前的消息

- 再均衡


# FAQ

## 消息发送消费流程

```
producer.send() -> producer -> producer interceptor -> producer serializer 

-> consumer.poll() -> consumer deserializer -> consumer interceptor -> consume
```

## 如何保证消息不丢

## 如何避免重复消费

## 如何实现延迟消息

## 如何实现死信队列