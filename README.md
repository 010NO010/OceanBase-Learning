<img width="529" height="162" alt="image" src="https://github.com/user-attachments/assets/edde9af2-e29a-4ffc-874c-806425960f4a" /># OceanBase-Learning  
# Author: 010no010
# 本教程旨在帮助人更了解基础的OceanBase，方便参加蚂蚁数据库竞赛，教程只是我个人的建议希望可以给大家带来帮助，有什么不足的希望大家给我一些指正
本教程适用于希望在 Windows 电脑上通过 WSL2 搭建 OceanBase 社区版的初学者。本教程以 Windows + WSL2 + Ubuntu 22.04 + OceanBase CE 4.5.0 为例。  目标：从一个全新的 Windows 环境开始，最终能够进入 obclient，执行 SQL

一、最终环境

本教程最终环境：
Windows
└── WSL2
    └── Ubuntu 22.04
        ├── OceanBase CE 4.5.0
        ├── OBD
        ├── OBClient
        ├── OBProxy
        ├── OBAgent
        ├── Prometheus
        └── Grafana（监听暂不可用）

其中最重要的是：
OceanBase Observer
        ↓
     2881
        ↓
    OBClient
只要能够通过 2881 连接到 Observer，就可以开始使用 OceanBase。

二、准备工作

2.1 Windows 要求
建议：
Windows 10 / Windows 11
开启 CPU 虚拟化
开启 WSL2
至少 2 个 CPU 核心
建议至少 6 GB 可用内存
建议至少 20 GB 可用磁盘空间

官方单机部署文档目前建议至少：

2 vCPU
6 GB 可用内存
20 GB 可用磁盘

注意：！！！！
这里的 6 GB 是给 OceanBase 单机体验环境的官方建议规格。我个人是建议给到4GB因为最低要求是4gb，不然跑不起来，要很慢

三、安装 WSL2

以管理员身份打开 PowerShell：
wsl --install
安装完成后重启 Windows。
然后查看 WSL：
wsl -l -v
正常情况下可以看到类似：

NAME            STATE           VERSION
Ubuntu-22.04    Running         2

如果 VERSION 是：
2
说明使用的是 WSL2。
（为什么要用wsl2）：
OceanBase 是一个分布式数据库系统，它的生产环境主要运行在 Linux 服务器上。
例如：
阿里云服务器
企业数据库服务器
云计算集群
Windows 本身不能直接运行 OceanBase
Windows
   |
   ├── exe程序
   ├── dll
   └── Windows服务
   而 OceanBase 需要：
   Linux
 |
 ├── gcc环境
 ├── shell脚本
 ├── Linux文件权限
 ├── Linux网络模型
 └── Linux进程管理
所以Windows CMD
obd cluster start demo是不可直接运行的

四、下载并进入 Ubuntu

在 Windows PowerShell 中：
wsl -d Ubuntu-22.04

进入后类似：
oceanbase@电脑名:~$
以后看到：
$
并且前面是 Ubuntu 用户名，就说明已经进入 Linux 环境。

五、检查 Ubuntu 系统

首先：
cat /etc/os-release
如果是 Ubuntu 22.04，会看到类似：
VERSION_ID="22.04"
检查磁盘：
df -h /
检查内存：
free -h
检查 CPU：
nproc

六、安装基础依赖

执行：
sudo apt update
然后：
sudo apt install -y curl wget ca-certificates
安装完成后检查：
curl --version

七、安装 OceanBase All-in-One

OceanBase 官方提供 All-in-One 安装方式，可以一次安装 OBD、OBClient 以及部署 OceanBase 所需的相关组件。
执行：
bash -c "$(curl -s https://obbusiness-private.oss-cn-shanghai.aliyuncs.com/download-center/opensource/oceanbase-all-in-one/installer.sh)"
安装完成后，如果当前终端找不到相关命令，可以执行：
source ~/.oceanbase-all-in-one/bin/env.sh
检查：
obd --version
以及：
obclient --version
如果能够正常输出版本号，说明安装成功。

八、检查 OBD

执行：
obd cluster list
第一次安装时可能没有任何集群。
这是正常的。

九、部署 OceanBase

官方提供：
obd demo
用于快速部署单节点 OceanBase。
不过如果希望自己控制配置，可以使用 OBD 配置文件部署。本教程使用配置文件方式，因为这样可以更加清楚地理解 OceanBase 的配置。

十、创建 OceanBase 配置文件

创建：
mkdir -p ~/ob-demo
cd ~/ob-demo
创建配置：
nano config.yaml
可以使用下面的配置作为学习环境参考：

oceanbase-ce:
  servers:
    - name: server1
      ip: 127.0.0.1
  global:
    home_path: /home/oceanbase/oceanbase-ce
    memory_limit: 4G
    system_memory: 1G

    datafile_size: 2G
    datafile_maxsize: 8G
    datafile_next: 2G

    log_disk_size: 14G

    cpu_count: 8
    production_mode: false

    enable_syslog_wf: false
    enable_syslog_recycle: true
  server1:
    zone: zone1
