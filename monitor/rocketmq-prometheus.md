# 基于 RocketMQ Prometheus Exporter 的监控

 本文将对 RocketMQ-Exporter 的设计实现做一个简单的介绍，读者可以通过本文了解到 RocketMQ-Exporter 的实现过程，以及通过 RocketMQ-Exporter 来搭建自己的 RocketMQ 监控系统。该项目的 git 地址[ https://github.com/apache/rocketmq-exporter](https://github.com/apache/rocketmq-exporter) 

 https://www.infoq.cn/article/NcSYj_2PQhBlqveuD1Kw 



### 监控指标

| 监控指标                                                     | 含义                                 |
| :----------------------------------------------------------- | :----------------------------------- |
| rocketmq_broker_tps                                          | broker 每秒生产消息数量              |
| rocketmq_broker_qps                                          | broker 每秒消费消息数量              |
| rocketmq_producer_tps                                        | 某个 topic 每秒生产的消息数量        |
| rocketmq_producer_put_size                                   | 某个 topic 每秒生产的消息大小 (字节) |
| rocketmq_producer_offset                                     | 某个 topic 的生产消息的进度          |
| rocketmq_consumer_tps                                        | 某个消费组每秒消费的消息数量         |
| rocketmq_consumer_get_size                                   | 某个消费组每秒消费的消息大小 (字节)  |
| rocketmq_consumer_offset                                     | 某个消费组的消费消息的进度           |
| rocketmq_group_get_latency_by_storetime                      | 某个消费组的消费延时时间             |
| rocketmq_message_accumulation（rocketmq_producer_offset-rocketmq_consumer_offset） | 消息堆积量（生产进度 - 消费进度）    |



### 告警指标

| 告警指标                                       | 含义              |
| :--------------------------------------------- | :---------------- |
| sum(rocketmq_producer_tps) by (cluster) >= 10  | 集群发送 tps 太高 |
| sum(rocketmq_producer_tps) by (cluster) < 1    | 集群发送 tps 太低 |
| sum(rocketmq_consumer_tps) by (cluster) >= 10  | 集群消费 tps 太高 |
| sum(rocketmq_consumer_tps) by (cluster) < 1    | 集群消费 tps 太低 |
| rocketmq_group_get_latency_by_storetime > 1000 | 集群消费延时告警  |
| rocketmq_message_accumulation > value          | 消费堆积告警      |



### rules

```yaml
###
# Sample prometheus rules/alerts for rocketmq.
#
###
# Galera Alerts
 
groups:
- name: GaleraAlerts
  rules:
  - alert: RocketMQClusterProduceHigh
    expr: sum(rocketmq_producer_tps) by (cluster) >= 10
    for: 3m
    labels:
      severity: warning
    annotations:
      description: '{{$labels.cluster}} Sending tps too high.'
      summary: cluster send tps too high
  - alert: RocketMQClusterProduceLow
    expr: sum(rocketmq_producer_tps) by (cluster) < 1
    for: 3m
    labels:
      severity: warning
    annotations:
      description: '{{$labels.cluster}} Sending tps too low.'
      summary: cluster send tps too low
  - alert: RocketMQClusterConsumeHigh
    expr: sum(rocketmq_consumer_tps) by (cluster) >= 10
    for: 3m
    labels:
      severity: warning
    annotations:
      description: '{{$labels.cluster}} consuming tps too high.'
      summary: cluster consume tps too high
  - alert: RocketMQClusterConsumeLow
    expr: sum(rocketmq_consumer_tps) by (cluster) < 1
    for: 3m
    labels:
      severity: warning
    annotations:
      description: '{{$labels.cluster}} consuming tps too low.'
      summary: cluster consume tps too low
  - alert: ConsumerFallingBehind
    expr: (sum(rocketmq_producer_offset) by (topic) - on(topic)  group_right  sum(rocketmq_consumer_offset) by (group,topic)) - ignoring(group) group_left sum (avg_over_time(rocketmq_producer_tps[5m])) by (topic)*5*60 > 0
    for: 3m
    labels:
      severity: warning
    annotations:
      description: 'consumer {{$labels.group}} on {{$labels.topic}} lag behind
        and is falling behind (behind value {{$value}}).'
      summary: consumer lag behind
  - alert: GroupGetLatencyByStoretime
    expr: rocketmq_group_get_latency_by_storetime > 1000
    for: 3m
    labels:
      severity: warning
    annotations:
      description: 'consumer {{$labels.group}} on {{$labels.broker}}, {{$labels.topic}} consume time lag behind message store time
        and (behind value is {{$value}}).'
      summary: message consumes time lag behind message store time too much 
```

