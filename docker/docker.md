# Docker

![docker](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker.jpg)

## 一.Docker的思想

- 集装箱：
  会将所有需要的内容放到不同的集装箱中，谁需要这些环境就直接拿到这个集装箱就可以了
- 标准化：
  运输的标准化：Docker有一个码头，所有上传的集装箱都放在了这个码头上，当谁需要某一个环境，就直接指派大海豚去搬运这个集装箱就可以了。
  命令的标准化：Docker提供了一系列的命令，帮助我们去获取集装箱等等操作。
  提供了REST的API：衍生出了很多图形化界面，Rancher。
- 隔离性：
  Docker在运行集装箱内的内容时，会在Linux的内核中，单独的开辟一片空间，这片空间不会影响到其他程序。

docker的核心分为以下三点：

- 注册中心（超级码头，上面放的就是集装箱）

- 镜像（集装箱）

- 容器（运行起来的镜像）

## 二.Docker优势所在

1. 可以为部署在容器中的应用提供一致的运行时环境。使用Docker打包好的镜像在任何安装了Docker的服务器上正常运行，而无需考虑环境、配置、依赖等问题，可以做到快速部署项目。

2. 对于项目的运行环境的迁移也很方便。

3. 可以通过Docker来`定制镜像`，做到`持续集成`、交付和部署。

4. 结合微服务，将服务定制成镜像，可以做到负载的`快速扩展`。

5. 容器相对于实体系统更`轻量`，启动`快速`。

6. 在`分布式部署`和`微服务`方面应用很有潜力。


## 三. Docker的基本操作

### 3.1 安装Docker

```sh
CentOS系统操作
1. 下载关于Docker的依赖环境
yum -y install yum-utils device-mapper-persistent-data lvm2
2. 设置一下下载Docker的镜像源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
3. 安装Docker
yum makecache fast
yum -y install docker-ce
4. 启动，并设置为开机自动启动，测试
# 启动Docker服务
systemctl start docker
# 设置开机自动启动
systemctl enable docker
# 测试
docker run hello-world

配置阿里云镜像加速
# 阿里云加速：https://dwo2x23b.mirror.aliyuncs.com
# 打开这个地址：http://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
# 使用支付宝快捷登录阿里云可以获取镜像地址
# Docker版本要求≥1.12

mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://dwo2x23b.mirror.aliyuncs.com"]
}
EOF
#重写加载配置文件
systemctl daemon-reload
#重启docker
systemctl restart docker
```

### 3.2 Docker的中央仓库

Docker官方的中央仓库：这个仓库是镜像最全的，但是下载速度很慢。https://hub.docker.com

国内的镜像网站：

- 网易蜂巢： https://c.163yun.com/hub#/home
- DaoCloud： http://hub.daocloud.io (推荐使用)

在公司内部会采用私服的方式拉取镜像。（添加配置）

```sh
# 需要在/etc/docker/daemon.json
{
	"registry-mirrors":["https://registry.docker-cn.com"],
    "insecure-registries":["ip:port"]
}
# 重启两个服务
systemctl daemon-reload
systemctl restart docker
```

### 3.3 镜像的操作

1.拉取镜像到本地

```sh
docker pull 镜像名称[:tag版本标签]
# example 
docker pull tomcat 默认去中央仓库下载 
# 在国内镜像网站daoCloud拉取镜像
docker pull daocloud.io/library/tomcat:8.5.15-jre8 
```

2.查看全部本地的镜像

```sh
docker images
```

3.删除本地镜像

```sh
docker rmi 镜像的id标识
```

4.导出镜像

```sh
docker save -o 文件名.tar 镜像名:标签
# example
docker save -o centos_7.tar centos:7
```

5.导入镜像

```sh
cat 文件名.tar | docker import  - 镜像名:标签
# example
cat centos_7.tar | docker import - centos:7

# ------
# 镜像载入
docker load --input 文件名.tar
# example
docker load --input centos_7.tar
# 或者
docker load < centos_7.tar

# ------
# 容器导入为镜像
cat 容器名.tar | docker import - 镜像名:标签
# example
cat registry.tar | docker import - registry:1.0.0

# 修改镜像名称
docker tag 镜像id 新镜像名称:版本
```

