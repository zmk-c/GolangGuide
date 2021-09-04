![image-20210617193053116](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker.png)

## 一.Docker 概述

### 1.目前存在的问题

- 一款产品：开发-上线 需要两套环境！
- 开发和运维：我在我的电脑上能运行！在别人的电脑上不能运行，版本更新导致的不可用，对于运维人员的考验很大
- 环境的配置是十分麻烦的，每个机器都要部署环境（Redis集群，MySQL集群，Hadoop集群等），费时费力
- 发布一个项目(jar+(Redis MySQL jdk ES))，配置麻烦，不能够跨平台

Docker给以上问题提出了解决方案，Docker的思想来自于集装箱，会将所有需要的内容放到不同的集装箱中，谁需要这些环境就直接拿到这个集装箱就可以了，其中`隔离性`是Docker的核心思想，Docker在运行集装箱内的内容时，会在Linux的内核中，单独的开辟一片空间，这片空间不会影响到其他程序。

### 2.Docker 的历史

Docker 公司起初是一家名为 **dotCloud** 的平台即服务（Platform-as-a-Service, PaaS）提供商。底层技术上，dotCloud 平台利用了 Linux 容器技术。为了方便创建和管理这些容器，dotCloud 开发了一套内部工具，之后被命名为“Docker”。Docker就是这样诞生的！

2013年，dotCloud 的 PaaS 业务并不景气，公司需要寻求新的突破。于是他们聘请了 Ben Golub 作为新的 CEO，将公司重命名为“Docker”，放弃dotCloud PaaS 平台，怀揣着“将 Docker 和容器技术推向全世界”的使命，开启了一段新的征程。

如今 Docker 公司被普遍认为是一家创新型科技公司，据说其市场价值约为 10 亿美元。Docker 公司已经通过多轮融资，吸纳了来自硅谷的几家风投公司的累计超过 2.4 亿美元的投资。 

可以说现在Docker越来越火了！

### 3.Docker 的优点

比较Docker和虚拟机技术的不同：

- 传统虚拟机：虚拟出一套硬件，运行一个完整的操作系统，然后在这个操作系统上安装和运行软件
- Docker：采用`容器化`技术，容器是没有自己的内核的，也没有虚拟出硬件，容器内的应用直接运行在宿主机的内核中。

Docker有很多的优点：

1. 可以为部署在容器中的应用提供一致的运行时环境。使用Docker打包好的镜像在任何安装了Docker的服务器上正常运行，而无需考虑环境、配置、依赖等问题，可以做到快速部署项目，在容器化之后，我们的开发，测试环境是高度一致的。
3. 可以通过Docker来定制镜像，做到持续集成、交付和部署。
4. 结合微服务，将服务定制成镜像，可以做到负载的快速扩展。
5. 容器相对于实体系统更轻量，启动快速，只包含核心部分。
6. 在分布式部署和微服务方面应用很有潜力。

> Docker的资料

