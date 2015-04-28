SHOW
=======

::

    SHOW <job-id>

返回任务的相关描述。

::

    127.0.0.1:7711> SHOW DI216f7fa19e1bbe6f583061dbba0e8e28c6eb64e605a0SQ
    1) "id"                                                 -- 任务 ID
    2) "DI216f7fa19e1bbe6f583061dbba0e8e28c6eb64e605a0SQ"
    3) "queue"                                              -- 所属队列
    4) "greeting"
    5) "state"                                              -- 任务状态
    6) "queued"
    7) "repl"
    8) (integer) 1
    9) "ttl"                                                -- 生存时间
    10) (integer) 86080
    11) "ctime"                                             -- 创建时间
    12) (integer) 1430193729317000000
    13) "delay"                                             -- 是否延迟？
    14) (integer) 0
    15) "retry"                                             -- 何时进行重试？
    16) (integer) 8640
    17) "nodes-delivered"                                   -- 已向以下节点发送该任务
    18) 1) "216f7fa1b58239ad29dd64c02bfe50e609e88575"
    19) "nodes-confirmed"                                   -- 以下节点已确认收到该任务
    20) (empty list or set)
    21) "next-requeue-within"
    22) (integer) 8319321
    23) "next-awake-within"
    24) (integer) 8318821
    25) "body"                                              -- 任务的内容
    26) "good morning!"
