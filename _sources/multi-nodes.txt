搭建多节点集群
=======================

因为每个 Disque 节点都会将自己的配置信息储存在 ``disque-server`` 运行的文件夹里面，
而同一个文件夹只能有一份这样的配置信息，
所以如果我们打算同时运行多个节点，
那么就必须在不同的文件夹里面运行 ``disque-server`` ，
并为每个节点指定不同的端口。

假设我们现在打算运行三个 Disque 节点，
那么首先要做的就是创建三个文件夹，
然后分别在这些文件夹里面运行 ``disque-server`` ：

::

    $ mkdir node10086 node10087 node10088   -- 创建三个文件夹

    $ cd node10086                          -- 进入 node10086 文件夹

    $ disque-server --port 10086 &          -- 然后启动一个端口号为 10086 的节点
    [1] 16941                                               

    $ cd ../node10087/                      -- 进入 node10087 文件夹
    
    $ disque-server --port 10087 &          -- 然后启动一个端口号为 10087 的节点
    [2] 16942

    $ cd ../node10088/                      -- 进入 node10088 文件夹

    $ disque-server --port 10088 &          -- 然后启动一个端口号为 10088 的节点
    [3] 16943

在启动了三个节点之后，
接下来要做的就是将这三个节点组合起来，
形成一个包含三个节点的集群，
步骤如下：

::

    $ disque -p 10086                               -- 连接端口号为 10086 的节点
    127.0.0.1:10086> CLUSTER INFO                   -- 打印集群信息
    cluster_state:ok
    cluster_known_nodes:1                           -- 目前集群只有一个节点
    cluster_reachable_nodes:0
    cluster_size:1
    cluster_stats_messages_sent:0
    cluster_stats_messages_received:0

    127.0.0.1:10086> CLUSTER MEET 127.0.0.1 10087   -- 将端口号为 10087 的节点加入到集群里面
    OK

    127.0.0.1:10086> CLUSTER INFO
    cluster_state:ok
    cluster_known_nodes:2                           -- 集群现在有两个节点
    cluster_reachable_nodes:1
    cluster_size:2
    cluster_stats_messages_sent:5
    cluster_stats_messages_received:5

    127.0.0.1:10086> CLUSTER MEET 127.0.0.1 10088   -- 将端口号为 10088 的节点加入到集群里面
    OK

    127.0.0.1:10086> CLUSTER INFO
    cluster_state:ok
    cluster_known_nodes:3                           -- 集群现在有三个节点
    cluster_reachable_nodes:2
    cluster_size:3
    cluster_stats_messages_sent:23
    cluster_stats_messages_received:23

大功告成！
现在我们已经建立了一个由三个节点组成的 Disque 集群，
你可以随意地向这个集群发送命令来进行测试了。
