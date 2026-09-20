# 🧭 OceanBase-Learning

> A practical beginner guide for learning OceanBase on Windows with WSL2.
> 一个面向初学者的 OceanBase 学习与本地部署教程，帮助 Windows 用户通过 WSL2 搭建 OceanBase 学习环境。

**Author:** `010no010`

> 本教程主要用于学习 OceanBase 基础知识、本地环境搭建以及参加 OceanBase 数据库相关竞赛。
>
> 本文内容主要来自个人实际安装与学习过程，属于个人经验总结。如果文档中存在错误或不完善的地方，欢迎提出指正。

---

## ✨ 项目愿景 / Vision

本项目希望帮助第一次接触 OceanBase 的学习者完成：

```text
Windows
   ↓
WSL2
   ↓
Ubuntu 22.04
   ↓
OceanBase CE
   ↓
OBD
   ↓
OBClient
   ↓
SQL
```

最终目标是：

> 从一个全新的 Windows 环境开始，完成 OceanBase 的安装、部署、启动，并能够进入 `obclient` 执行 SQL。

本教程特别适合：

* 第一次接触 OceanBase 的初学者
* 使用 Windows 电脑进行学习的用户
* 想参加 OceanBase 数据库相关竞赛的学习者
* 想了解 WSL2 + Linux + OceanBase 基础部署流程的用户

---

## 🚀 当前内容 / Features

目前教程覆盖：

* Windows + WSL2 环境准备
* Ubuntu 22.04 安装与检查
* OceanBase All-in-One 安装
* OBD 基础使用
* OceanBase 单节点学习环境部署
* Observer 启动与状态检查
* OBClient 登录
* Root 密码修改
* 基础 SQL 测试
* OceanBase 常用命令
* Grafana 启动异常排查
* `memory_limit` 配置问题排查
* `ChunkLoadError` 网页问题说明
* Windows / Ubuntu 命令区别
* OceanBase 后续学习路线

---

## 🧱 环境架构 / Architecture

本教程最终环境：

```text
Windows
└── WSL2
    └── Ubuntu 22.04
        ├── OceanBase CE 4.5.0
        ├── OBD
        ├── OBClient
        ├── OBProxy
        ├── OBAgent
        ├── Prometheus
        └── Grafana
```

其中最重要的是：

```text
OceanBase Observer
        ↓
      2881
        ↓
    OBClient
```

只要能够通过 `2881` 连接到 Observer，就可以开始使用 OceanBase。

---

## 🖥️ 为什么使用 WSL2？

OceanBase 主要运行在 Linux 环境中，而 WSL2 可以让 Windows 用户在不更换操作系统的情况下运行 Linux 环境。

简单理解：

```text
Windows
   ↓
  WSL2
   ↓
Ubuntu 22.04
   ↓
OceanBase
```

相比传统虚拟机，WSL2 更适合本地学习和开发场景。

---

## 📦 安装方式 / Installation

### 1. Windows 环境要求

建议使用：

| 项目       | 建议                      |
| -------- | ----------------------- |
| 操作系统     | Windows 10 / Windows 11 |
| CPU      | 至少 2 个 CPU 核心           |
| 内存       | 官方单机体验建议至少 6 GB 可用内存    |
| 磁盘       | 官方单机体验建议至少 20 GB 可用空间   |
| 虚拟化      | 开启 CPU 虚拟化              |
| Linux 环境 | WSL2                    |

> **注意：**
>
> 官方单机部署文档目前建议至少：
>
> * 2 vCPU
> * 6 GB 可用内存
> * 20 GB 可用磁盘
>
> 本教程实际使用过程中，为了适应资源较紧张的学习环境，使用过 `memory_limit: 4G` 的配置。
>
> `4G` 是本教程中的实践配置，并不代表官方最低要求，也不建议将此配置直接用于生产环境。

---

### 2. 安装 WSL2

以管理员身份打开 PowerShell：

```powershell
wsl --install
```

安装完成后重新启动 Windows。

检查 WSL：

```powershell
wsl -l -v
```

正常情况下可以看到类似：

```text
NAME            STATE           VERSION
Ubuntu-22.04    Running         2
```

如果 `VERSION` 是：

```text
2
```

说明当前使用的是 WSL2。

---

### 3. 进入 Ubuntu

在 Windows PowerShell 中执行：

```powershell
wsl -d Ubuntu-22.04
```

进入后类似：

```text
oceanbase@电脑名:~$
```

以后如果看到：

```text
$
```

并且前面是 Ubuntu 用户名，通常就说明已经进入 Linux 环境。

---

### 4. 检查 Ubuntu

检查系统版本：

```bash
cat /etc/os-release
```

如果是 Ubuntu 22.04，会看到：

