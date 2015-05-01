.. highlight:: c

Disque 源码分析
======================

本文将对 Disque 的核心数据结构进行介绍，
并在最后通过分析 ``ADDJOB`` 命令的实现来帮助大家了解 Disque 的运作原理。

因为时间关系，
本章只介绍了 Disque 源码中最重点的部分，
并且只对集群和命令的运作原理进行了最基本的介绍，
但对于有兴趣深入了解 Disque 源码的读者来说，
应该是一个还不错的入门向导。


代码重用
-------------

Disque 重用了大量 Redis 的底层代码，
比如数据结构部分、事件部分、网络通信部分、服务器主循环部分等等。

这不是 antirez 第一次做这种事 —— 
更早出现的 Redis Sentinel 也大量地重用了 Redis 的代码，
并在这个基础上，
提供了一集不同的命令，
用于监视 Redis 服务器。
但从代码的角度来说，
Redis Sentinel 实际上就是一个修改版的 Redis 。

可以说，
Disque 和 Redis Sentinel 都把 Redis 中的一些子系统当做一个框架来使用，
特别是 Redis 中的网络通信部分，
尤为如此。

因为以上原因，
熟悉 Redis 代码的人阅读 Disque 的代码应该会比较容易上手。

对于熟悉 Redis 的人来说，
阅读 Disque 代码主要应该关注以下几个文件：

- ``disque.h`` & ``disque.c`` ，Disque 的队列实现。

- ``job.h`` & ``job.c`` ，Disque 的任务实现。

- ``disque.h`` & ``disque.c`` ，Disque 的服务器进程，相当于 Redis 中的 ``redis.h`` 和 ``redis.c`` ，但进行了相应的修改。

- ``cluster.h`` & ``cluster.c`` ，Disque 集群在 Redis 集群的基础上进行了一些修改，但基本的运作方式是相同的。


服务器状态
--------------

Disque 的服务器状态由 ``disque.h/disqueServer`` 结构表示，
其中的 ``jobs`` 属性和 ``queues`` 属性分别记录了储存在服务器里面的所有任务和所有队列：

::

    struct disqueServer {

        // 其他属性……

        /* Jobs & Queues */

        // jobs 是一个字典，它包含了服务器储存的所有任务
        // 其中字典的键为任务 ID ，而键对应的值则是一个 job.h/job 结构
        dict *jobs;                 /* Main jobs hash table, by job ID. */

        // queues 也是一个字典，它包含了服务器储存的所有队列
        // 其中字典的键为队列的名字，而键对应的则是一个 queue.h/queue 结构
        dict *queues;               /* Main queues hash table, by queue name. */

        // 其他属性……
    };

接下来，
就让我们来看看 ``job`` 结构的定义和 ``queue`` 结构的定义。


队列
--------------

Disque 中的每个队列都由一个 ``queue.h/queue`` 结构定义：

::

    typedef struct queue {

        // 队列的名字
        robj *name;      /* Queue name as a string object. */

        // 一个跳跃表，储存了所有被放进队列里面的任务
        // 各个任务按照任务 ID 从小到大有序地进行排列
        skiplist *sl;    /* The skiplist with the queued jobs. */

        // 其他属性……

    } queue;

在了解了队列结构之后，
接下来让我们看看 Disque 是怎样表示每个任务的。


任务
---------------

Disque 中的每个任务都由一个 ``job.h/job.h`` 结构定义：

::

    typedef struct job {

        // 任务 ID
        char id[JOB_ID_LEN];    /* Job ID. */

        // 其他属性……

        // 所属的队列
        robj *queue;            /* Job queue name. */

        // 任务的内容
        sds body;               /* Body, or NULL if job is just an ACK. */

        // 其他属性……

    } job;


节点与集群
----------------

Disque 集群的工作方式和 Redis 集群的工作方式非常相似，
但是不同于 Redis 能够自由选择单机模式（standalone mode）或者集群模式（cluster mode），
Disque 总是运行在集群模式之下 ——
当一个 Disque 服务器启动时，
它就是一个 Disque 集群的节点（node）了。

