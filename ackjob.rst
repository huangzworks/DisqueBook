ACKJOB
========

::

    ACKJOB jobid1 jobid2 ... jobidN

通过给定任务 ID ，
向节点告知任务已经被执行。

接收到 ``ACK`` 消息的节点会将该消息复制至多个节点，
并尝试对任务和来自集群的 ``ACK`` 消息进行垃圾回收操作，
从而释放被占用的内存。

::

    disque> GETJOB FROM greeting                    -- 取出三个任务，然后确认其中两个已被执行
    1) 1) "greeting"
    2) "DI216f7fa17693623ffb3bd8b0902e134f4ab6a5d305a0SQ"
    3) "hello world!"

    disque> GETJOB FROM greeting
    1) 1) "greeting"
    2) "DI216f7fa16a8e4a7428b18c2b0ec180963795b0b705a0SQ"
    3) "good morning!"

    disque> GETJOB FROM greeting
    1) 1) "greeting"
    2) "DI216f7fa11413878f376588c85aca7c7fa22232f905a0SQ"
    3) "bye bye~"

    disque> ACKJOB DI216f7fa17693623ffb3bd8b0902e134f4ab6a5d305a0SQ
    (integer) 1

    disque> ACKJOB DI216f7fa11413878f376588c85aca7c7fa22232f905a0SQ
    (integer) 1