```text
VERSION_ID="22.04"
```

检查磁盘：

```bash
df -h /
```

检查内存：

```bash
free -h
```

检查 CPU：

```bash
nproc
```

---

### 5. 安装基础依赖

更新软件源：

```bash
sudo apt update
```

安装基础工具：

```bash
sudo apt install -y curl wget ca-certificates
```

检查：

```bash
curl --version
```

---

### 6. 安装 OceanBase All-in-One

OceanBase 提供 All-in-One 安装方式，可以一次安装 OBD、OBClient 以及相关组件。

执行：

```bash
bash -c "$(curl -s https://obbusiness-private.oss-cn-shanghai.aliyuncs.com/download-center/opensource/oceanbase-all-in-one/installer.sh)"
```

如果当前终端找不到相关命令，可以执行：

```bash
source ~/.oceanbase-all-in-one/bin/env.sh
```

检查 OBD：

```bash
obd --version
```

检查 OBClient：

```bash
obclient --version
```

如果能够正常输出版本号，说明基础安装成功。

---

## ⚙️ OBD 部署 / Deployment

### 1. 检查 OBD

执行：

```bash
obd cluster list
```

第一次安装时可能没有任何集群。

这是正常的。

---

### 2. 快速部署方式

OceanBase 官方提供：

```bash
obd demo
```

用于快速部署单节点 OceanBase。

如果只是想快速体验 OceanBase，可以优先考虑这种方式。

---

### 3. 配置文件部署

如果希望更加深入地理解 OceanBase 的配置，可以使用 OBD 配置文件进行部署。

本教程采用配置文件方式作为学习示例。

创建目录：

```bash
mkdir -p ~/ob-demo
cd ~/ob-demo
```

创建配置文件：

```bash
nano config.yaml
```

参考配置：

```yaml
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
```

> **注意：**
>
> 上面的配置主要用于学习环境示例。
>
> `memory_limit: 4G` 是针对资源比较紧张的本地学习环境的实践配置，不应该机械地复制到生产环境。
>
> 如果机器资源充足，应根据实际环境和 OceanBase 官方部署要求调整资源配置。

---

## 🧪 Testing / 测试方式

### 1. 部署测试

执行：

```bash
obd cluster deploy demo -c config.yaml
```

然后：

```bash
obd cluster list
```

如果能够看到：

```text
demo
```

说明集群已经完成部署。

---

### 2. 启动测试

执行：

```bash
obd cluster start demo
```

重点观察：

```text
Start observer ok
observer program health check ok
Connect to observer 127.0.0.1:2881 ok
```

如果出现：

```text
observer program health check ok
```

以及：

```text
Connect to observer 127.0.0.1:2881 ok
```

说明 OceanBase Observer 已经能够正常工作。

---

### 3. 最终连接测试

执行：

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

如果成功进入：

```text
Welcome to the OceanBase.
```

并看到：

```text
Server version: OceanBase_CE 4.5.0.0
```

以及：

```text
obclient[root@sys][(none)]>
```

就可以认为 OceanBase 已经成功搭建并可以开始使用。

---

## 🧪 SQL 基础测试

登录：

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

查看版本：

```sql
SELECT VERSION();
```

创建数据库：

```sql
CREATE DATABASE test_db;
```

进入数据库：

```sql
USE test_db;
```

创建表：

```sql
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    score INT
);
```

插入数据：

```sql
INSERT INTO students VALUES
(1, '张三', 90),
(2, '李四', 85),
(3, '王五', 95);
```

查询：

```sql
SELECT * FROM students;
```

预期结果类似：

```text
+----+------+-------+
| id | name | score |
+----+------+-------+
|  1 | 张三 |    90 |
|  2 | 李四 |    85 |
|  3 | 王五 |    95 |
+----+------+-------+
```

到这里就完成了：

```text
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
```

---

## 🔐 Root 密码 / Root Password

第一次登录：

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

然后输入部署时设置的密码。

如果使用 OBD 部署，并且没有自己设置密码，可以检查配置：

```bash
grep -nE 'root_password|password' \
/home/oceanbase/.obd/cluster/demo/config.yaml
```

> 不要将真实的 `root_password` 提交到 GitHub。

---

### 修改 Root 密码

进入 OceanBase：

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

执行：

```sql
ALTER USER root IDENTIFIED BY '你的新密码';
```

例如：

```sql
ALTER USER root IDENTIFIED BY '123456';
```

仅建议在本地学习环境中使用简单密码。

**生产环境不要使用 `123456` 等弱密码。**

---

## 🛠️ 常见问题 / Troubleshooting

### `memory_limit` 配置错误

某些环境下，如果配置：

```yaml
memory_limit: 3G
```

可能出现：

