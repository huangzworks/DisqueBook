安装方法
===============

以下是下载、编译并安装 Disque 的完整步骤：

1. 执行命令 ``git clone https://github.com/antirez/disque.git`` ，使用 Git 克隆 Disque 的项目文件夹。

2. 执行 ``cd disque`` ，切换至 Disque 的项目文件夹。

3. 执行 ``make`` ，编译 Disque 。在编译好的文件里面， ``disque-server`` 文件是 Disque 的服务器程序，而 ``disque`` 文件则是 Disque 的客户端程序。

4. 可选：执行 ``make test`` ，测试编译好的 Disque 程序。
