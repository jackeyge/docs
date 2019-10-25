# 通过Prometheus 监控 zookeepper



### 插件

 https://github.com/jiankunking/zookeeper_exporter 

 https://blog.csdn.net/qq_25934401/article/details/84345905 

### 监控指标

| 指标名                        | 解释                 |
| ----------------------------- | -------------------- |
| zk_version                    | 版本                 |
| zk_avg_latency                | 平均 响应延迟        |
| zk_max_latency                | 最大 响应延迟        |
| zk_min_latency                | 最小 响应延迟        |
| zk_packets_received           | 收包数               |
| zk_packets_sent               | 发包数               |
| zk_num_alive_connections      | 活跃连接数           |
| zk_outstanding_requests       | 堆积请求数           |
| zk_server_state               | 主从状态             |
| zk_znode_count                | znode 数             |
| zk_watch_count                | watch 数             |
| zk_ephemerals_count           | 临时节点数           |
| zk_approximate_data_size      | 近似数据总和大小     |
| zk_open_file_descriptor_count | 打开 文件描述符 数   |
| zk_max_file_descriptor_count  | 最大 文件描述符 数   |
| **leader才有的指标**          |                      |
| zk_followers                  | Follower 数          |
| zk_synced_followers           | 已同步的 Follower 数 |
| zk_pending_syncs              | 阻塞中的 sync 操作   |



### 报警指标

- **zk_outstanding_requests** 堆积请求数
- **zk_pending_syncs** 阻塞中的 sync 操作
- **zk_avg_latency** 平均 响应延迟
- **zk_open_file_descriptor_count** 打开 文件描述符 数
- **zk_max_file_descriptor_count** 最大 文件描述符 数
- **zk_up** 1
- **zk_server_state** 主从状态
- **zk_num_alive_connections** 活跃连接数


### rule文件

```yaml
groups:
- name: zookeeperStatsAlert 
  rules:
  - alert: 堆积请求数过大
    expr: avg(zk_outstanding_requests) by (instance) > 10 
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} "
      description: "积请求数过大"
  - alert: 阻塞中的 sync 过多
    expr: avg(zk_pending_syncs) by (instance) > 10 
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} " 
      description: "塞中的 sync 过多"
  - alert: 平均响应延迟过高
    expr: avg(zk_avg_latency) by (instance) > 10
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} "
      description: '平均响应延迟过高'
  - alert: 打开文件描述符数大于系统设定的大小
    expr: zk_open_file_descriptor_count > zk_max_file_descriptor_count * 0.85
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} "
      description: '打开文件描述符数大于系统设定的大小'
  - alert: zookeeper服务器宕机
    expr: zk_up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} "
      description: 'zookeeper服务器宕机'
  - alert: zk主节点丢失
    expr: absent(zk_server_state{state="leader"})  != 1 
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} "
      description: 'zk主节点丢失'

```

