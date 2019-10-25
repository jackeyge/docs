# zookeeper 监控

### 四字命令

最简单的监控方式就是使用ZooKeeper的四字命令，你可以直接通过telnet或者nc命令查看状态。

![img](https://mmbiz.qpic.cn/mmbiz_png/rtibSseGoBickhibcyRW3sJUIzibKQQQhgs4VWlXABVLuKeBYzhuxSibfwxFF0QAdqTUrZIibic1aicDw8vkRHKQjaiaslQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

四字命令



![img](https://mmbiz.qpic.cn/mmbiz_png/rtibSseGoBickhibcyRW3sJUIzibKQQQhgs4kV3N7Hqh2lnNicUEPC77gDPN2toBTYjpf4JHDuo4E8o7uwBudpGIUJw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

四字命令



![img](https://mmbiz.qpic.cn/mmbiz_png/rtibSseGoBickhibcyRW3sJUIzibKQQQhgs40a22br1IRg4A49UjSqxBc1CmmdrjQQc5sicsFiaiaDd085VB5FFsnB9TA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

四字命令

常用的四字命令如下：

| ZooKeeper四字命令 | 功能描述                                                     |
| ----------------- | ------------------------------------------------------------ |
| conf              | 打印配置                                                     |
| **cons**          | 列出所有连接到这台服务器的客户端全部连接/会话详细信息。包括"接受/发送"的包数量、会话id、操作延迟、最后的操作执行等等信息。 |
| crst              | 重置所有连接的连接和会话统计信息。                           |
| dump              | 列出那些比较重要的会话和临时节点。这个命令只能在leader节点上有用。 |
| envi              | 打印出服务环境的详细信息。                                   |
| reqs              | 列出未经处理的请求                                           |
| ruok              | 即"Are you ok"，测试服务是否处于正确状态。如果确实如此，那么服务返回"imok"，否则不做任何相应。 |
| stat              | 输出关于性能和连接的客户端的列表。                           |
| srst              | 重置服务器的统计。                                           |
| srvr              | 列出连接服务器的详细信息                                     |
| wchs              | 列出服务器watch的详细信息。                                  |
| wchc              | 通过session列出服务器watch的详细信息，它的输出是一个与watch相关的会话的列表。 |
| wchp              | 通过路径列出服务器watch的详细信息。它输出一个与session相关的路径。 |
| **mntr**          | 输出可用于检测集群健康状态的变量列表                         |

### 如何使用四字命令？

可以在客户端可以通过 telnet 或 nc 向 ZooKeeper 提交相应的命令。举个最常用的栗子：

```
echo mntr | nc ip 2181
```

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





[zabbix 监控zookeeper篇]: https://www.cnblogs.com/yxy-linux/p/8023660.html

