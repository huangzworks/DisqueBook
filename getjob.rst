GETJOB
=========

::

    GETJOB [TIMEOUT <ms-timeout>] [COUNT <count>] FROM queue1 queue2 ... queueN

从给定的队列里面取出可用的任务，
或者在超时时间达到时，
返回 ``NULL`` 。

在默认情况下，
命令每次最多只会返回一个任务，
但使用 ``COUNT count`` 选项可以指定每次最多可以获取的任务数量。

任务会以数组的形式被返回，
每个任务由三个元素组成：

1. 任务来源的队列。
2. 任务的 ID 。
3. 任务本身。

如果用户给定了多个队列，
那么命令将按照队列被给定的顺序，
从左到右地对它们进行处理。

::

    disque> ADDJOB greeting "hello world!" 0                -- 将三个任务放进队列里面
    DI216f7fa17693623ffb3bd8b0902e134f4ab6a5d305a0SQ

    disque> ADDJOB greeting "good morning!" 0
    DI216f7fa16a8e4a7428b18c2b0ec180963795b0b705a0SQ

    disque> ADDJOB greeting "bye bye~" 0
    DI216f7fa11413878f376588c85aca7c7fa22232f905a0SQ

    disque> GETJOB FROM greeting                            -- 依次地从队列里面取出三个任务
    1) 1) "greeting"                                        -- 任务来自 "greeting" 队列
    2) "DI216f7fa17693623ffb3bd8b0902e134f4ab6a5d305a0SQ"   -- 任务的 ID
    3) "hello world!"                                       -- 任务的内容

    disque> GETJOB FROM greeting
    1) 1) "greeting"
    2) "DI216f7fa16a8e4a7428b18c2b0ec180963795b0b705a0SQ"
    3) "good morning!"

    disque> GETJOB FROM greeting
    1) 1) "greeting"
    2) "DI216f7fa11413878f376588c85aca7c7fa22232f905a0SQ"
    3) "bye bye~"
