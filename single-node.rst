搭建单节点集群
======================

Disque 以集群模式运行，
每个服务器都是集群中的一个节点，
用户可以运行任意数量的节点，
只要确保每个节点的端口号不同即可。

在默认情况下，
运行 Disque 服务器程序 ``disque-server`` 将启动一个端口号为 ``7711`` 的 Disque 节点：

::

    $ ./disque-server 
    528:C 28 Apr 11:50:08.519 # Warning: no config file specified, using the default config. In order to specify a config file use ./disque-server /path/to/disque.conf
    528:P 28 Apr 11:50:08.521 * Increased maximum number of open files to 10032 (it was originally set to 256).
    528:P 28 Apr 11:50:08.528 * Node configuration loaded, I'm 216f7fa1b58239ad29dd64c02bfe50e609e88575
                                            Disque 0.0.1 (0bbd831e/0) 64 bit
              _ -                                                        
            .                               Port: 7711
            .    o    .                     PID: 528
                    .                                                   
                  -                              http://disque.io       
                                                                                

    528:P 28 Apr 11:50:08.528 # Server started, Disque version 0.0.1
    528:P 28 Apr 11:50:08.529 * The server is now ready to accept connections on port 7711

因为 Disque 集群并不限制节点的数量，
所以即使只有一个节点，
Disque 也是可以执行命令的，
如果你只是打算测试一下 Disque ，
那么现在就可以使用 Disque 客户端对 Disque 进行测试了。

另一方面，
如果你打算构建一个由多个节点组成的 Disque 集群，
那么请继续阅读接下来的一节。
