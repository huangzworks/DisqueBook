CLUSTER MEET
================

::

    CLUSTER MEET ip port

将地址为 ``ip`` ，
端口号为 ``port`` 的 Disque 节点加入到当前节点所在的集群里面：

::

    127.0.0.1:7711> CLUSTER NODES    -- 集群里面只有端口号为 7711 的节点（当前节点）
    33d40d27b7be502a28c5b97a6d569347a0bc178d :7711 myself 0 0 connected

    127.0.0.1:7711> CLUSTER MEET 127.0.0.1 7712
    OK

    127.0.0.1:7711> CLUSTER NODES    -- 端口号为 7712 的节点已经被添加到集群当中
    6c56c20d868d76ab199b8d1ea23a99db7ee2a8e2 127.0.0.1:7712 noflags 0 1430839222607 connected
    33d40d27b7be502a28c5b97a6d569347a0bc178d 127.0.0.1:7711 myself 0 0 connected
