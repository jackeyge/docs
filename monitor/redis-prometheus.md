# Prometheus 监控 Redis

### 插件

 [https://github.com/oliver006/redis_exporter](https://yq.aliyun.com/go/articleRenderRedirect?url=https%3A%2F%2Fgithub.com%2Foliver006%2Fredis_exporter) 



### 指标

| 监控指标                                             | 含义                                                         |
| :--------------------------------------------------- | :----------------------------------------------------------- |
| **#Server**                                          | ＃Redis 服务器的信息                                         |
| redis_version:2.8.13                                 | ＃redis版本                                                  |
| redis_git_sha1:00000000                              |                                                              |
| redis_git_dirty:0                                    |                                                              |
| redis_build_id:ba7e0c54ae404843                      |                                                              |
| redis_mode:standalone                                | ＃redis运行模式                                              |
| os:Linux 2.6.32-504.16.2.el6.x86_64 x86_64           | ＃操作系统版本                                               |
| arch_bits:64                                         | ＃操作系统架构                                               |
| multiplexing_api:epoll                               | #Redis 所使用的事件处理机制                                  |
| gcc_version:4.4.7                                    | ＃gcc版本                                                    |
| process_id:5221                                      | ＃当前运行进程ID                                             |
| run_id:62765912921734d0b192e4c7dec5bdeb92cf5af7      | ＃ Redis 服务器的随机标识符（用于 Sentinel 和集群）          |
| tcp_port:6382                                        | ＃当前监听端口                                               |
| uptime_in_seconds:267366                             | ＃运行时间，单位是秒                                         |
| uptime_in_days:3                                     | ＃运行时间，单位是天                                         |
| hz:10                                                |                                                              |
| lru_clock:12856621                                   | ＃以分钟为单位进行自增的时钟，用于 LRU 管理                  |
| config_file:/opt/XXX/redis_conf/6382.conf            | ＃使用的配置文件的绝对路径                                   |
|                                                      |                                                              |
| **# Clients**                                        | ＃记录了已连接客户端的信息                                   |
| connected_clients:3                                  | ＃已经连接的客户端数量，只包括直接连接的客户端，不包括连接到从节点的客户端 |
| client_longest_output_list:0                         | ＃ 当前连接的客户端当中，最长的输出列表                      |
| client_biggest_input_buf:0                           | ＃ 当前连接的客户端当中，最大输入缓存                        |
| blocked_clients:0                                    | ＃ 正在等待阻塞命令（BLPOP、BRPOP、BRPOPLPUSH）的客户端的数量 |
|||
| **# Memory**                                         | ＃记录了服务器的内存信息                                     |
| used_memory:20195844128                              | ＃当前redis已经分配的内存数量，单位byte                      |
| used_memory_human:18.81G                             | ＃当前redis已经分配的内存数量，常用方便读取的单位            |
| used_memory_rss:20550877184                          | ＃从操作系统的角度，返回 Redis 已分配的内存总量（俗称常驻集大小）。这个值和 top 、 ps 等命令的输出一致。 |
| used_memory_peak:21053144288                         | #内存使用峰值                                                |
| used_memory_peak_human:19.61G                        | ＃内存使用峰值的友好读取数量                                 |
| used_memory_lua:33792                                | ＃Lua 引擎所使用的内存大小，单位byte                         |
| mem_fragmentation_ratio:1.02                         | # used_memory_rss 和 used_memory 之间的比率,即20550877184／20195844128=1.0175795106约为1.02。这个比值比1高一点点比较理想，比值太高说明有大量碎片，小于1时说明部分redis内存已经被操作系统交换到swap了，可能会影响响应时间。 |
|                                                      | //当 Redis 释放内存时，分配器可能会，也可能不会，将内存返还给操作系统。如果 Redis 释放了内存，却没有将内存返还给操作系统，那么 used_memory 的值可能和操作系统显示的 Redis 内存占用并不一致。查看 used_memory_peak 的值可以验证这种情况是否发生。 |
| mem_allocator:jemalloc-3.6.0                         | ＃在编译时指定的， Redis 所使用的内存分配器。可以是 libc 、 jemalloc 或者 tcmalloc |
|                                                      |                                                              |
| **# Persistence**                                    | ＃ RDB 持久化和 AOF 持久化有关的信息                         |
| loading:0                                            | ＃ 一个标志值，记录了服务器是否正在载入持久化文件            |
| rdb_changes_since_last_save:13652713                 | ＃最后一次持久化到现在的时间，单位秒                         |
| rdb_bgsave_in_progress:0                             | ＃一个标志值，表示是否正在创建RDB文件                        |
| rdb_last_save_time:1455590116                        | ＃最后一次创建RDB文件的UNIX时间戳，可以使用data -d @1455590116查看具体时间，如[t@bjb0541 ~]$ date -d @1455590116   Tue Feb 16 10:35:16 CST 2016 |
| rdb_last_bgsave_status:ok                            | ＃最近一次创建RDB成功还是失败                                |
| rdb_last_bgsave_time_sec:202                         | ＃最近一次创建RDB的耗时，单位秒                              |
| rdb_current_bgsave_time_sec:-1                       | ＃如果服务器当前正在写入RDB，这个时间就是已经操作耗费的时间。 |
| aof_enabled:0                                        | ＃是否启用了aof                                              |
| aof_rewrite_in_progress:0                            | #一个标记，当前是否正在创建AOF                               |
| aof_rewrite_scheduled:0                              | ＃一个标志值，记录了在 RDB 文件创建完毕之后，是否需要执行预约的 AOF 重写操作。 |
| aof_last_rewrite_time_sec:-1                         | ＃最近一次AOF的耗费的时间                                    |
| aof_current_rewrite_time_sec:-1                      | ＃如果当前正在写AOF，已经操作的时候值                        |
| aof_last_bgrewrite_status:ok                         | ＃最近一次AOF后台的成功或失败                                |
| aof_last_write_status:ok                             | ＃最近一次的AOF的成功或失败                                  |
|                                                      |                                                              |
| **# Stats**                                          | ＃状态                                                       |
| total_connections_received:88378                     | ＃已经接收的请求数                                           |
| total_commands_processed:35467619                    | ＃已经执行的命令数                                           |
| instantaneous_ops_per_sec:165                        | ＃每秒执行的操作数                                           |
| rejected_connections:0                               | ＃因为最大客户端数量限制而被拒绝的连接请求数量               |
| sync_full:5                                          | ＃完全同步次数（我猜测的）                                   |
| sync_partial_ok:0                                    |                                                              |
| sync_partial_err:0                                   |                                                              |
| expired_keys:120                                     | ＃因为过期而被删除的键数                                     |
| evicted_keys:0                                       | ＃因为最大内存容量限制而被驱逐（evict）的键数量              |
| keyspace_hits:15287495                               | ＃查找命中的次数                                             |
| keyspace_misses:0                                    | ＃查找失败的次数                                             |
| pubsub_channels:0                                    | ＃订阅的频道数                                               |
| pubsub_patterns:0                                    | ＃订阅的模式数                                               |
| latest_fork_usec:23562                               | ＃最近一次FOCK所用的时间                                     |
|                                                      |                                                              |
| **# Replication**                                    | ＃主从复制信息                                               |
| role:slave                                           | ＃主机角色                                                   |
| master_host:192.168.171.139                          | ＃主服务器IP                                                 |
| master_port:6381                                     | ＃主服务器端口                                               |
| master_link_status:up                                | ＃主服务器状态，UP正常，DOWN已经断开                         |
| master_last_io_seconds_ago:0                         | ＃距离最近一次与主服务器进行通信已经过去了多少秒。           |
| master_sync_in_progress:0                            | ＃标记值，表示当前是否正在进行主从复制。                     |
| slave_repl_offset:83184763601                        |                                                              |
| slave_priority:100                                   |                                                              |
| slave_read_only:1                                    |                                                              |
| connected_slaves:0                                   | ＃已连接的从服务器数量                                       |
| master_repl_offset:0                                 |                                                              |
| repl_backlog_active:0                                |                                                              |
| repl_backlog_size:1048576                            |                                                              |
| repl_backlog_first_byte_offset:281559737             |                                                              |
| repl_backlog_histlen:1048576                         |                                                              |
|                                                      |                                                              |
| **# CPU**                                            | ＃CPU信息                                                    |
| used_cpu_sys:1939.74                                 | ＃耗费系统CPU                                                |
| used_cpu_user:4148.13                                | ＃耗费用户CPU                                                |
| used_cpu_sys_children:101.03                         | ＃后台进程耗费的系统CPU                                      |
| used_cpu_user_children:900.62                        | ＃后台进程耗费的用户CPU                                      |
|                                                      |                                                              |
| **# Keyspace**                                       | ＃部分记录了数据库相关的统计信息，比如数据库的键数量、设置有过期时间的key的数量等。对于每个数据库，这个部分都会添加一行以下格式的信息： |
| db0:keys=16839997,expires=16061394,avg_ttl=560458485 |                                                              |

### 告警项

| 告警指标                                          | 含义                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
|**存活监控**||
| **redis uptime监控**(redis_uptime)                | 重启监控                                                     |
|||
|**连接数监控**||
| **连接个数**(connected_clients)                   | 客户端连接个数，如果连接数过高，影响redis吞吐量。常规建议不要超过5000 |
| **拒绝的连接个数**(rejected_connections)          | redis连接个数达到maxclients限制，拒绝新连接的个数。          |
| **新创建连接个数**(total_connections_received)    | 如果新创建连接过多，过度地创建和销毁连接对性能有影响         |
| **连接数使用率**(connected_clients_pct)           | 连接数使用百分比，通过(connected_clients/macclients)计算；如果达到1，redis开始拒绝新连接创建。 |
| **list阻塞调用被阻塞的连接个数**(blocked_clients) | BLPOP这类命令没使用过，如果监控数据大于0，还是建议排查原因。 |
|||
|**内存监控**||
| **redis分配的内存大小**(used_memory)                       |            redis真实使用内存，不包含内存碎片                                                  |
| **redis内存使用比例**(used_memory_pct)  | 已分配内存的百分比  |
|  **redis进程使用内存大小**(used_memory_rss)| 进程实际使用的物理内存大小  |
| **redis内存碎片率**(mem_fragmentation_ratio) |   表示(used_memory_rss/used_memory)，碎片率过大，导致内存资源浪费 |
|||
|**redis键空间的状态监控**||
|**键个数**(keys)| redis实例包含的键个数。建议控制在1kw内；单实例键个数过大，可能导致过期键的回收不及时。|
|**设置有生存时间的键个数**(keys_expires)| 是纯缓存或业务的过期长，都建议对键设置TTL; 避免业务的死键问题. （expires字段）|
|**估算设置生存时间键的平均寿命**(avg_ttl)| redis会抽样估算实例中设置TTL键的平均时长，单位毫秒。如果无TTL键或在Slave则avg_ttl一直为0|
|**LRU淘汰的键个数**(evicted_keys)| 因used_memory达到maxmemory限制，并设置有淘汰策略的实例；（对排查问题重要，可不设置告警）|
|**过期淘汰的键个数**(expired_keys)| 删除生存时间为0的键个数；包含主动删除和定期删除的个数。|
|||
|**qps**||
|**redis处理的命令数**(total_commands_processed)| 监控采集周期内的平均qps,如果请求数过多，redis过载导致请求堆积。|
|**redis当前的qps**(instantaneous_ops_per_sec)| redis内部较实时的每秒执行的命令数；可和total_commands_processed监控互补。|
|||
|**请求命中率监控**||
|**请求键被命中次数**(keyspace_hits)| redis请求键被命中的次数|
|**请求键未被命中次数**(keyspace_misses)| redis请求键未被命中的次数；当命中率较高如95%，如果请求量大，未命中次数也会很多。|
|**请求键的命中率**(keyspace_hit_ratio)|使用keyspace_hits/(keyspace_hits+keyspace_misses)计算所得，是度量Redis缓存服务质量的标准|
|||
|**fork阻塞时长监控**||
|**最近一次fork阻塞的微秒数**(latest_fork_usec)|最近一次Fork操作阻塞redis进程的耗时数，单位微秒。|
|||
|**网络指标监控**||
|**redis网络入口流量字节数**(total_net_input_bytes) ||
|**redis网络出口流量字节数**(total_net_output_bytes)||
|**redis网络入口kps**（instantaneous_input_kbps）    ||
|**redis网络出口kps**(instantaneous_output_kbps)     ||
													 










