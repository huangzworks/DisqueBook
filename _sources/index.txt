.. Disque 使用教程 documentation master file, created by
   sphinx-quickstart on Sat Apr 25 12:14:34 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Disque 使用教程（DisqueBook.com）
===========================================

Disque 是一个内存储存的分布式任务队列实现，
它由 Redis 的作者 Salvatore Sanfilippo (\ `@antirez <https://twitter.com/antirez>`_\ )开发，
目前正处于预览版（alpha）阶段。

本文档将对 Disque 的安装方法和运行方法进行介绍，
说明各个 Disque 命令的作用，
并给出各个命令的运行示例，
帮助读者更好地理解 Disque 的使用方法。

关于 Disque 项目的更多信息可以参考 Disque 项目的 GitHub 页面：
`github.com/antirez/disque <https://github.com/antirez/disque>`_ 。


安装与运行
-------------------------------------------

.. toctree::
   :maxdepth: 2

   install
   client
   single-node
   multi-nodes


命令文档
-------------------------------------------

.. toctree::
   :maxdepth: 2

   addjob
   getjob

   qlen
   qpeek
   show

   ackjob
   fastack

   info
   hello

   enqueue
   dequeue
   deljob

.. 待实现命令
   qstat
   scan


关于本文档
-------------------------------------------

本文档由\ `黄健宏(huangz) <http://www.huangz.me/>`_\ 翻译并改写，
翻译部分的版权归 Disque 官方所有，
改写部分的版权归作者所有。

追踪本文档的更新信息请关注\ `本文档的 github 项目页面 <https://github.com/huangz1990/DisqueBook>`_\ ，
或者关注译者的各个社交网站。
有任何问题、意见或建议，
请在文档下方的论坛里面留言，
或者直接联系译者。


通过捐款支持本文档
-------------------------------------------

.. image:: image/huangz-payment-code.png
   :align: right

对文档进行翻译需要花费大量的时间和精力，
如果你喜欢这个《Disque 使用教程》文档的话，
欢迎通过捐款的方式，
支持译者的翻译工作，
非常感谢！

你可以使用\ `支付宝钱包 <https://mobile.alipay.com/main/download.htm?action=mobileClient>`_\ 扫描右侧的二维码进行捐款， 
或者通过向支付宝帐号 huangz1990@gmail.com 转帐进行捐款。