### 3.4 容器的操作

1.运行容器

```sh
# 简单操作   后期一般用docker-compose运行容器
docker run 镜像的标识or镜像名称[:tag]
# 常用的参数
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像的标识or镜像名称[:tag]
# -d：代表后台运行容器
# -p 宿主机端口:容器端口：为了映射当前Linux端口和容器端口
# --name 容器名称：指定容器的名称
例如:docker run -d -p 8081:8080 --name tomcat bf980021
```

2.查看正在运行的容器

```sh
docker ps [-qa]
# -a：查看全部的容器，包括没有运行
# -q：只查看容器的id标识
```

3.查看容器的日志

```sh
docker logs -f 容器id
# -f：可以滚动查看日志的最后几行
```

4.导出容器

```
docker export 容器id > 容器名.tar
# example
docker export 5e56bf30b549 > registry.tar
```

5.进入到容器内部

```sh
docker exec -it 容器id bash
```

6.删除容器（删除容器前，需要停止容器）

```sh
# 停止指定的容器
docker stop 容器id
# 停止全部容器
docker stop $(docker ps -qa)
# 删除指定的容器
docker rm 容器id
# 删除全部容器
docker rm $(docker ps -qa)
```

7.启动容器

```sh
docker start 容器id
```

### 3.5"docker run"命令参数解析

```sh
"docker run"命令参数解析：
  -d, --detach=false //指定容器运行于前台还是后台，默认为false			   
  -i, --interactive=false //打开STDIN，用于控制台交互						
  -t, --tty=false //分配tty设备，该可以支持终端登录，默认为false			   
  -u, --user="" //指定容器的用户										  
  -a, --attach=[] //登录容器（必须是以docker run -d启动的容器）
  -w, --workdir="" //指定容器的工作目录
  -c, --cpu-shares=0 //设置容器CPU权重，在CPU共享场景使用
  -e, --env=[] //指定环境变量，容器中可以使用该环境变量
  -m, --memory="" //指定容器的内存上限
  -P, --publish-all=false //指定容器暴露的端口                            
  -p, --publish=[] //指定容器暴露的端口                                   
  -h, --hostname="" //指定容器的主机名
  -v, --volume=[] //给容器挂载存储卷，挂载到容器的某个目录
  --volumes-from=[] //给容器挂载其他容器上的卷，挂载到容器的某个目录
  --cap-add=[] //添加权限，权限清单详见：http://linux.die.net/man/7/capabilities
  --cap-drop=[] //删除权限，权限清单详见：http://linux.die.net/man/7/capabilities
  --cidfile="" //运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法
  --cpuset="" //设置容器可以使用哪些CPU，此参数可以用来容器独占CPU
  --device=[] //添加主机设备给容器，相当于设备直通
  --dns=[] //指定容器的dns服务器
  --dns-search=[] //指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件
  --entrypoint="" //覆盖image的入口点
  --env-file=[] //指定环境变量文件，文件格式为每行一个环境变量
  --expose=[] //指定容器暴露的端口，即修改镜像的暴露端口
  --link=[] //指定容器间的关联，使用其他容器的IP、env等信息
  --lxc-conf=[] //指定容器的配置文件，只有在指定--exec-driver=lxc时使用
  --name="" //指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字
  --net="bridge" //容器网络设置:
  　　bridge //使用docker daemon指定的网桥
  　　host //容器使用主机的网络
  　　container:NAME_or_ID //使用其他容器的网路，共享IP和PORT等网络资源
  　　none //容器使用自己的网络（类似--net=bridge），但是不进行配置
  --privileged=false //指定容器是否为特权容器，特权容器拥有所有的capabilities
  --restart="no" //指定容器停止后的重启策略:
  　　no //容器退出时不重启
  　　on-failure //容器故障退出（返回值非零）时重启
  　　always //容器退出时总是重启
  --rm=false //指定容器停止后自动删除容器(不支持以docker run -d启动的容器)
  --sig-proxy=true //设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理
```