CLUSTER INFO
================

::

    CLUSTER INFO

打印集群的相关信息：

::

    127.0.0.1:7711> CLUSTER INFO
    cluster_state:ok                        -- 集群运作状态
    cluster_known_nodes:1                   -- 集群包含的节点数量
    cluster_reachable_nodes:1               -- 集群目前可正常连接的节点数量
    cluster_size:0                          -- 集群目前储存着数据的非空主服务器的数量
    cluster_stats_messages_sent:396         -- 集群已发送的消息数量
    cluster_stats_messages_received:396     -- 集群已接收的消息数量