```text
Invalid config
name=memory_limit
value=3G
ret=-4147
```

最终出现：

```text
OB_INVALID_CONFIG
```

可以检查：

```bash
grep -E 'memory_limit|system_memory' \
/home/oceanbase/.obd/cluster/demo/config.yaml
```

如果确认是 Observer 因内存配置启动失败，应根据当前机器资源调整配置。

---

### Grafana 启动失败

可能出现：

```text
Failed to start 127.0.0.1 grafana
```

或者：

```text
grafana program health check
```

长时间没有通过。

这不一定意味着 OceanBase 数据库本身启动失败。

优先检查：

```text
observer program health check ok
```

以及：

```text
Connect to observer 127.0.0.1:2881 ok
```

如果 Observer 正常，并且 `2881` 可以连接，那么可以先继续学习 OceanBase。

Grafana 属于监控可视化组件，不是当前 SQL 学习的核心组件。

---

### `deployed` 不代表正在运行

执行：

```bash
obd cluster list
```

如果看到：

```text
demo | deployed
```

这里的：

```text
deployed
```

表示：

> 集群已经完成部署。

它不等于：

> 集群当前正在运行。

可以使用：

```bash
obd cluster display demo
```

查看详细状态。

最终仍然建议实际连接：

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

---

### 浏览器出现 `ChunkLoadError`

使用 OceanBase Web 管理界面时，可能遇到：

```text
ChunkLoadError:
Loading chunk xxx failed
```

例如：

```text
missing:
http://localhost:2886/xxxxx.js
```

这种情况属于 Web 前端资源加载问题。

首先确认数据库：

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

如果可以正常连接，不要因为网页报错就直接重新安装 OceanBase。

可以后续单独排查：

* 浏览器缓存
* OBProxy
* OCP Express
* Web 前端组件

---

## 🖥️ Windows 与 Ubuntu 命令区别

WSL2 初学者非常容易把两个环境混淆。

### Windows PowerShell

例如：

```powershell
wsl -l -v
```

或者：

```powershell
wsl -d Ubuntu-22.04
```

### Ubuntu

例如：

```bash
obd cluster list
```

```bash
obd cluster start demo
```

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

简单理解：

```text
Windows
   ↓
进入 WSL2
   ↓
Ubuntu
   ↓
运行 OceanBase 命令
```

---

## 🔄 日常使用流程

以后每次打开 Windows，可以使用：

```powershell
wsl -d Ubuntu-22.04
```

进入 Ubuntu。

然后首先尝试：

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

如果能够连接：

```text
obclient[root@sys][(none)]>
```

说明 OceanBase 已经运行，不需要再次执行启动命令。

如果无法连接，再执行：

```bash
obd cluster start demo
```

然后重新连接：

```bash
obclient -h127.0.0.1 -P2881 -uroot@sys -p
```

---

## ⏹️ 停止 OceanBase

停止集群：

```bash
obd cluster stop demo
```

再次启动：

```bash
obd cluster start demo
```

---

## 🔎 常用命令速查

| 操作           | 命令                                          |
| ------------ | ------------------------------------------- |
| 查看 WSL       | `wsl -l -v`                                 |
| 进入 Ubuntu    | `wsl -d Ubuntu-22.04`                       |
| 查看集群         | `obd cluster list`                          |
| 查看详细状态       | `obd cluster display demo`                  |
| 启动集群         | `obd cluster start demo`                    |
| 停止集群         | `obd cluster stop demo`                     |
| 登录 OceanBase | `obclient -h127.0.0.1 -P2881 -uroot@sys -p` |
| 查看版本         | `SELECT VERSION();`                         |

---

## 🧭 完整启动流程

```text
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
```

---

## 🗺️ 学习路线 / Roadmap

### 第一阶段：基础环境

```text
Linux / WSL
    ↓
OceanBase 安装
    ↓
OBD
    ↓
OBClient
```

### 第二阶段：SQL 基础

```text
CREATE
  ↓
INSERT
  ↓
SELECT
  ↓
UPDATE
  ↓
DELETE
```

### 第三阶段：SQL 进阶

```text
JOIN
  ↓
GROUP BY
  ↓
ORDER BY
  ↓
子查询
```

### 第四阶段：数据库原理

```text
索引
 ↓
事务
 ↓
锁
 ↓
执行计划
```

### 第五阶段：SQL 性能优化

```text
EXPLAIN
 ↓
索引优化
 ↓
慢 SQL
 ↓
执行计划分析
```

### 第六阶段：OceanBase 专项

```text
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
```

### 第七阶段：OceanBase 数据库大赛

```text
赛题分析
    ↓
SQL 优化
    ↓
性能测试
    ↓
Benchmark
    ↓
最终优化
```

