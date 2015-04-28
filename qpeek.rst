QPEEK
=========

::

    QPEEK <qname> <count>

在不取出任务的情况下，
从队列里面返回指定数量的任务：

- 当 ``count`` 为正数时，
  命令会按照任务入队的时间，
  以从旧到新的顺序返回任务（和 ``GETJOB`` 命令一样，尽可能地维持一个先进先出的顺序）。

- 当 ``count`` 为负数时，
  命令以从新到旧的顺序返回任务。

::

    disque> ADDJOB greeting "hello world!" 0                    -- 将三个任务放入队列
    DI216f7fa1590f8975b29db52f663c18f80f73c24705a0SQ

    disque> ADDJOB greeting "good morning!" 0
    DI216f7fa1c33c8d99678f44b5658545f3f2603b8d05a0SQ

    disque> ADDJOB greeting "bye bye~" 0
    DI216f7fa1d137377120adaf571418aff559578b8a05a0SQ

    disque> QPEEK greeting 10                                   -- 按从旧到新的顺序返回任务
    1) 1) "DI216f7fa1590f8975b29db52f663c18f80f73c24705a0SQ"
    2) "hello world!"
    2) 1) "DI216f7fa1c33c8d99678f44b5658545f3f2603b8d05a0SQ"
    2) "good morning!"
    3) 1) "DI216f7fa1d137377120adaf571418aff559578b8a05a0SQ"
    2) "bye bye~"

    disque> QPEEK greeting -10                                  -- 按从新到旧的顺序返回任务
    1) 1) "DI216f7fa1d137377120adaf571418aff559578b8a05a0SQ"
    2) "bye bye~"
    2) 1) "DI216f7fa1c33c8d99678f44b5658545f3f2603b8d05a0SQ"
    2) "good morning!"
    3) 1) "DI216f7fa1590f8975b29db52f663c18f80f73c24705a0SQ"
    2) "hello world!"
