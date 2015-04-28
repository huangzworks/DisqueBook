DELJOB
========

::

    DELJOB <job-id> ... <job-id>

在节点里面彻底地删除给定的任务。
这个命令和 ``FASTACK`` 很相似，
唯一的不同是，
``DELJOB`` 命令引发的删除操作只会在单个节点里面执行，
它不会将 ``DELJOB`` 集群总线消息（cluster bus message）发送至其他节点。
