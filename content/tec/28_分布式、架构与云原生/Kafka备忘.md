Kafka备忘

# 关键词

核心组件：
- producer
- consumer
- broker
- zookeeper

- partition
- replica
    - Leader
    - AR(Assigned Replicas): 分区中的所有副本
    - ISR(In-Sync Replicas)：同步中的副本，仍然可能会有一定程度的滞后
    - OSR(Out-Sync Replicas): 滞后过多的副本

- HW: High Watermark
- LEO: Log End Offset LEO是Log End Offset的缩写，它标识当前日志文件中下一条待写入消息的offset，LEO的大小相当于当前日志分区中最后一条消息的offset值加1。分区ISR集合中的每个副本都会维护自身的LEO，而ISR集合中最小的LEO即为分区的HW，对消费者而言只能消费HW之前的消息


# FAQ

## 如何实现延迟消息

## 如何实现死信队列