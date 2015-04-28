QLEN
=========

::

    QLEN <queue-name>

返回队列目前存放的任务数量。

::

    disque> QLEN greeting                               -- 空队列
    (integer) 0

    disque> ADDJOB greeting "hello world!" 0            -- 将第一个任务推入到队列
    DI216f7fa141be40b1a54db80875bacec836377a6905a0SQ

    disque> QLEN greeting
    (integer) 1

    disque> ADDJOB greeting "good morning!" 0           -- 将第二个任务推入到队列
    DI216f7fa19e1bbe6f583061dbba0e8e28c6eb64e605a0SQ

    disque> QLEN greeting
    (integer) 2