官网：[Empowering App Development for Developers | Docker](https://www.docker.com/)

![image-20210617193938992](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E5%AE%98%E7%BD%91.png)

文档地址：[Docker Documentation | Docker Documentation](https://docs.docker.com/)

仓库地址：[Docker Hub](https://hub.docker.com/)

## 二. Docker 安装

### 1.Docker 的基本组成

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg2018.cnblogs.com%2Fblog%2F680719%2F201811%2F680719-20181129220709127-1721830996.png&refer=http%3A%2F%2Fimg2018.cnblogs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1626523822&t=99d12efc17049fd8131beab65c3dc2e6)

**镜像（image）**：docker 镜像就好比一个模板，可以通过这个模板来创建容器服务，redis镜像=>run=>redis01容器（提供服务），通过镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中的）。

**容器（container）：**docker 利用容器技术，独立运行一个或者一组应用，通过镜像来创建的。

**仓库（repository）:**存放镜像的地方，分为公有仓库和私有仓库。

### 2.安装Docker

#### 前期准备

```shell
#安装的是centos7 系统内核是3.10以上
[zmk@centos7 /]$ uname -r
3.10.0-957.el7.x86_64
[zmk@centos7 /]$ cat /etc/os-release
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

#### 安装

帮助文档：

```shell
#1.卸载旧的版本
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
#2.下载关于Docker的依赖环境
sudo yum install -y yum-utils

#3.设置一下下载Docker的镜像源(这里用的国内得阿里源镜像)
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#4.更新yum软件包索引
sudo yum makecache fast

#4.安装Docker
sudo yum install docker-ce docker-ce-cli containerd.io

#5.启动Docker服务
sudo systemctl start docker
设置开机自动启动
sudo systemctl enable docker
```

可能会出现以下错误：

![image-20210617221619084](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E9%94%99%E8%AF%AF.png)

解决方法:

```shell
#通过将用户添加到docker用户组可以将sudo去掉，命令如下
sudo groupadd docker #添加docker用户组
sudo gpasswd -a $USER docker #将登陆用户加入到docker用户组中
newgrp docker #更新用户组
```

最后使用`docker version`查看，以下就是成功了

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E6%88%90%E5%8A%9F.png" alt="image-20210617221732985" style="zoom:50%;" />

下面进行docker下载进行测试：

```shell
#6.测试 会检查本地有没有hello-world镜像，如果没有则取dockerhub拉取
$ docker run hello-world
```

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E4%B8%8B%E8%BD%BD%E9%95%9C%E5%83%8F%E6%B5%8B%E8%AF%95.png" alt="image-20210617222004634" style="zoom:50%;" />

下面为docker配置阿里云镜像加速：

```shell
#6.配置阿里云镜像加速
#打开这个地址：http://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
#使用支付宝快捷登录阿里云可以获取镜像地址
#Docker版本要求≥1.12
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://dwo2x23b.mirror.aliyuncs.com"]
}
EOF

#7.重写加载配置文件
sudo systemctl daemon-reload

#8.重启docker
sudo systemctl restart docker

#9.卸载docker
sudo yum remove docker-ce docker-ce-cli containerd.io

sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

至此，docker的安装就彻底完成啦！下面回顾以下docker run的流程：

![image-20210618091733437](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E6%B5%81%E7%A8%8B.png)

### 3.底层原理

#### Docker是怎么工作的？

Docker 是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问。Docker-Server 接收到Docker-Client的指令 ，就会执行这个命令。

![image-20210618092705432](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E5%B7%A5%E4%BD%9C.png)

#### Docker为什么比VM快？

1. Docker有着比虚拟机更少的抽象层。

2. Docker利用的是宿主机的内核，vm需要的是Guest OS。

   ![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fuploadfiles.nowcoder.com%2Ffiles%2F20190907%2F5088755_1567871706761_20190906230851620.png&refer=http%3A%2F%2Fuploadfiles.nowcoder.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1626571773&t=674ff7e27b91ba7bb9064321dd92ba38)

   所以说，新建一个容器的时候，docker不需要像虚拟机一样重新加载一个操作系统内核，避免引导。虚拟机是加载Guest OS，分钟级别，而docker是利用宿主机的操作系统，秒级。

> 学习完所有的命令，再回过头看这段理论，会很清晰！

## 三.Docker 命令

### 1.帮助命令

```shell
docker version  	#显示docker的版本信息
docker info			#显示docker的系统信息，包括镜像和容器信息
docker 命令 --help   #帮助命令
```

帮助文档地址：[Reference documentation | Docker Documentation](https://docs.docker.com/reference/)

### 2.镜像命令

##### `docker images`查看所有镜像

```shell
[zmk@centos7 /]$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   3 months ago   13.3kB

#解释
REPOSITORY  镜像的仓库源
TAG			镜像的标签
IMAGE ID	镜像的id
CREATED		镜像的创建时间
SIZE		镜像的大小

#可选项
  -a, --all             #列出所有镜像
  -q, --quiet           #只显示镜像id
```

##### `docker search`搜索镜像

```shell
 [zmk@centos7 /]$ docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11008     [OK]
mariadb                           MariaDB Server is a high performing open sou…   4171      [OK]

#可选项，通过收藏来过滤
--filter=STARS=3000		#搜索出来的镜像就是STARS大于3000的

[zmk@centos7 /]$ docker search mysql --filter=STARS=3000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   11008     [OK]
mariadb   MariaDB Server is a high performing open sou…   4171      [OK]
```

##### `docker pull`下载镜像

```shell
#下载镜像docker pull 镜像名[:tag]
[zmk@centos7 /]$ docker pull mysql
Using default tag: latest	#如果不写tag，默认就是lasted最新的
latest: Pulling from library/mysql
69692152171a: Pull complete	#分层下载，docker image的核心	联合文件系统
1651b0be3df3: Pull complete
951da7386bc8: Pull complete
0f86c95aa242: Pull complete
37ba2d8bd4fe: Pull complete
6d278bb05e94: Pull complete
497efbd93a3e: Pull complete
f7fddf10c2c2: Pull complete
16415d159dfb: Pull complete
0e530ffc6b73: Pull complete
b0a4a1a77178: Pull complete
cd90f92aa9ef: Pull complete
Digest: sha256:d50098d7fcb25b1fcb24e2d3247cae3fc55815d64fec640dc395840f8fa80969	#签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest	#真实地址


#等价于它
docker pull mysql
docker pull docker.io/library/mysql:latest

#指定版本下载
[zmk@centos7 /]$ docker pull mysql:5.7 #下载mysql5.7的版本
5.7: Pulling from library/mysql
69692152171a: Already exists #分层下载的思想，可以共有之前下载的
1651b0be3df3: Already exists
951da7386bc8: Already exists
0f86c95aa242: Already exists
37ba2d8bd4fe: Already exists
6d278bb05e94: Already exists
497efbd93a3e: Already exists
a023ae82eef5: Pull complete
e76c35f20ee7: Pull complete
e887524d2ef9: Pull complete
ccb65627e1c3: Pull complete
Digest: sha256:a682e3c78fc5bd941e9db080b4796c75f69a28a8cad65677c23f7a9f18ba21fa
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

![image-20210618100808039](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%20pull.png)

##### `docker rmi`删除镜像

```shell
docker rmi -f 容器id	#删除指定的容器
docker rmi -f 容器id 容器id 容器id...	#删除多个容器
docker rmi -f $(docker images -aq)	#删除全部的容器
```

### 3.容器命令

**说明：我们有了镜像才可以创建容器，可以下载一个centos镜像来测试学习**

```shell
docker pull centos
```

##### `docker run`新建容器并启动

```shell
docker run [可选参数] image

#参数说明
--name="Name"	容器名字，centos7,centos8等用来区分容器
-d				后台方式运行
-it				使用交互方式运行，进入容器查看内容
-p				指定容器的端口
	-p ip:主机端口:容器端口
	-p 主机端口:容器端口（常用）
	-p 容器端口
	容器端口
-P				随机指定端口

#测试，启动并进入容器                      控制台
[zmk@centos7 /]$ docker run -it centos /bin/bash
[root@dc1db08428d5 /]# ls    #查看容器内的centos，基础版本，很多命令都不完善！
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

#从容器中退回主机  容器停止退出
[root@dc1db08428d5 /]# exit
exit

#第二种退出方式
Ctrl+P+Q  #容器不停止退出
```

##### `docker attach`进入正在运行的容器

docker attach命令可以attach到一个已经运行的容器的stdin，然后进行命令执行的动作。但是需要注意的是，如果在这里输入`exit`，会导致**容器的停止**。

```shell
[zmk@centos7 ~]$ docker start 28043ddc00c4
28043ddc00c4
[zmk@centos7 ~]$ docker attach 28043ddc00c4
[root@28043ddc00c4 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```

##### `docker ps`列出所有运行的容器

```shell
#docker ps 命令
	#列出当前正在运行的容器
-a 	#列出当前正在运行的容器+带出历史运行过的容器
-n=?#显示最近创建的容器个数
-aq #只显示容器的id


[zmk@centos7 /]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[zmk@centos7 /]$ docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS                          PORTS     NAMES
dc1db08428d5   centos         "/bin/bash"   5 minutes ago   Exited (0) About a minute ago             angry_bhaskara
f289bff1a143   d1165f221234   "/hello"      12 hours ago    Exited (0) 12 hours ago                   cool_shannon
[zmk@centos7 /]$ docker ps -a -n=1
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                     PORTS     NAMES
dc1db08428d5   centos    "/bin/bash"   10 minutes ago   Exited (0) 7 minutes ago             angry_bhaskara
[zmk@centos7 /]$ docker ps -aq
dc1db08428d5
f289bff1a143
```

##### `docker rm`删除容器

```shell
docker rm 容器id #删除指定的容器，不能删除正在运行的容器，如果要强制删除 docker rm -f
docker rm -f $(docker ps -aq) #删除所有容器
docker ps -aq|xargs docker rm #删除所有容器
```

##### 启动和停止容器

```shell
docker start 容器id	#启动容器
docker restart 容器id #重启容器
docker stop 容器id	#停止当前正在运行的容器
docker kill 容器id	#强制停止当前容器
```

### 4.常用其他命令

##### 后台启动容器

```shell
# docker run -d 镜像名
[zmk@centos7 /]$ docker run -d centos
0e8b862581e9bd49590f37995e7011f93b05ad12755541f1c79d859f958a03e3
[zmk@centos7 /]$ docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                     PORTS     NAMES
0e8b862581e9   centos    "/bin/bash"   5 seconds ago   Exited (0) 4 seconds ago             determined_hugle
[zmk@centos7 /]$

#问题docker ps -a发现centos停止了
#常见的坑，docker容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
#nginx，容器启动后，发现自己没有提供服务，就会立刻停止
```

##### `docker logs`查看日志

```shell
docker logs	#显示日志
-tf 	#根据指定格式的时间戳显示
--tail number #要显示的日志条数

#测试
[zmk@centos7 /]$ docker run -d centos /bin/bash -c "while true;do echo helloword;sleep 1;done"
#查看全部的日志信息
[zmk@centos7 /]$ docker logs -tf 4c63d7eb8117
#查看指定的日志信息

[zmk@centos7 /]$ docker logs -tf --tail 10 4c63d7eb8117
2021-06-18T03:21:25.868706054Z helloword #日志信息
2021-06-18T03:21:26.872859429Z helloword
2021-06-18T03:21:27.875830155Z helloword
2021-06-18T03:21:28.878657412Z helloword
2021-06-18T03:21:29.881151691Z helloword
2021-06-18T03:21:30.884480441Z helloword
2021-06-18T03:21:31.887277631Z helloword
2021-06-18T03:21:32.890503057Z helloword
2021-06-18T03:21:33.893543957Z helloword
2021-06-18T03:21:34.896107456Z helloword
2021-06-18T03:21:35.898898363Z helloword
```

##### `docker top`查看容器中进程的信息

```shell
[zmk@centos7 /]$ docker top 4c63d7eb8117
UID     PID     PPID      C      STIME      TTY     TIME         CMD
root    20179   20158     0      11:19      ?       00:00:00     /bin/bash -c while true;do echo helloword;sleep 1;done
root    20689   20179     0      11:26      ?       00:00:00     /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
```

##### `docker inspect`查看镜像的元数据

```shell
[zmk@centos7 /]$ docker inspect 4c63d7eb8117
[
    {
        "Id": "4c63d7eb8117919092b4eba4c56af6a6abf148700949b21575c4a93e87f32cc7",
        "Created": "2021-06-18T03:19:24.416606386Z",
        "Path": "/bin/bash",
        "Args": [
            "-c",
            "while true;do echo helloword;sleep 1;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 20179,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-06-18T03:19:24.978768139Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/4c63d7eb8117919092b4eba4c56af6a6abf148700949b21575c4a93e87f32cc7/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/4c63d7eb8117919092b4eba4c56af6a6abf148700949b21575c4a93e87f32cc7/hostname",
        "HostsPath": "/var/lib/docker/containers/4c63d7eb8117919092b4eba4c56af6a6abf148700949b21575c4a93e87f32cc7/hosts",
        "LogPath": "/var/lib/docker/containers/4c63d7eb8117919092b4eba4c56af6a6abf148700949b21575c4a93e87f32cc7/4c63d7eb8117919092b4eba4c56af6a6abf148700949b21575c4a93e87f32cc7-json.log",
        "Name": "/quizzical_bartik",
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
            "CgroupnsMode": "host",
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
                "LowerDir": "/var/lib/docker/overlay2/97176951e6d1222941716e61c469e5ef300252b2574d7cd6068387725a128e53-init/diff:/var/lib/docker/overlay2/6db6b5c4cf121666de7bf740fede8c4bb974c1ced9c76e4d16bc1222225119cb/diff",
                "MergedDir": "/var/lib/docker/overlay2/97176951e6d1222941716e61c469e5ef300252b2574d7cd6068387725a128e53/merged",
                "UpperDir": "/var/lib/docker/overlay2/97176951e6d1222941716e61c469e5ef300252b2574d7cd6068387725a128e53/diff",
                "WorkDir": "/var/lib/docker/overlay2/97176951e6d1222941716e61c469e5ef300252b2574d7cd6068387725a128e53/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "4c63d7eb8117",
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
                "/bin/bash",
                "-c",
                "while true;do echo helloword;sleep 1;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "de006405fbbc13f7725e40bf5258b55b025b39eae81d81e6df063fe521f2fbd7",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/de006405fbbc",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "3e7116d86c170859207fd5b08ef2bfb8f4b50dbdf2e0b6b7b5e49a55295507fe",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "9a86da52fcd256f5ac6e2836061559b30192e9246f21823f0e203fe5b2e276df",
                    "EndpointID": "3e7116d86c170859207fd5b08ef2bfb8f4b50dbdf2e0b6b7b5e49a55295507fe",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

##### `docker exec`进入当前正在运行的容器

可以看到，其中参数`-i -t -d`与`docker run`有些相同。其实，exec会进入创建一个伪终端，与直接`run`创建出来的相似。但是不同点在于，**不会因为输入`exit`而终止容器**。

```shell
#命令 docker exec -it 容器id bashShell
#测试
[zmk@centos7 /]$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
4c63d7eb8117   centos    "/bin/bash -c 'while…"   17 minutes ago   Up 17 minutes             quizzical_bartik
[zmk@centos7 /]$ docker exec -it 4c63d7eb8117 /bin/bash
[root@4c63d7eb8117 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@4c63d7eb8117 /]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 03:19 ?        00:00:00 /bin/bash -c while true;do echo helloword;sleep 1;done
root       1113      0  0 03:37 pts/0    00:00:00 /bin/bash
root       1199      1  0 03:39 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
root       1200   1113  0 03:39 pts/0    00:00:00 ps -ef
```

##### `docker cp`从容器内拷贝文件到主机上

```shell
#命令 docker cp 容器id:容器内路径 目的主机路径
docker cp4c63d7eb8117:/home/test.go /home/zmk/

#容器停止运行也可以拷贝 只要容器在，数据就在。
```

### 命令小结

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E5%91%BD%E4%BB%A4%E5%B0%8F%E7%BB%93.png" alt="image-20210619103047194" style="zoom: 50%;" />

### 作业练习

> Docker 安装 Nginx

```shell
#步骤
#1.搜索镜像 docker search 建议去docker hub搜索，可以看到帮助文档
#2.下载镜像 docker pull 
#3.运行测试
[zmk@centos7 ~]$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    d1a364dc548d   3 weeks ago    133MB
mysql        5.7       2c9028880e58   5 weeks ago    447MB
centos       latest    300e315adb2f   6 months ago   209MB

#-d 后台运行
#--name 给容器命名 不指定名称默认为镜像名
#-p 宿主机端口:容器内部端口
[zmk@centos7 ~]$ docker run -d --name nginx01 -p 3344:80 nginx
0ff8b1fc9e6ce4af47061fc8ed071563539efd3de97e26cdde04570c040694db
[zmk@centos7 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS                                   NAMES
0ff8b1fc9e6c   nginx     "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds   0.0.0.0:3344->80/tcp, :::3344->80/tcp   nginx01

#测试
[zmk@centos7 ~]$ curl localhost:3344
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
```

端口暴露的概念：

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%AE%BF%E9%97%AE%E6%B5%81%E7%A8%8B.png" alt="image-20210619105541712" style="zoom:50%;" />

进入nginx：

```shell
[zmk@centos7 ~]$ docker exec -it nginx01 /bin/bash
root@0ff8b1fc9e6c:/# ls
bin   dev                  docker-entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc                   lib   media  opt  root  sbin  sys  usr
root@0ff8b1fc9e6c:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@0ff8b1fc9e6c:/# cd /etc/nginx/
root@0ff8b1fc9e6c:/etc/nginx# ls
conf.d  fastcgi_params  mime.types  modules  nginx.conf  scgi_params  uwsgi_params
```

思考题：我们每次改动nginx配置文件，都需要进入容器内部？十分麻烦，我要是可以在容器外部提供一个映射路径，达到在容器修改文件名，容器内部就可以自动修改？

可以用后面的数据卷技术。

### 可视化

- portainer（先用这个）

##### 什么是portainer?

Docker图形化界面管理工具！提供一个后台面板供我们操作

##### 安装测试

```shell
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

访问测试：外网：http://192.168.19.130:8088

![image-20210620090046442](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/portainer.png)

自己设置用密码和密码进入

一般选择local模式：

![image-20210620090412581](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/portainer_local.png)

进入之后的面板如下所示：

![image-20210620090709744](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/portainer_detail.png)

一般来说我们不会使用这个可视化面板，自己测试看看就行

- Rancher（CI/CD再用）

## 四.Docker镜像讲解

### 1.镜像是什么

镜像是一种轻量级，可执行的软件独立包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码，运行时，库，环境变量和配置文件。

所有的应用，直接打包docker镜像，就可以直接跑起来。

### 2.Docker镜像加载原理

##### UnionFS（联合文件系统）

UnionFS（联合文件系统）：联合文件系统是一种分层，轻量级并且高性能的文件系统，它支持对文件系统的修改，作为一次提交来一层层的叠加，同时可以将不同的目录挂载到同一个虚拟文件系统下。Union文件系统是Docker镜像的基础，镜像可以通过分层来进行继承，基于基础镜像可以制作各种具体的镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

我们下载的时候看到的一层层就是这个！

##### Docker镜像加载原理

Docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统就是UnionFS。

**bootfs(boot file system)**主要包含`bootloader`和`kernel`，bootloader主要引导加载kernel，linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层就是bootfs，这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核，当boot加载完成之后整个内核就都在内存中了，内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

**rootfs(root file system)**在bootfs之上，包含的就是典型的Linux系统中的/dev、/proc、/bin、/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，CentOS等。

<img src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fn1.itc.cn%2Fimg8%2Fwb%2Fsmccloud%2F2015%2F05%2F07%2F143098875606357619.JPG&refer=http%3A%2F%2Fn1.itc.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1626744694&t=064263e9cb4f51e8d8f04abd29f36b78" alt="img" style="zoom:50%;" />

平时我们安装虚拟机的CentOS都是好几个G，为什么Docker中的CentOS才200M?

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker_centos.png" alt="image-20210620093348312" style="zoom:50%;" />

对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令，工具和程序库就可以了，因为底层直接用的Host的kernel，自己子需要提供rootfs就可以了。由此可见对于不同的linux发行版,bootfs基本是一致的，rootfs会有所差别，因此不同发行版可以共用bootfs。

### 3.分层理解

所有的Docker镜像都起始于一个基础镜像层，当进行修改或增加内容时，就会在当前镜像层之上，创建新的镜像层。

举一个简单的例子，假如基于Ubuntu 16.04创建一个新的镜像，这就是新镜像的第一层；如果在该镜像中添加Python包，就会在基础镜像层之上创建第二个镜像层；如果继续添加一个安全补丁，就会创建第三个镜像层。

该镜像当前已经包含3个镜像层，如下图所示：

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E5%88%86%E5%B1%82.png" alt="image-20210620104039716" style="zoom:50%;" />

在添加额外的镜像层的同时，镜像始终保持是当前所有镜像的组合，理解这一点非常重要。下图举了一个非常简单的例子，每个镜像层包含3个文件，而镜像包含了来自两个镜像层的6个文件。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E5%88%86%E5%B1%822.png" alt="image-20210620105010364" style="zoom:50%;" />

上图中的镜像层跟之前图中的略有区别，主要目的是便于展示文件。

下图中展示了一个稍微复杂的三层镜像，在外部看来整个镜像只有6个文件，这是因为最上层中的文件7是文件5的一个更新版本。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E5%88%86%E5%B1%823.png" alt="image-20210620105059658" style="zoom:50%;" />

这种情况下，上层镜像层中的文件覆盖了底层镜像层中的文件，这样就使得文件的更新版本作为一个新镜像层添加到镜像当中，Docker通过存储引擎（新版本采用快照机制）的方式来实现镜像层堆栈，并保证多镜像对外展示为同一的文件系统。

Linux上可用的存储引擎有AUFS、Overlay2、Device Mapper、Btrfs以及ZFS，顾名思义，每种存储引擎都基于Linux中对于的文件系统或整个块设备技术，并且每种存储引擎都有其独有的性能特点。

Docker在Windows上仅支持windowsfilter一种存储引擎，该引擎基于NTFS文件系统之上实现了分层和CoW。

下图展示了与系统显示相同的三层镜像，所有镜像层堆叠并合并，对外提供统一的视图。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E5%88%86%E5%B1%824.png" alt="image-20210620105425217" style="zoom:50%;" />

Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。这一层就是我们通常说的容器层，容器之下的都叫镜像层。

### 4.commit镜像

```shell
docker commit 提交容器成为一个新的副本
# 命令和git类似
docker commit -m="提交信息" -a="作者" 容器id 目标镜像名:[TAG]
```

> 上面的学习只是Docker的入门，下面是Docker的精髓！

## 五.容器数据卷

### 1.什么是容器数据卷

如果数据都在容器中，那么我们将容器删除，数据就会丢失！

需求：数据可用持久化

MySQL，容器删了，相当于删库跑路！

需求：MySQL数据可以存储在本地

容器之间可以有一个数据共享的技术！Docker容器中产生的数据，同步到本地！

这就是卷技术！目录的挂载，将我们容器内的目录，挂载到Linux上面！

![image-20210621091202366](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%8D%B7%E6%8A%80%E6%9C%AF.png)

总结一句话：容器的持久化和同步操作！容器间也可以数据共享的！

### 2.使用数据卷

> 方式一：直接使用命令挂载 -v

```shell
docker run -it -v 主机目录:容器内目录

#测试
[zmk@centos7 ~]$ docker run -it -v /home/test:/home/ centos
#使用docker inspect检查是否挂载成功
```

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E6%95%B0%E6%8D%AE%E5%8D%B7.png" alt="image-20210621092113703" style="zoom:50%;" />

**测试同步**

容器内

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%B5%8B%E8%AF%95%E5%90%8C%E6%AD%A51.png" alt="image-20210621092304058" style="zoom:50%;" />

主机已经同步

![image-20210621092321829](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%B5%8B%E8%AF%95%E5%90%8C%E6%AD%A52.png)

**再来测试**

1. 停止容器

2. 在宿主机上修改文件

3. 启动容器

4. 容器内的数据依旧是同步的

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E4%B8%BB%E6%9C%BA%E4%B8%8A%E4%BF%AE%E6%94%B9%E6%96%87%E4%BB%B6.png" alt="image-20210621093516620" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%AE%B9%E5%99%A8%E5%90%AF%E5%8A%A8%E5%90%8E%E8%87%AA%E5%8A%A8%E5%90%8C%E6%AD%A5.png" alt="image-20210621093548578" style="zoom:50%;" />

##### 实战：配置MySQL数据持久化

```shell
#运行容器需要做数据挂载! 安装启动mysql需要配置密码，这是注意点
#官方测试：docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

#配置启动mysql
-d					 后台运行
-p 主机端口:容器端口	端口映射
-v					 卷挂载
-e 					 环境配置，这是设置了MySQL密码
--name				 容器名字

docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
```

![image-20210621095538538](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker_mysql%E5%90%AF%E5%8A%A8%E5%B1%82%E9%AB%98.png)

启动成功之后，在本地使用sqlyog测试连接一下

![image-20210621095455233](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/sqlyog%E8%BF%9E%E6%8E%A5%E6%B5%8B%E8%AF%95.png)

新建数据库测试同步，数据存储在本地

![image-20210621095859634](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%96%B0%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%BA%93%E6%B5%8B%E8%AF%95%E5%90%8C%E6%AD%A5.png)

### 3.具名挂载和匿名挂载

```shell
#匿名挂载
-v 容器内路径  （不指定主机内路径）
docker run -d -P --name nginx01 -v /etc/nginx nginx

#查看所有的卷的情况   匿名卷挂载
[root@centos7 /]# docker volume ls
DRIVER    VOLUME NAME
local     7360c401b5358793e8ecbd09ee936c15a3e2308b5c9c027248f931233e759891

#这里发现，这就是匿名挂载，我们在-v 只写了容器内的路径，没有写容器外的路径！

#启动一个具名挂载
-v 卷名:容器内路径
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
#查看所有卷的情况   具名挂载，这里可以看到一个名为juming-nginx的卷
[root@centos7 /]# docker volume ls
DRIVER    VOLUME NAME
local     juming-nginx
```

![image-20210621102445828](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%85%B7%E5%90%8D%E6%8C%82%E8%BD%BD.png)

所有的docker容器内部的卷，没有指定主机目录的情况下，都是在`/var/lib/docker/volumes/xxx/_data`中存储。

我们可以通过具名挂载可以方便的找到我们的一个卷，大多数情况下使用`具名挂载`。

```shell
#如何确定是具名挂载还是匿名挂，还是指定路径挂载！
-v 容器内路径				  #匿名挂载
-v 卷名:容器内路径  	   		 #具名挂载
-v /主机路径:容器内路径    	    #指定路径挂载
```

**拓展**

```shell
#通过 -v 容器内路径，ro rw 改变读写权限
ro	readonly #只读
rw  readwrite #可读

#一旦设置了这个，容器对我们挂载出来的内容就有限定了
docker run -d -p --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -p --name nginx02 -v juming-nginx:/etc/nginx:rw nginx

# ro 只要看到ro就说明这个路径只能通过宿主机来操作，容器内部是无法操作的
```

### 4.初识Dockerfile

> 方式一：编写Dockefile的时侯挂载

Dockerfile就是用来构建docker镜像的构建文件！命令脚本！先体验一下，通过这个脚本可以生成镜像，镜像是一层一层的，脚本是一个一个的命令，每个命令都是一层。

```shell
#创建一个dockerfile文件
#文件内容如下，一个简单的测试  指令都是大写
FROM centos

VOLUME ["/volume01","/volume02"] #匿名挂载

CMD echo "-----end-----"
CMD /bin/bash

#生成镜像
-f 	dockerfile的路径,文件名为Dockerfile则无需加该参数，默认寻找Dockerfile
-t 	镜像名和标签
[zmk@centos7 docker-test-volume]$ docker build -f dockerfile01 -t zmk/centos:1.0 .
```

![image-20210621110622467](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker_build.png)

```shell
#启动自己的镜像
[zmk@centos7 dockerfile]$ docker run -it zmk/centos:1.0
```

进入容器，在volume01中创建container.txt文件

![image-20210621165043440](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%87%AA%E5%8A%A8%E6%8C%82%E8%BD%BD.png)

查看卷挂载的路径

![image-20210621165229188](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%9F%A5%E7%9C%8B%E5%8D%B7%E6%8C%82%E8%BD%BD%E7%9A%84%E8%B7%AF%E5%BE%84.png)

测试一下刚才的文件是否同步

![image-20210621165511488](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%B5%8B%E8%AF%95%E5%90%8C%E6%AD%A5%E6%88%90%E5%8A%9F.png)

### 5.数据卷容器

![image-20210621143557727](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%95%B0%E6%8D%AE%E5%8D%B7%E5%AE%B9%E5%99%A8.png)

```shell
#运行父容器
docker run -it --name docker01 zmk/centos:1.0

#在父容器docker01的基础上运行docker02容器
docker run -it --name docker02 --volumes-from docker01 zmk/centos:1.0

#像父容器的volume02卷中添加一个test.txt文件，再进入docker02进行查看，观察是否同步
```

![image-20210621171031788](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%95%B0%E6%8D%AE%E5%8D%B7%E5%AE%B9%E5%99%A8%E6%88%90%E5%8A%9F.png)

测试，可以删除父容器docker01，查看一个docker02和docker03是否还可以访问这个文件，发现仍然可以访问。

![image-20210621171552998](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%B5%8B%E8%AF%95%E5%88%A0%E9%99%A4docker01.png)

## 六.Dockerfile

Dockerfile的核心是用来构建docker镜像的文件！命令参数脚本！

### 1.构建步骤

1. 编写一个dockerfile文件
2. docker build构建成为一个镜像
3. docker run 运行镜像
4. docker push发布镜像（Docker Hub、阿里云镜像仓库）

### 2.Dockerfile文件指令

**1、 FROM**

指定基础镜像（必须有的指令，并且必须是第一条指令）

**2、 WORKDIR**

格式为 `WORKDIR` <工作目录路径>

使用 `WORKDIR` 指令可以来**指定工作目录**（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如果目录不存在，`WORKDIR` 会帮你建立目录

**3、COPY**

格式：

```dockerfile
COPY <源路径>... <目标路径>
COPY ["<源路径1>",... "<目标路径>"]
```

`COPY` 指令将从构建上下文目录中 <源路径> 的文件/目录**复制**到新的一层的镜像内的 <目标路径> 位置

**4、RUN**

用于执行命令行命令

格式：`RUN` <命令>

**5、EXPOSE**

格式为 `EXPOSE` <端口 1> [<端口 2>...]

`EXPOSE` 指令是**声明运行时容器提供服务端口，这只是一个声明**，在运行时并不会因为这个声明应用就会开启这个端口的服务

在 Dockerfile 中写入这样的声明有两个好处

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射
- 运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口

**6、ENTRYPOINT**

`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为两种格式

- `exec` 格式：

```dockerfile
<ENTRYPOINT> "<CMD>"
```

- `shell` 格式：

```dockerfile
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
```

`ENTRYPOINT` 指令是**指定容器启动程序及参数**

**总结：更加丰富的操作**

```shell
FROM		#基础镜像  centos ubuntu等,从这开始构建
MAINTAINER  #镜像是谁写的，姓名+邮箱
RUN         #镜像构建的时候需要运行的命令
ADD			#步骤：搭建带有mysql镜像的容器，这个mysql压缩包需要添加进去
WORKDIR     #镜像的工作目录
VOLUME      #挂载的目录
EXPOSE      #暴露端口配置，和-p一样
CMD         #指定这个容器启动时要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT  #指定这个容器启动时要运行的命令，可以追加命令
ONBUILD     #当构建一个被继承的DockerFile，这个时候就会运行ONBUILD指令，是一个触发指令
COPY        #类似ADD，将我们的文件拷贝到镜像中
ENV			#构建的时候设置环境变量
```

##### 实战：构建自己的centos

```shell
#1.编写Dockerfile文件
FROM centos
MAINTAINER zmk<zmkfml@163.com>

# 默认进入/usr/local路径
ENV MYPATH /usr/local

# 下载vim和net-tools
RUN yum -y install vim
RUN yum -y install net-tools

# 镜像工作目录
WORKDIR $MYPATH

EXPOSE 80

CMD echo $MYPATH
CMD "----END----"
CMD /bin/bash

#2.通过这个文件来构建镜像  文件名为Dockerfile则无需加该参数，默认寻找Dockerfile
#命令 docker build -f dockerfile文件路径 -t 镜像名:[tag]
[zmk@centos7 dockerfile]$ docker build -f mydockerfile-centos -t mycentos:0.1 .

#3.测试运行镜像
[zmk@centos7 dockerfile]$ docker run -it --name mycentos mycentos:0.1

#4.发布镜像
```

对比：之前原生的centos

![image-20210621153947706](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%AE%98%E7%BD%91%E4%B8%8B%E8%BD%BD%E7%9A%84centos%E9%95%9C%E5%83%8F.png)

我么增加之后的镜像

![image-20210621153823174](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%87%AA%E5%B7%B1%E6%9E%84%E5%BB%BA%E7%9A%84centos.png)

我们可以通过`docker history`来查看镜像的历史

![image-20210621154910713](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker_history.png)

### 3.发布自己的镜像

> 发布到DockerHub上

1. 地址：[Docker Hub](https://hub.docker.com/)注册自己的账号
2. 登录Docker Hub账号

```shell
#登录docker命令
[root@centos7 /]# docker login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
 
 [root@centos7 /]# docker login -u 283680926
```

3. 登录完毕后就可以提交镜像了docker  push

```shell
#需要在自己docker镜像前加上自己的DockerHub用户名
#先更改tag标签
[root@centos7 /]# docker tag 831023b5d535 283680926/centos-stronger:1.0
[root@centos7 /]# docker images
REPOSITORY                  TAG       IMAGE ID       CREATED          SIZE
mycentos                    0.1       831023b5d535   2 hours ago      295MB
283680926/centos-stronger   1.0       831023b5d535   
#push到DockerHub上
[root@centos7 /]# docker push 283680926/centos-stronger:1.0
The push refers to repository [docker.io/283680926/centos-stronger]
dde9ce4475c1: Pushed
98641cabf03c: Pushed
2653d992f4ef: Pushed
1.0: digest: sha256:5cd363caec787e1715b174cffb605af74af9a5ccfa9569591bb014f55c08b3a1 size: 953
```

### 小结

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%E5%B0%8F%E7%BB%93%E5%9B%BE.png" alt="image-20210621192706108" style="zoom: 80%;" />

## 七.Docker网络

### 1.理解Docker0

使用`ip addr`查看当前网络

![image-20210621195301114](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/ip%20addr.png)

可以发现有三个网络，那么来问题了：**docker 是如何处理容器网络访问的？即两个容器如何访问？**

```shell
#进入容器
[zmk@centos7 home]$ docker run -it --name centos01 centos
#容器内部进行ip addr查看网络地址
[root@7ed1c2257364 /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
84: eth0@if85: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever


#思考，linux能不能ping通容器内部！
[zmk@centos7 home]$ ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.990 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.099 ms

#说明linux可以ping通docker容器内部
```

> 原理

我们每启动一个docker容器，docker就会给docker容器分配一个ip，我们只要安装了docker，就会有一个网卡`docker0`桥接模式。使用的技术是`veth-pair`技术。

**在宿主机内部使用ip addr查看**

![image-20210621204257459](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/evth-pair.png)

发现多了一个`85: veth9f17390@if84:`网卡，这好像与centos01容器中的`84: eth0@if85:`网卡对应。

**再启动一个centos02容器进入内部使用ip addr查看**

![image-20210621204628604](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/centos02.png)

**宿主机再次使用ip addr查看**

![image-20210621204847226](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%AE%BF%E4%B8%BB%E6%9C%BA02.png)

```shell
#我们发现这个容器带来网卡，都是一对一对的
#veth-pair 就是一对的虚拟设备接口，都是成对出现的。一端连着协议，一端彼此相连。
#正因为有这个特性，veth-pair充当桥梁，连接着各种虚拟网络设备。
```

**测试centos02是否能ping通centos01**

```shell
[root@4aa4f939888b /]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.176 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.082 ms

#结论：容器和容器之间是可以互相ping通的
```

绘制一个图来解释就会很明白

![image-20210621211301416](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%9B%BE%E5%BD%A2%E8%A7%A3%E9%87%8A.png)

结论：centos01和centos02是共有一个路由器`docker0`，**所有的容器再不指定网络的情况下，都是docker0路由的，docker会给我们的容器分配一个默认的可用ip**。

> 小结

Docker使用的是Linux的桥接，宿主机是一个Docker容器的网桥docker0

![image-20210621212339233](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%B0%8F%E7%BB%93.png)

### 2.--link

> 思考一个场景，我们编写了一个微服务，database url=ip，项目不重启，数据库ip换掉了，我们希望可以处理这个问题，可以用`名字`来进行访问容器

```shell
#启动centos01容器
[zmk@centos7 home]$ docker run -it --name centos01 centos /bin/bash
[root@d8627a36e0c7 /
[zmk@centos7 home]$ docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
d8627a36e0c7   centos    "/bin/bash"   14 seconds ago   Up 13 seconds             centos01
#启动centos02容器
[zmk@centos7 home]$ docker run -it --name centos02 centos /bin/bash
#测试是否能通过容器名ping通centos01容器
[root@3fd672e488ae /]# ping centos01
ping: centos01: Name or service not known

#发现ping不通，那么如何解决呢？
--link 
#使用--link将centos02容器与centos03容器连接起来
[zmk@centos7 home]$ docker run -it --name centos03 --link centos02 centos /bin/bash
#这时候发现centos03可以ping通centos02容器了
[root@bf5ff9ce2424 /]# ping centos02
PING centos02 (172.17.0.3) 56(84) bytes of data.
64 bytes from centos02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.154 ms
64 bytes from centos02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.110 ms
64 bytes from centos02 (172.17.0.3): icmp_seq=3 ttl=64 time=0.107 ms

#但是反向centos02却不能ping通centos03,需要配置
```

**原理**

![image-20210622090026081](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/--link%E5%8E%9F%E7%90%86.png)

![image-20210622090156556](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/centos02%E8%AE%BF%E9%97%AEcentos03.png)

--link就是在`hosts`配置中增加了一个`172.17.0.3      centos02 3fd672e488ae`，我们现在用Docker已经不建议使用--link了，采用自定义网络的方式。Docker0不支持容器名连接访问，可以在自定义网络中配置通过容器名访问。

### 3.自定义网络

> 查看所有的docker网络

![image-20210622090623821](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%9F%A5%E7%9C%8B%E6%89%80%E6%9C%89%E7%9A%84docker%E7%BD%91%E7%BB%9C.png)

##### 网络模式

- bridger：桥接模式 docker默认，多采用自定义桥接网络
- none：不配置网络，但拥有自己的network namespace
- host：和宿主机共享网络
- container：容器内网络联通，指定新创建的容器和已经存在的容器共享一个network namespace

##### 创建网络

```shell
#命令：docker net create [参数] 网络名
--driver 网络模式，默认采用桥接模式，不写也可以
--subnet 子网掩码
--gateway 网关

[zmk@centos7 home]$ docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
d958a06ad0d7381de47b7ca390bda03a30c9ffd8ecc5889d88dad7328f7f5cdf
#查看创建的网络
[zmk@centos7 home]$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
25dd1b75f4ec   bridge    bridge    local
7031a83106ae   host      host      local
d958a06ad0d7   mynet     bridge    local
bf408a4b25cd   none      null      local
```

通过`docker network inspect 网络名`查看自己的网络

![image-20210622092020950](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%87%AA%E5%B7%B1%E7%9A%84%E7%BD%91%E7%BB%9C.png)

在我们自定义的网络下启动两容器测试

```shell
#--net 网络名
[zmk@centos7 home]$ docker run -it --name centos01 --net mynet centos /bin/bash

[zmk@centos7 home]$ docker run -it --name centos02 --net mynet centos /bin/bash
```

![image-20210622092713673](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%AE%B9%E5%99%A8%E5%8A%A0%E5%85%A5%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BD%91%E7%BB%9C.png)

这时候在自定义的网络中是非常完善的，可以通过加入到自定义网络中的容器名访问

![image-20210622093014535](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BD%91%E7%BB%9C%E4%B8%AD%E9%80%9A%E8%BF%87%E5%AE%B9%E5%99%A8%E5%90%8D%E7%9B%B4%E6%8E%A5%E8%AE%BF%E9%97%AE.png)

![image-20210622093102511](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%9C%A8%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BD%91%E7%BB%9C%E4%B8%AD%E9%80%9A%E8%BF%87%E5%AE%B9%E5%99%A8%E5%90%8D%E7%9B%B4%E6%8E%A5%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AE.png)

> 小结

我们自定义的网络docker都已经帮我们维护好了对应的关系，推荐这样使用网络。

好处：不同的集群使用不同的网络，保证集群是安全和健康的。

![image-20210622093741611](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%A5%BD%E5%A4%84.png)

### 4.网络连通

在上面的小结中我们如何能连通访问两个不同网段呢，正常情况下网卡直接是不能连通的，但是容器和网络可以连通。

```shell
#测试打通centos-net-01到mynet网络
docker network connect [可选项] 网络名 容器名
```

![image-20210622155730721](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker%20connect%E6%96%B9%E6%B3%95.png)

```shell
#将docker centos-net-01连接到mynet网络中
[zmk@centos7 home]$ docker network connect mynet centos-net-01
#查看mynet网络
[zmk@centos7 home]$ docker network inspect mynet
```

![image-20210622155351333](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E7%BD%91%E7%BB%9C%E8%BF%9E%E9%80%9A.png)

一个容器两个ip地址，类似于公网ip和私网ip。

打通成功

![image-20210622160142937](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%89%93%E9%80%9A%E6%88%90%E5%8A%9F.png)

##### 实战：部署redis集群

模拟部署一个，当redis-master3号机器宕机后，redis-slaver3号顶上去的一个模型。

![image-20210622160716279](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E9%83%A8%E7%BD%B2redis%E9%9B%86%E7%BE%A4.png)

```shell
#创建redis网卡
docker network create redis --subnet 172.38.0.0/16

#通过脚本创建六个redis配置
for port in $(seq 1 6); 
do 
mkdir -p /home/redis/node-${port}/conf 
touch /home/redis/node-${port}/conf/redis.conf 
cat <<EOF >/home/redis/node-${port}/conf/redis.conf
port 6379 
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

#启动脚本
docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /home/redis/node-${port}/data:/data \
-v /home/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

#启动以redis-1为例
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /home/redis/node-1/data:/data \
-v /home/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
```

6个redis启动成功

![image-20210622164954091](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/6%E4%B8%AAredis%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.png)

```shell
#创建集群
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
```

创建成功

![image-20210622171740238](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%88%9B%E5%BB%BA%E9%9B%86%E7%BE%A4.png)

查看redis集群信息

![image-20210622171852289](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%9F%A5%E7%9C%8Bredis%E9%9B%86%E7%BE%A4%E4%BF%A1%E6%81%AF.png)

查看集群节点信息

![image-20210622172340854](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%9F%A5%E7%9C%8B%E9%9B%86%E7%BE%A4%E8%8A%82%E7%82%B9%E4%BF%A1%E6%81%AF.png)

测试

```shell
#可以看到172.38.0.12的机子进行了set操作
127.0.0.1:6379> set test test-result
-> Redirected to slot [6918] located at 172.38.0.12:6379
OK
```

下面将172.38.0.12容器redis-2停止`docker stop redis-2`，再去get查询，如果能查询到，说明搭建的集群高可用

![image-20210622172729012](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%8F%AF%E7%94%A8.png)

>到目前为止，关于docker的基础已经学完了，再接再厉呀~
>
>学习的地址：[【狂神说Java】Docker最新超详细版教程通俗易懂_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1og4y1q7M4)