Disque 集群部分的代码记录在 ``cluster.h`` 和 ``cluster.c`` 里面，
运作机制和 Redis 的集群模式并无太大区别。
Disque 的主要作用就是在多个节点之间传播和复制任务，
当一个节点执行用户的 ``ADDJOB`` 命令，
将一个新任务添加到自己内部的队列里面的同时，
它会将这个新任务以同步或者异步的方式（默认同步，可以通过 ``ASYNC`` 选择异步），
传播给集群中的其他节点，
而这种传播是通过发送集群消息（cluster message）来完成的。

以下是 Disque 集群用于传播任务、任务 ID 以及任务请求的集群消息数据结构：

::

    // 传播任务及其内容
    /* This data section is used in different message types where we need to
     * transmit one or multiple full jobs.
     *
     * Used by: ADDJOB, YOURJOBS. */
    typedef struct {
        uint32_t numjobs;   /* Number of jobs stored here. */
        uint32_t datasize;  /* Number of bytes following to describe jobs. */
        /* The variable data section here is composed of 4 bytes little endian
         * prefixed length + serialized job data for each job:
         * [4 bytes len] + [serialized job] + [4 bytes len] + [serialized job] ...
         * For a total of exactly 'datasize' bytes. */
         unsigned char jobs_data[8]; /* Defined as 8 bytes just for alignment. */
    } clusterMsgDataJob;

    // 传播任务 ID
    /* This data section is used when we need to send just a job ID.
     *
     * Used by: GOTJOB, SETACK, and many more. */
    typedef struct {
        char id[JOB_ID_LEN];
        uint32_t aux; /* Optional field:
                         For SETACK: Number of nodes that may have this message.
                         For QUEUEJOB: Delay starting from msg reception. */
    } clusterMsgDataJobID;

    // 向其他节点请求传播某个队列的任务
    /* This data section is used by NEEDJOBS to specify in what queue we need
     * a job, and how many jobs we request. */
    typedef struct {
        uint32_t count;     /* How many jobs we request. */
        uint32_t qnamelen;  /* Queue name total length. */
        char qname[8];      /* Defined as 8 bytes just for alignment. */
    } clusterMsgDataNeedJobs;


命令执行流程
----------------

在简单地了解了 Disque 的各个主要数据结构之后，
让我们对 ``ADDJOB`` 命令的实现代码进行分析，
并通过追踪这个命令的执行流程来了解 Disque 是怎样创建、储存并向其他节点传播一个任务的：

