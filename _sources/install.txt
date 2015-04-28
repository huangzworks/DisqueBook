安装方法
===============

以下是下载、编译并安装 Disque 的完整步骤：

1. 执行命令 ``git clone https://github.com/antirez/disque.git`` ，使用 Git 克隆 Disque 的项目文件夹。

2. 执行 ``cd disque`` 命令，切换至 Disque 的项目文件夹。

3. 执行 ``make`` 命令，编译 Disque 。

4. 执行 ``cd src`` 命令，进入到 ``src`` 文件夹，里面有编译好的 Disque 服务器文件 ``disque-server`` 和 Disque 客户端文件 ``disque`` 。

5. 可选：执行 ``make test`` ，测试编译好的 Disque 程序。
