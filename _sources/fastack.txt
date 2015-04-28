FASTACK
==========

::

    FASTACK jobid1 jobid2 ... jobidN

尽最大努力在集群范围内对给定的任务进行删除。

在网络连接良好并且所有节点都在线时，
这个命令的效果和 ``ACKJOB`` 命令的效果一样，
但是因为这个命令引发的消息交换比 ``ACKJOB`` 要少，
所以它的速度比 ``ACKJOB`` 要快不少。

但是当集群中包含了失效节点的时候，
``FASTACK`` 命令比 ``ACKJOB`` 命令更容易出现多次发送同一消息的情况。

::

    disque> ADDJOB greeting "hello world!" 0
    DI216f7fa1c513d73d69c968aa25d482c498f4c1f005a0SQ

    disque> FASTACK DI216f7fa1c513d73d69c968aa25d482c498f4c1f005a0SQ
    (integer) 1