obproxy-ce:
  servers:
    - 127.0.0.1
  global:
    listen_port: 2883
    prometheus_listen_port: 2884

    obproxy_sys_password: your_obproxy_password

注意：！！！！！！！！！！！！
memory_limit: 4G 是针对资源比较紧张的学习环境的实践配置。如果你的机器资源充足，可以根据官方单机部署规格使用更高的内存配置。不要机械照抄 4G 到生产环境。

十一、为什么我这里特别强调 memory_limit？

这是整个安装过程中非常容易踩的坑。
例如：
memory_limit: 3G
在某些 OceanBase 版本/部署模式下可能无法满足 Observer 的配置要求。
实际启动日志可能出现：

Invalid config
name=memory_limit
value=3G
ret=-4147

最终：
OB_INVALID_CONFIG
如果遇到这个问题，首先检查：
grep -E 'memory_limit|system_memory' \
/home/oceanbase/.obd/cluster/demo/config.yaml

十二、部署集群

如果使用配置文件部署，可以执行：
obd cluster deploy demo -c config.yaml
然后查看：
obd cluster list
如果看到：
demo
说明集群已经完成部署。

十三、启动 OceanBase

执行：
obd cluster start demo
正常情况下会看到：

Start observer ok
observer program health check ok
Connect to observer 127.0.0.1:2881 ok

如果还部署了 OBProxy、OBAgent、Prometheus、Grafana，还会继续检查这些组件。
但是我的GraFana组件一直不好用，不过Grafana 是监控可视化组件，不影响我们现在学习和使用 OceanBase 数据库。

十四、 Grafana 会报错

在 WSL 学习环境中，可能出现：
[WARN] Failed to start 127.0.0.1 grafana
或者：
grafana program health check
长时间没有通过。
这不一定意味着 OceanBase 数据库本身失败。
真正重要的是：
observer program health check ok
以及：
Connect to observer 127.0.0.1:2881 ok
如果这两个正常，就说明 OceanBase Observer 已经能够工作

十五、检查 OceanBase 状态

执行：
obd cluster display demo
可以查看集群中各组件的状态。
也可以：
obd cluster list
查看集群列表。
注意：
deployed
表示：集群已经部署。它不等于：集群正在运行。
因此真正判断 OceanBase 是否可以使用，最好实际连接 2881。

十六、第一次登录 OceanBase

使用 OBClient：
obclient -h127.0.0.1 -P2881 -uroot@sys -p
然后：
Enter password:
输入你的 root_password。
如果成功，会看到：
Welcome to the OceanBase.
以及：
Server version: OceanBase_CE 4.5.0.0
最后出现：
obclient[root@sys][(none)]>
看到这个，就说明：
OceanBase 已经真正可以使用了！

十七、如何查看 root 密码

如果使用 OBD 部署，并且没有自己设置密码，可以检查配置：
grep -nE 'root_password|password' \
/home/oceanbase/.obd/cluster/demo/config.yaml
可以看到：
root_password: xxxxxxxxx

十八、修改 root 密码

进入 OceanBase：
obclient -h127.0.0.1 -P2881 -uroot@sys -p
然后执行：
ALTER USER root IDENTIFIED BY '你的新密码';
例如：
ALTER USER root IDENTIFIED BY '123456';
学习环境可以这样做。
但是：生产环境绝对不要使用 123456 这种弱密码

十九、第一次 SQL 测试（这里我给出了一个简单的Sql语句例子，旨在测试用）

登录：
obclient[root@sys][(none)]>

执行：
SELECT VERSION();
然后创建数据库：
CREATE DATABASE test_db;
进入数据库：
USE test_db;
创建表：
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    score INT
);
插入数据：
INSERT INTO students VALUES
(1, '张三', 90),
(2, '李四', 85),
(3, '王五', 95);
查询：
SELECT * FROM students;
应该得到类似：

+----+------+-------+
| id | name | score |
+----+------+-------+
|  1 | 张三 |    90 |
|  2 | 李四 |    85 |
|  3 | 王五 |    95 |
+----+------+-------+

到这里，你已经完成了：
安装
 ↓
部署
 ↓
启动
 ↓
登录
 ↓
创建数据库
 ↓
创建数据表
 ↓
插入数据
 ↓
查询数据

二十、以后如何启动 OceanBase？

每次打开 Windows 后：
1. 进入 Ubuntu
PowerShell：
wsl -d Ubuntu-22.04
2. 检查 OceanBase 是否已经运行
直接尝试：
obclient -h127.0.0.1 -P2881 -uroot@sys -p
如果能够登录：
obclient[root@sys][(none)]>
说明 OceanBase 已经在运行。不需要再次执行：
obd cluster start demo

二十一、如果 OceanBase 没有运行

执行：
obd cluster start demo
然后再次：
obclient -h127.0.0.1 -P2881 -uroot@sys -p

二十二、如果浏览器出现 ChunkLoadError