::

    void addjobCommand(client *c) {

        // 变量声明
        long long replicate = server.cluster->size > 3 ? 3 : server.cluster->size;
        long long ttl = 3600*24;
        long long retry = -1;
        long long delay = 0;
        long long maxlen = 0; /* Max queue length for job to be accepted. */
        mstime_t timeout;
        int j, retval;
        int async = 0;  /* Asynchronous request? */
        int extrepl = getMemoryWarningLevel() > 0; /* Replicate externally? */
        static uint64_t prev_ctime = 0;

        // 参数分析
        /* Parse args. */
        for (j = 4; j < c->argc; j++) {
            char *opt = c->argv[j]->ptr;
            int lastarg = j == c->argc-1;
            if (!strcasecmp(opt,"replicate") && !lastarg) {
                retval = getLongLongFromObject(c->argv[j+1],&replicate);
                if (retval != DISQUE_OK || replicate <= 0 || replicate > 65535) {
                    addReplyError(c,"REPLICATE must be between 1 and 65535");
                    return;
                }
                j++;
            } else if (!strcasecmp(opt,"ttl") && !lastarg) {
                retval = getLongLongFromObject(c->argv[j+1],&ttl);
                if (retval != DISQUE_OK || ttl <= 0) {
                    addReplyError(c,"TTL must be a number > 0");
                    return;
                }
                j++;
            } else if (!strcasecmp(opt,"retry") && !lastarg) {
                retval = getLongLongFromObject(c->argv[j+1],&retry);
                if (retval != DISQUE_OK || retry < 0) {
                    addReplyError(c,"RETRY time must be a non negative number");
                    return;
                }
                j++;
            } else if (!strcasecmp(opt,"delay") && !lastarg) {
                retval = getLongLongFromObject(c->argv[j+1],&delay);
                if (retval != DISQUE_OK || delay < 0) {
                    addReplyError(c,"DELAY time must be a non negative number");
                    return;
                }
                j++;
            } else if (!strcasecmp(opt,"maxlen") && !lastarg) {
                retval = getLongLongFromObject(c->argv[j+1],&maxlen);
                if (retval != DISQUE_OK || maxlen <= 0) {
                    addReplyError(c,"MAXLEN must be a positive number");
                    return;
                }
                j++;
            } else if (!strcasecmp(opt,"async")) {
                async = 1;
            } else {
                addReply(c,shared.syntaxerr);
                return;
            }
        }

        /* Parse the timeout argument. */
        if (getTimeoutFromObjectOrReply(c,c->argv[3],&timeout,UNIT_MILLISECONDS)
            != DISQUE_OK) return;

        /* REPLICATE > 1 and RETRY set to 0 does not make sense, why to replicate
         * the job if it will never try to be re-queued if case the job processing
         * is not acknowledged? */
        if (replicate > 1 && retry == 0) {
            addReplyError(c,"With RETRY set to 0 please explicitly set  "
                            "REPLICATE to 1 (at-most-once delivery)");
            return;
        }

        /* DELAY greater or equal to TTL is silly. */
        if (delay >= ttl) {
            addReplyError(c,"The specified DELAY is greater than TTL. Job refused "
                            "since would never be delivered");
            return;
        }

        /* When retry is not specified, it defaults to 1/10 of the TTL. */
        if (retry == -1) {
            retry = ttl/10;
            if (retry == 0) retry = 1;
        }

        /* Check if REPLICATE can't be honoured at all. */
        int additional_nodes = extrepl ? replicate : replicate-1;

        if (additional_nodes > server.cluster->reachable_nodes_count) {
            if (extrepl &&
                additional_nodes-1 == server.cluster->reachable_nodes_count)
            {
                addReplySds(c,
                    sdsnew("-NOREPL Not enough reachable nodes "
                           "for the requested replication level, since I'm unable "
                           "to hold a copy of the message for memory usage "
                           "problems.\r\n"));
            } else {
                addReplySds(c,
                    sdsnew("-NOREPL Not enough reachable nodes "
                           "for the requested replication level\r\n"));
            }
            return;
        }

        // 检查队列是否已经达到最大长度
        /* If maxlen was specified, check that the local queue len is
         * within the requested limits. */
        if (maxlen && queueNameLength(c->argv[1]) > (unsigned long) maxlen) {
            addReplySds(c,
                sdsnew("-MAXLEN Queue is already longer than "
                       "the specified MAXLEN count\r\n"));
            return;
        }

        // 创建一个新任务
        /* Create a new job. */
        job *job = createJob(NULL,JOB_STATE_WAIT_REPL,ttl); // 创建任务
        job->queue = c->argv[1];                            // 记录任务所在队列的名字
        incrRefCount(c->argv[1]);
        job->repl = replicate;

        /* If no external replication is used, add myself to the list of nodes
         * that have a copy of the job. */
        if (!extrepl)
            dictAdd(job->nodes_delivered,myself->name,myself);

        // 设置任务的各项属性
        /* Job ctime is milliseconds * 1000000. Jobs created in the same
         * millisecond gets an incremental ctime. The ctime is used to sort
         * queues, so we have some weak sorting semantics for jobs: non-requeued
         * jobs are delivered roughly in the order they are added into a given
         * node. */
        job->ctime = mstime()*1000000;
        if (job->ctime <= prev_ctime) job->ctime = prev_ctime+1;
        prev_ctime = job->ctime;

        job->etime = server.unixtime + ttl;
        job->delay = delay;
        job->retry = retry;
        job->body = sdsdup(c->argv[2]->ptr);

        /* Set the next time the job will be queued. Note that once we call
         * enqueueJob() the first time, this will be set to 0 (never queue
         * again) for jobs that have a zero retry value (at most once jobs). */
        if (delay) {
            job->qtime = server.mstime + delay*1000;
        } else {
            /* This will be updated anyway by enqueueJob(). */
            job->qtime = server.mstime + retry*1000;
        }

        /* Register the job locally in all the cases but when the job
         * is externally replicated and asynchronous replicated at the same
         * time: in this case we don't want to take a local copy at all. */
        if (!(async && extrepl) && registerJob(job) == DISQUE_ERR) {
            /* A job ID with the same name? Practically impossible but
             * let's handle it to trap possible bugs in a cleaner way. */
            serverLog(DISQUE_WARNING,"ID already existing in ADDJOB command!");
            freeJob(job);
            addReplyError(c,"Internal error creating the job, check server logs");
            return;
        }

        /* For replicated messages where ASYNC option was not asked, block
         * the client, and wait for acks. Otherwise if no synchronous replication
         * is used, or ASYNC option was enabled, we just queue the job and
         * return to the client ASAP.
         *
         * Note that for REPLICATE > 1 and ASYNC the replication process is
         * best effort. */
        if (replicate > 1 && !async) {

            // 决定以同步方式向其他节点传播这个任务

            c->bpop.timeout = timeout;
            c->bpop.job = job;
            c->bpop.added_node_time = server.mstime;
            blockClient(c,DISQUE_BLOCKED_JOB_REPL);
            setJobAssociatedValue(job,c);
            /* Create the nodes_confirmed dictionary only if we actually need
             * it for synchronous replication. It will be released later
             * when we move away from JOB_STATE_WAIT_REPL. */
            job->nodes_confirmed = dictCreate(&clusterNodesDictType,NULL);
            /* Confirm itself as an acknowledged receiver if this node will
             * retain a copy of the job. */
            if (!extrepl) dictAdd(job->nodes_confirmed,myself->name,myself);
        } else {

            // 决定以异步方式传递这个任务

            if (job->delay == 0) {
                if (!extrepl) enqueueJob(job); /* Will change the job state. */
            } else {
                /* Delayed jobs that don't wait for replication can move
                 * forward to ACTIVE state ASAP, and get scheduled for
                 * queueing. */
                job->state = JOB_STATE_ACTIVE;
                updateJobAwakeTime(job,0);
            }
            addReplyJobID(c,job);
            AOFLoadJob(job);
        }

        // 向其他节点传播这个任务……

        /* If the replication factor is > 1, send REPLJOB messages to REPLICATE-1
         * nodes. */
        if (additional_nodes > 0)
            clusterReplicateJob(job, additional_nodes, async);

        /* If the job is asynchronously and externally replicated at the same time,
         * send a QUEUE message ASAP to one random node, and delete the job from
         * this node right now. */
        if (async && extrepl) {
            dictEntry *de = dictGetRandomKey(job->nodes_delivered);
            if (de) {
                clusterNode *n = dictGetVal(de);
                clusterSendEnqueue(n,job,job->delay);
            }
            /* We don't have to unregister the job since we did not registered
             * it if it's async + extrepl. */
            freeJob(job);
        }
    }

总的来说，
``ADDJOB`` 命令要做的就是以下几件事情：

1. 创建一个新的 ``job`` 结构，并将用户给定的任务信息记录起来。

2. 查找用户指定的队列，如果找到就检查它是否超出了最大长度，如果未超长，那么就将任务添加到队列中；如果队列未创建，那么创建队列。

3. 以同步或者异步的方式，通过集群消息将这个任务传播给集群中的其他节点。


结论
----------------

总的来说，
Disque 对于 Redis 的变动并不大，
而且新添加的内容也不难读懂，
如果你也想要一窥 Disque 这个新的分布式任务队列实现，
那么不要犹豫，
赶紧去读读 Disque 的代码吧！

| huangz 
| 2015.5.1