> 以上路线属于个人学习建议。
>
> 如果已经有自己的学习规划，可以按照自己的计划进行。

---

## 🔐 安全提醒 / Safety Notes

本项目主要用于学习和本地体验。

请注意：

* 不要将真实数据库密码提交到 GitHub。
* 不要把 `root_password` 写入公开配置文件。
* 不要提交 `.env`、密钥、Token 或其他敏感信息。
* 不要在生产环境使用 `123456` 等弱密码。
* 修改真实配置前建议先进行备份。
* WSL2 单机环境不等于生产环境。
* 如果机器资源不足，OceanBase 可能因为内存不足启动失败。
* 如果端口被占用，应先确认具体组件，不要直接重装。
* `deployed` 表示已经部署，不一定代表正在运行。
* 最终应该通过 `2881` 实际连接确认 Observer 是否正常。

---

## 📖 Documentation / 相关文档

本项目建议按照以下文档继续扩展：

```text
docs/
├── ARCHITECTURE.zh-CN.md
├── TESTING.zh-CN.md
├── ROADMAP.zh-CN.md
└── TROUBLESHOOTING.zh-CN.md
```

后续可以将较长的安装、排错和比赛学习内容逐步拆分到 `docs/` 中，让 README 保持简洁。

---

## 📁 Repository Structure / 仓库结构

推荐最终结构：

```text
OceanBase-Learning/
├── README.md
├── README.zh-CN.md
├── LICENSE
├── docs/
│   ├── ARCHITECTURE.zh-CN.md
│   ├── TESTING.zh-CN.md
│   ├── ROADMAP.zh-CN.md
│   └── TROUBLESHOOTING.zh-CN.md
├── examples/
│   └── config.yaml.example
└── .gitignore
```

其中：

| 文件 / 目录           | 用途       |
| ----------------- | -------- |
| `README.md`       | 英文主文档    |
| `README.zh-CN.md` | 中文文档     |
| `docs/`           | 详细说明     |
| `examples/`       | 示例配置     |
| `LICENSE`         | 项目许可证    |
| `.gitignore`      | Git 忽略规则 |

---

## 📝 Commit Message / 提交规范

建议使用 Conventional Commits 风格。

| 类型         | 用途   | 示例                                          |
| ---------- | ---- | ------------------------------------------- |
| `feat`     | 新内容  | `feat: add OceanBase SQL examples`          |
| `fix`      | 修复错误 | `fix: correct WSL installation command`     |
| `docs`     | 文档修改 | `docs: update OceanBase installation guide` |
| `test`     | 测试   | `test: add SQL smoke test`                  |
| `refactor` | 重构   | `refactor: reorganize documentation`        |
| `chore`    | 日常维护 | `chore: update gitignore`                   |

例如：

```bash
git add .
git commit -m "docs: improve OceanBase installation guide"
```

---

## 🧪 发布前检查 / Pre-release Checklist

发布或提交 README 前建议检查：

```text
[ ] 安装命令可以正常执行
[ ] WSL2 环境说明准确
[ ] OceanBase 版本信息准确
[ ] SQL 示例可以执行
[ ] 没有提交真实密码
[ ] 没有提交 Token 或密钥
[ ] 配置文件中的密码已经替换
[ ] 常见问题说明完整
[ ] README 结构清晰
[ ] LICENSE 已确定
```

---

## 📜 License / 许可证

本项目当前许可证：

> **待确定**

如果后续希望公开发布，可以根据项目需求选择合适的开源许可证，例如 MIT License、Apache License 2.0 等。

在正式添加许可证之前，请不要在 README 中声称项目已经采用某个 License。

---

## 🙏 Feedback / 反馈

如果你在按照本教程安装 OceanBase 的过程中遇到问题，欢迎提出：

* 安装失败
* OBD 部署问题
* Observer 启动问题
* SQL 问题
* WSL2 环境问题
* 文档错误
* 教程遗漏

也欢迎指出本文档中的错误和不足。

本项目会根据实际学习和使用过程持续完善。

---

## 📚 References / 参考资料

* OceanBase 官方文档：快速体验 OceanBase 社区版 4.5.0
* OceanBase 官方文档：单机部署 OceanBase 集群
* OceanBase 官方文档：快速部署命令 `obd demo`
* OceanBase 官方文档：OBClient 文档

---

## 🌟 Personal Note / 个人备注

> 一个好的学习项目不只是把软件安装成功。

```text
真实问题优先
默认更安全
先解释再执行
先备份再修改
先测试再发布
写文档给未来的自己
通过真实反馈持续改进
```

希望这份教程能够帮助更多第一次接触 OceanBase 的学习者。


**如果本文档存在错误或不足，欢迎指出。**

**感谢阅读。**