使用 OceanBase 网页管理界面时，可能遇到类似：
ChunkLoadError:
Loading chunk xxx failed
例如：
missing:
http://localhost:2886/xxxxx.js
这种错误属于 Web 前端资源加载问题。
首先确认数据库：
obclient -h127.0.0.1 -P2881 -uroot@sys -p
如果可以连接，不要因为网页报错就重新安装 OceanBase。可以后续单独排查网页缓存、OBProxy/OCP Express 或相关 Web 组件

二十三、Windows 与 Ubuntu 命令不要搞混

这是 WSL2 初学者非常容易遇到的问题。
Windows PowerShell
例如：
wsl -l -v
或者：
wsl -d Ubuntu-22.04

二十四、如何停止 OceanBase？

停止集群：
obd cluster stop demo

再次启动：
obd cluster start demo

二十六、如何查看集群状态？

obd cluster list
以及：
obd cluster display demo

二十五、最常用命令速查
进入 Ubuntu
wsl -d Ubuntu-22.04
查看 WSL
wsl -l -v
查看 OceanBase 集群
obd cluster list
查看详细状态
obd cluster display demo
启动
obd cluster start demo
停止
obd cluster stop demo
登录 OceanBase
obclient -h127.0.0.1 -P2881 -uroot@sys -p
查看版本
进入 OBClient 后：
SELECT VERSION();

二十六、启动流程
以后真正使用时，可以使用下面这套流程：
Windows
   │
   ├── PowerShell
   │
   └── wsl -d Ubuntu-22.04
            │
            ▼
        Ubuntu 22.04
            │
            ├── obd cluster list
            │
            ├── 如果未运行
            │       │
            │       ▼
            │   obd cluster start demo
            │
            ▼
       OceanBase Observer
            │
            │ 2881
            ▼
         OBClient
            │
            ▼
      执行 SQL / 学习


二十七、安装成功的最终判断标准

不要只看：
obd cluster list
是否出现：demo
真正建议执行：

obclient -h127.0.0.1 -P2881 -uroot@sys -p

如果看到：
Welcome to the OceanBase.
并且：
Server version: OceanBase_CE 4.5.0.0
最后：
obclient[root@sys][(none)]>
那么就可以认为：
OceanBase 已经成功搭建并可以开始使用。

二十八、接下来学什么？

假如你的目标是学习 OceanBase 数据库，或者参加 OceanBase 数据库相关比赛，可以按照下面的路线继续：
第一阶段
Linux / WSL
    ↓
OceanBase 安装
    ↓
OBD
    ↓
OBClient

第二阶段
SQL 基础
    ↓
CREATE
    ↓
INSERT
    ↓
SELECT
    ↓
UPDATE
    ↓
DELETE

第三阶段
多表查询
    ↓
JOIN
    ↓
GROUP BY
    ↓
ORDER BY
    ↓
子查询

第四阶段
数据库原理
    ↓
索引
    ↓
事务
    ↓
锁
    ↓
执行计划

第五阶段
SQL 性能优化
    ↓
EXPLAIN
    ↓
索引优化
    ↓
慢 SQL
    ↓
执行计划分析

第六阶段
OceanBase 专项
    ↓
MySQL 模式
    ↓
Oracle 模式
    ↓
分区
    ↓
租户
    ↓
资源管理
    ↓
分布式数据库特性

第七阶段
OceanBase 数据库大赛
    ↓
赛题分析
    ↓
SQL 优化
    ↓
性能测试
    ↓
Benchmark
    ↓
最终优化
（上面所列出的是我的个人建议，如果已有规划请按照自己的规划来学）

二十九、注意事项
1.本教程主要用于学习和本地体验。
2.WSL2 单机部署不等于生产环境部署。
3.不要把真实数据库密码提交到 GitHub。
4.不要把 root 密码写进公开配置文件。
5.不要把 123456 用于生产环境。
6.如果机器资源不足，OceanBase 可能因为内存不足启动失败。
7.如果端口被占用，先检查具体组件，不要直接重装。
8.deployed 表示部署完成，不一定代表正在运行。
9.最终应该通过 2881 实际连接来确认 Observer 是否正常。
10.WSL、Ubuntu、OBD、OceanBase、OBProxy、Grafana 是不同层次的组件，不要把它们混为一谈。

最后希望大家学习OceanBase顺利，本文档有什么不足之处希望大家指出，谢谢大家

参考文献：
OceanBase:快速体验OceanBase 社区版4.5.0 ：https://www.oceanbase.com/docs/common-oceanbase-database-cn-1000000004475429?utm_source=chatgpt.com
OceanBase：单机部署 OceanBase 集群： https://www.oceanbase.com/docs/community-obd-cn-1000000002023460?utm_source=chatgpt.com
OceanBase：快速部署命令 obd demo   https://www.oceanbase.com/docs/common-obd-cn-1000000000314353?utm_source=chatgpt.com
OceanBase: OBClient 文档  https://www.oceanbase.com/en/docs/common-obclient-doc-cn-1000000006361920?utm_source=chatgpt.com
