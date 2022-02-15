---
title: Docker
date: 2021-05-25
categories: [Docker]
toc: true
---



## Docker概述

### Docker为什么会出现

一款产品：生产- -上线 两套环境，应用环境，应用配置

**问题**：我在我的电脑上可以运行！

Java → jar（环境） → 打包项目带上环境（镜像）→ Docker仓库 → 下载发布的镜像 → 直接运行

![image-20211119151050326](../../.vuepress/public/images/image-20211119151050326.png)

Docker 的思想来自于集装箱

**隔离**：Docker 的核心思想！每个箱子是互相隔离的！

Docker 通过隔离机制，可以将服务器利用到极致！

Docker 是基于 Go 语言开发的开源项目！

**官网**：[https://www.docker.com/](https://www.docker.com/)

**文档地址**：[https://docs.docker.com/](https://docs.docker.com/) （文档非常详细）

**仓库地址**：[https://hub.docker.com/](https://hub.docker.com/)

### Docker 能干嘛

> 之前的虚拟机技术

![image-20211119151138878](../../.vuepress/public/images/image-20211119151138878.png)

**虚拟机技术缺点**：

1. 资源占用十分多
2. 冗余步骤多
3. 启动很慢！

> 容器化技术

**容器化技术不是模拟一个完整的操作系统**

![image-20211119151209886](../../.vuepress/public/images/image-20211119151209886.png)

比较 Docker 和 虚拟机 的不同：

- 传统虚拟机，虚拟出一条硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件
- 容器内的应用直接运行在主机的内核中，容器没有自己的内核，也没有虚拟我们的硬件，所以比较轻便
- 每个容器都是互相隔离的，每个容器内都有一个属于自己的文件系统，互不影响

> DevOps（开发、运维）

##### 应用更快速的交付和部署

传统：一堆帮助文档，安装程序

Docker：打包镜像发布测试，一键运行

##### 更便捷的升级和扩缩容

使用Docker之后，部署应用和搭积木一样！

##### 更简单的运维系统

在容器化之后，我们的开发，测试环境都是高度一致的

##### 更高效的计算资源利用

Docker是内核级别的虚拟化，可以在一个虚拟机上运行很多的容器实例，服务器的性能可以被压榨到极致！

## Docker安装

### Docker的基本组成

![image-20211119151232167](../../.vuepress/public/images/image-20211119151232167.png)

**镜像（image）：**

Docker 镜像就好比是一个模版，可以通过这个模版创建容器服务，tomcat 镜像 ⇒ run ⇒ tomcat01 容器（提供服务），通过这个镜像可以创建多个容器。

**容器（container）：**

Docker 利用容器技术，可以独立运行一个或一组应用，通过镜像来创建。

**仓库（respository）：**

仓库就是存放镜像的地方！

仓库分为共有仓库和私有仓库！

### 安装Docker

> 环境准备

- 需要 Linux 基础
- CentOS 7

> 环境查看

```shell
# 系统内核 3.10 以上
uname -r
3.10.0-1062.18.1.el7.x86_64
```

```shell
# 系统版本
cat /etc/os-release 
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

> 安装

```bash
# 1.卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 2.需要的安装包
yum install -y yum-utils

# 3.设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo # 默认是国外的！

yum-config-manager \
    --add-repo \
		https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 阿里云的，速度快（推荐）

# 更新yum软件包索引
yum makecache fast

# 4.安装docker相关内容 docker-ce 社区版 ee 企业版
yum install docker-ce docker-ce-cli containerd.io

# 5.启动docker
systemctl start docker

# 6.查看docker 版本
docker version

# 6.测试hello-world
docker run hello-world

# 7.查看下载的 hello-world 镜像
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        9 months ago        13.3kB
```

> 卸载Docker

```bash
# 1.卸载依赖
yum remove docker-ce docker-ce-cli containerd.io

# 2.删除资源
rm -rf /var/lib/docker
```

### 阿里云镜像加速

```bash
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ztlvr8p4.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
```

### 运行原理

![image-20210527143507647](D:\SynologyDrive\allen\img\image-20210527143507647.png)

### 底层原理

**Docker是怎么工作的？**

Docker 是一个 Clinet-Server 结构的系统，Docker 的守护进程运行在主机上，通过 Socket 从客户端访问！

DockerServer 接收到 Client-Clinet 的指令，就会执行这个命令！

![](D:\SynologyDrive\allen\img\Snipaste_2021-05-27_14-40-59.png)

**Docker为什么虚拟机快？**

1. 有着比虚拟机更少的抽象层
2. 利用的是宿主机的内核，虚拟机需要的是Guest OS

![](D:\SynologyDrive\allen\img\Snipaste_2021-05-27_14-43-10.png)

所以说，新建一个容器的时候，Docker不需要像虚拟机一样重新加载一个操作系统内核，避免引导。虚拟机是加载Guest OS，分钟级别的，而Docker是利用宿主机的操作系统，省略了这个复杂的过程，秒级！

![](D:\SynologyDrive\allen\img\Snipaste_2021-05-27_14-46-09.png)



## Docker的常用命令

### 帮助命令

```bash
# 显示docker的版本信息
docker version

# 显示docker的系统信息，包括镜像和容器的数量
docker info

# 帮助命令
docker 命令 --help
```

帮助文档的地址：[https://docs.docker.com/reference/](https://docs.docker.com/reference/)



### 镜像命令

**docker images 查看所有本地的主机上的镜像**

```bash
[root@AllenCloud ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        9 months ago        13.3kB

# 解释
REPOSITORY 镜像的仓库源
TAG        镜像的标签
IMAGE ID   镜像的ID
CREATED    镜像的创建时间
SIZE       镜像的大小

# 可选项
  -a, --all             # 列出所有的镜像
  -q, --quiet           # 只显示镜像的ID
```

**docker search 搜索镜像**

```bash
[root@AllenCloud ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10103               [OK]                
mariadb                           MariaDB is a community-developed fork of MyS…   3708                [OK]

# 可选性，通过收藏过滤
 --filter=STARS=3000   # 搜索大于STARS3000的镜像
 [root@AllenCloud ~]# docker search mysql --filter=stars=3000
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql               MySQL is a widely used, open-source relation…   10921               [OK]                
mariadb             MariaDB Server is a high performing open sou…   4125                [OK]
```

**docker pull 下载镜像**

```bash
# 下载镜像 docker pull 镜像名[:tag]
[root@AllenCloud ~]# docker pull mysql
Using default tag: latest   # 如果不指定tag，默认就是 latest
latest: Pulling from library/mysql
bb79b6b2107f: Pull complete # 分层下载，docker image的核心，联合文件系统
49e22f6fb9f7: Pull complete 
842b1255668c: Pull complete 
9f48d1f43000: Pull complete 
c693f0615bce: Pull complete 
8a621b9dbed2: Pull complete 
0807d32aef13: Pull complete 
a56aca0feb17: Pull complete 
de9d45fd0f07: Pull complete 
1d68a49161cc: Pull complete 
d16d318b774e: Pull complete 
49e112c55976: Pull complete 
Digest: sha256:8c17271df53ee3b843d6e16d46cff13f22c9c04d6982eb15a9a47bd5c9ac7e2d # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest   # 真实地址

# 俩等价
docker pull mysql
docker pull docker.io/library/mysql:latest

# 指定版本下载
[root@AllenCloud ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
bb79b6b2107f: Already exists 
49e22f6fb9f7: Already exists 
842b1255668c: Already exists 
9f48d1f43000: Already exists 
c693f0615bce: Already exists 
8a621b9dbed2: Already exists 
0807d32aef13: Already exists 
f15d42f48bd9: Pull complete 
098ceecc0c8d: Pull complete 
b6fead9737bc: Pull complete 
351d223d3d76: Pull complete 
Digest: sha256:4d2b34e99c14edb99cdd95ddad4d9aa7ea3f2c4405ff0c3509a29dc40bcb10ef
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

```shell
[root@AllenCloud ~]# docker images mysql
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest              c0cdc95609f1        2 weeks ago         556MB
mysql               5.7                 a70d36bc331a        4 months ago        449MB
```

**docker rmi 删除镜像**

```bash
[root@AllenCloud ~]# docker rmi -f 镜像ID   # 删除指定的镜像
[root@AllenCloud ~]# docker rmi -f 镜像ID 镜像ID 镜像ID   # 删除多个镜像
[root@AllenCloud ~]# docker rmi -f $(docker images -aq)   # 删除全部镜像
```



### 容器命令

说明：我们有了镜像才可以创建容器，Linux，下载一个centos镜像测试

```bash
docker pull centos
```

**新建容器并启动**

```bash
docker run [可选参数] image

# 参数说明
-- name="Name"   容器名称 例如 tomcat01，用来区分容器
-d               后台方式运行
-it              使用交互方式运行，进入容器查看内容
-p               指定容器的端口 -p 8080:8080
		-p ip:主机端口:容器端口
		-p 主机端口:容器端口 （常用）
		-p 容器端口
		容器端口
-p               随机指定端口

# 测试，启动并进入容器
[root@AllenCloud ~]# docker run -it centos /bin/bash
[root@8f9882e43593 /]# ls   # 查看容器内的centos基础版本，很多命令都是不完善的
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 退出容器
[root@99733ca27eee /]# exit 
exit

[root@AllenCloud /]# ls
bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  www
```

**列出所有运行中的容器**

```bash
# docker ps 命令
        # 列出当前正在运行的容器
-a      # 列出当前正在运行的容器+带出历史运行过的容器
-n=？   # 显示最近创建的容器
-q      # 只显示容器的编号

[root@AllenCloud /]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@AllenCloud /]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
99733ca27eee        centos              "/bin/bash"         About a minute ago   Exited (0) About a minute ago                       objective_diffie
8f9882e43593        centos              "/bin/bash"         4 minutes ago        Exited (0) 3 minutes ago                            unruffled_kilby
695e7bf207bd        bf756fb1ae65        "/hello"            About an hour ago    Exited (0) About an hour ago                        blissful_lovelace
```

**退出容器**

```bash
exit   # 直接容器停止并退出
ctrl + p + q   #容器不停止退出
```

**删除容器**

```bash
docker rm 容器ID                # 删除指定容器，不能删除正在运行的容器，如强制删除 rm -f
docker rm -f $(docker ps -aq)  # 删除所有容器
docker ps -a -q|xargs docker rm   # 删除所有的容器
```

**启动和停止容器的操作**

```bash
docker start 容器ID         # 启动容器
docker restart 容器ID       # 重启容器
docker stop 容器ID          # 停止当前正在运行的容器
docker kill 容器ID          # 强制停止容器
```

### 常用其他命令

**后台启动容器**

```bash
# 命令 docker run -d 镜像名
[root@AllenCloud /]# docker run -d centos

# 问题 docker ps 发现centos停止了

# 常见的坑，docker 容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
# nginx容器启动后，发现自己没有提供服务，就会立刻停止，就没有程序了
```

**查看日志**

```bash
# 显示指定条数日志
		-tf      # 显示日志
		-- tail num # 要显示日志的条数
docker logs -tf -tail 条数 容器ID

# 自己编写一个shell脚本
docker run -d centos /bin/sh -c "while ture;do echo Allen;sleep 1;done"
```

**查看容器中进程信息**

```bash
docker top 容器ID
```

**查看镜像容器的源数据**

```bash
docker inspect 容器ID

[root@AllenCloud ~]# docker inspect 7a037d59f2c0
[
    {
        "Id": "7a037d59f2c020115bd690f201f0ebcc438c611eec5423213c65b7c995f87a6f",
        "Created": "2020-10-28T02:28:25.103158773Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while ture;do echo Allen;sleep 1;done"
        ],
        "State": {
            "Status": "exited",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-10-28T02:28:25.368732568Z",
            "FinishedAt": "2020-10-28T02:28:25.365900855Z"
        },
        "Image": "sha256:0d120b6ccaa8c5e149176798b3501d4dd1885f961922497cd0abef155c869566",
        "ResolvConfPath": "/var/lib/docker/containers/7a037d59f2c020115bd690f201f0ebcc438c611eec5423213c65b7c995f87a6f/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/7a037d59f2c020115bd690f201f0ebcc438c611eec5423213c65b7c995f87a6f/hostname",
        "HostsPath": "/var/lib/docker/containers/7a037d59f2c020115bd690f201f0ebcc438c611eec5423213c65b7c995f87a6f/hosts",
        "LogPath": "/var/lib/docker/containers/7a037d59f2c020115bd690f201f0ebcc438c611eec5423213c65b7c995f87a6f/7a037d59f2c020115bd690f201f0ebcc438c611eec5423213c65b7c995f87a6f-json.log",
        "Name": "/frosty_benz",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/09483bc565df823e84f6332bdebef0388766eeb5a3eb3f8da5ceffd3ece8b5bb-init/diff:/var/lib/docker/overlay2/909695029be294db1f7b4904da0ef4fe5d3a0e54432ee5cb8dc4b12229991098/diff",
                "MergedDir": "/var/lib/docker/overlay2/09483bc565df823e84f6332bdebef0388766eeb5a3eb3f8da5ceffd3ece8b5bb/merged",
                "UpperDir": "/var/lib/docker/overlay2/09483bc565df823e84f6332bdebef0388766eeb5a3eb3f8da5ceffd3ece8b5bb/diff",
                "WorkDir": "/var/lib/docker/overlay2/09483bc565df823e84f6332bdebef0388766eeb5a3eb3f8da5ceffd3ece8b5bb/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "7a037d59f2c0",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while ture;do echo Allen;sleep 1;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20200809",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "56f4f481806c3f6305b503ac898757a7ab10f5342d17211ead69f6b6066c6395",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/56f4f481806c",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "9422f013ca9b06d7d435af29451e0f54191eb875c80c3ccc6ed041e83b4030f6",
                    "EndpointID": "",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

**进入当前正在运行的容器命令**

```bash
docker exec -it 容器ID   # 开启一个新的终端，可以在里面操作

docker attach 容器ID     # 进入容器正在执行的终端，不会打开新的进程！
```

**从容器内拷贝文件到主机上**

```bash
docker cp 容器ID:容器内文件路径 目的地主机路径

[root@AllenCloud home]# docker cp 12cea0a5c430:/home/test.java /home/
[root@AllenCloud home]# ls
Allen.java  test.java
```

## 小结

![](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/Untitled%201.png)

```bash
# 当前shell下，attach连接指定运行镜像
attach      Attach local standard input, output, and error streams to a running container
# 通过Dockerfile定制镜像
build       Build an image from a Dockerfile
# 提交当前容器为新的镜像
commit      Create a new image from a container's changes
# 从容器中拷贝指定文件或目录到宿主机上
cp          Copy files/folders between a container and the local filesystem
# 创建一个新的容器，同run，但不启动
create      Create a new container
# 查看容器变化
diff        Inspect changes to files or directories on a container's filesystem
# 从docker容器获取服务实时事件
events      Get real time events from the server
# 在已存在的容器上运行命令
exec        Run a command in a running container
# 导出容器的内容流为一个tar归档文件
export      Export a container's filesystem as a tar archive
# 展示一个镜像形成历史
history     Show the history of an image
# 列出系统当前镜像
images      List images
# 从tar包中的内容创建一个新的文件系统映射
import      Import the contents from a tarball to create a filesystem image
# 显示系统相关信息
info        Display system-wide information
# 查看容器详细信息
inspect     Return low-level information on Docker objects
# kill指定Docker容器
kill        Kill one or more running containers
# 从一个tar包中加载一个镜像
load        Load an image from a tar archive or STDIN
# 注册或登录一个Docker源服务器
login       Log in to a Docker registry
# 从当前Docker registry退出
logout      Log out from a Docker registry
# 输出当前容器日志信息
logs        Fetch the logs of a container
# 暂停容器暂停容器中所有的进程
pause       Pause all processes within one or more containers
# 查看映射端口对应的容器内部源端口
port        List port mappings or a specific mapping for the container
# 列出容器列表
ps          List containers
# 从Docker镜像源服务器中拉取指定镜像或库镜像
pull        Pull an image or a repository from a registry
# 推送指定镜像或库镜像到Docker镜像源服务器中
push        Push an image or a repository to a registry
# 重命名Docker容器名
rename      Rename a container
# 重启正在运行的容器
restart     Restart one or more containers
# 移除一个或多个容器
rm          Remove one or more containers
# 移除一个或多个容器[无容器使用该镜像才可以删除，否则需删除相关内容才可继续，或 -f 强制删除]
rmi         Remove one or more images
# 创建一个新容器并运行一个新命令
run         Run a command in a new container
# 保存一个镜像为一个tar包
save        Save one or more images to a tar archive (streamed to STDOUT by default)
# 在Docker Hub中搜索镜像
search      Search the Docker Hub for images
# 启动一个或多个已经被停止的容器
start       Start one or more stopped containers
# 显示容器使用的系统资源
stats       Display a live stream of container(s) resource usage statistics
# 停止一个运行中的容器
stop        Stop one or more running containers
# 标记本地镜像，将其归入某一仓库
tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
# 查看容器中运行的进程信息，支持 ps 命令参数
top         Display the running processes of a container
# 恢复容器中所有的进程
unpause     Unpause all processes within one or more containers
# 更新
update      Update configuration of one or more containers
# 显示 Docker 版本信息
version     Show the Docker version information
# 阻塞运行直到容器停止
wait        Block until one or more containers stop, then print their exit codes
```

> Docker安装Nginx

```bash
# 1.搜索镜像
docker search nginx

# 2.下载镜像
docker pull nginx

# 3.查看镜像
docker images

# 4.后台启动镜像，给容器命名，指定端口为30000
docker run -d --name nginx01 -p 30000:80 nginx

# 5.查看容器
docker ps

# 6.本机测试
curl localhost:30000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# 7.进入nginx
docker exec -it nginx01 /bin/bash
root@f47b058d460d:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@f47b058d460d:/# cd /etc/nginx/
root@f47b058d460d:/etc/nginx# ls
conf.d  fastcgi_params  koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params  uwsgi_params  win-utf

# 8.停止nginx
docker stop nginx01
```

端口暴露的概念

![](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/Untitled%202.png)

> Docker 安装 tomcat

```bash
docker run -it --rm tomcat:9.0   # 一般用来测试，用完即删！

# 1.下载tomcat
docker pull tomcat

# 2.后台启动
docker run -d --name tomcat01 -p 30001:8080 tomcat

# 3.进入容器
docker exec -it tomcat01 /bin/bash

# 4.发现问题，Linux命令少了，webapps没有项目，阿里云镜像的原因，默认最小的镜像，所有不必要的都剔除掉！
# 保证最小可使用环境
root@7d79330227b3:/usr/local/tomcat# ls -al
total 172
drwxr-xr-x 1 root root  4096 Oct 23 01:09 .
drwxr-xr-x 1 root root  4096 Oct 28 08:05 ..
-rw-r--r-- 1 root root 18982 Oct  6 14:17 BUILDING.txt
-rw-r--r-- 1 root root  5409 Oct  6 14:17 CONTRIBUTING.md
-rw-r--r-- 1 root root 57092 Oct  6 14:17 LICENSE
-rw-r--r-- 1 root root  2333 Oct  6 14:17 NOTICE
-rw-r--r-- 1 root root  3257 Oct  6 14:17 README.md
-rw-r--r-- 1 root root  6898 Oct  6 14:17 RELEASE-NOTES
-rw-r--r-- 1 root root 16262 Oct  6 14:17 RUNNING.txt
drwxr-xr-x 2 root root  4096 Oct 23 01:09 bin
drwxr-xr-x 1 root root  4096 Oct 28 08:05 conf
drwxr-xr-x 2 root root  4096 Oct 23 01:08 lib
drwxrwxrwx 1 root root  4096 Oct 28 08:05 logs
drwxr-xr-x 2 root root  4096 Oct 23 01:09 native-jni-lib
drwxrwxrwx 2 root root  4096 Oct 23 01:08 temp
drwxr-xr-x 2 root root  4096 Oct 23 01:08 webapps
drwxr-xr-x 7 root root  4096 Oct  6 14:14 webapps.dist
drwxrwxrwx 2 root root  4096 Oct  6 14:11 work
root@7d79330227b3:/usr/local/tomcat# cd webapps
root@7d79330227b3:/usr/local/tomcat/webapps# ls
root@7d79330227b3:/usr/local/tomcat/webapps#
```

> 安装ES + Kibana

```bash
# es暴露的端口很多！
# es十分耗内存！
# es的数据一般要放置到安全目录！
# --net somenetwork 网络配置
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch

# 启动后Linux卡 docker stats 查看cpu

# 增加内存限制
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms64m -Xmx512m" -e "discovery.type=single-node" elasticsearch

# 启动成功测试
[root@AllenCloud ~]# curl localhost:9200
{
  "name" : "biwey7U",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "GPFYryaoSv6YxASA_JTDhw",
  "version" : {
    "number" : "5.6.12",
    "build_hash" : "cfe3d9f",
    "build_date" : "2018-09-10T20:12:43.732Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

网络如何才能连接过去:

![](D:\SynologyDrive\allen\img\Snipaste_2021-05-31_15-28-34.png)

## 可视化

### portainer

Docker图形化界面管理工具，提供一个后台面板操作

```bash
docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

访问测试：[http://8.131.70.202:8088/](http://8.131.70.202:8088/)

### Rancher（CI/CD 可持续化集成和部署时用）



## Docker镜像

### 镜像是什么？

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于软件运行环境开发的软件，它包含运行某个软件所需要的所有内容，包括代码，运行时库、环境变量和配置文件。

所有的应用，直接打包docker镜像，就可以直接跑起来！

如何得到镜像：

- 从远程仓库下载
- 拷贝
- 自己制作镜像 DockerFile

### 镜像加载原理

> UnionFS（联合文件系统）

UnionFS（联合文件系统）：是一种分层、轻量级并且高效能的文件系统，它支持对文件系统的修改作为一次提交来一层层叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来的，这样最终的文件系统会包含所有底层的文件和目录。

> Docker镜像加载原理

Docker 的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS

bootfs（boot file system）主要包含 hootloader 和 kernel，bootloader 主要是引导加载 kernel，Linux 刚启动时会加载 bootfs 文件系统，在 Docker 镜像的最底层时 bootfs。这一层与我们典型的 Linux/Unix 系统是一样的，包含 boot 加载器和内核。当 boot 加载完成之后整个内核就都在内存中了，此时内存的使用权已由 bootfs 转交给内核，此时系统也会卸载 bootfs。

rootfs（root file system），在 bootfs 之上。包含的就是典型 Linux 系统中的 /dev，/proc，/bin，/etc等标准目录和文件。rootfs 就是各种不同的操作系统发行版，比如 Ubuntu，Centos 等等。

![](D:\SynologyDrive\allen\img\Snipaste_2021-05-31_16-09-45.png)

平时我们安装进虚拟机的 CentOS 都是好几个G，为什么 Docker 这里才200M？

![](D:\SynologyDrive\allen\img\Snipaste_2021-05-31_16-11-31.png)

对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令，工具和程序库就可以了，因为底层直接用 Host 的 kernel，自己只需要提供 rootfs 就可以了。由此可见对不不同的 Linux 发行版，bootfs 基本是一致的，rootfs 会由差别，因此不同的发行版可以公用 bootfs。



### 分层理解

