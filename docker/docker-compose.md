# Docker-compose容器编排技术

![docker_compose](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/docker_compose.jpg)

项目地址: [https://github.com/docker/compose](https://github.com/docker/compose )

Compose 是用来定义和运行一个或多个容器应用的`工具`。

Compose 可以`简化`容器镜像的建立及容器的运行。

Compose 使用python语言开发，非常适合在`单机环境`里部署一个或多个容器，并自动把多个容器`互相关联`起来。

Compose 还是Docker`三剑客`之一。

### 本质
通过docker-api来与docker-server进行交互的。

### 两个重要概念
- **服务**(***service***)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

- **项目**(***project***)：由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml中定义。

### 安装docker
```bash
# 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 指定docker社区版的镜像源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum-config-manager --enable docker-ce-edge

yum-config-manager --enable docker-ce-test

# 安装docker社区版
yum install -y docker-ce

# 启动docker
systemctl start docker

# 查看docker版本
docker --version

# 开机启动
chkconfig docker on
```

### 安装compose
```bash
# 官方下载地址
curl -L https://github.com/docker/compose/releases/download/1.23.0-rc2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
# 国内下载地址
curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
# 如果没有curl命令则可以安装curl：
yum instal -y curl
chmod +x /usr/local/bin/docker-compose
docker-compose version
```

### 环境变量
```bash
# 环境变量可以用来配置 Compose 的行为,以DOCKER_开头的变量和用来配置 Docker 命令行客户端的使用一样。

COMPOSE_PROJECT_NAME

COMPOSE_FILE

DOCKER_HOST  

DOCKER_TLS_VERIFY

DOCKER_CERT_PATH
```

### docker-compose.yaml
整理的不全，详细请看 [官方文档](https://docs.docker.com/compose/compose-file/compose-file-v2/)
```yaml
# 例子docker-compose.yml(不能正常运行，仅供参考)
version: '2'   #文档版本

networks:
  helloworld:  #一个叫helloworld的网络
    driver: bridge  #使用桥接模式驱动网络

services:
  web:
    image: hello-world  #指定服务的镜像名称或者ID，如果镜像不在本地，compose会尝试拉取
    restart: always 	#表示总是重新启动容器
    net: "bridge"  #网络采取桥接模式
    ports:
      - "80:80"
    depends_on:		#用于解决依赖，启动先后的问题
      - redis
    networks:
      - helloworld
    dns:
      - 8.8.8.8
      - 9.9.9.9
    links:
      - web
    extra_hosts:
      - "redis:192.168.1.100"

  redis:
    image: redis
    container_name: redis
    commond: redis-server.sh ./redis.conf
    volumes:
      - ~/data:/data:rw
    expose:		#暴露端口，但不映射到宿主机
      - "6379"
    networks:
      - helloworld
```

1. image
- 是指定服务的镜像名称或镜像ID。如果镜像在本地不存在，Compose将会尝试拉取镜像。
```
services:
    web:
        image: hello-world
```

2. ==build==
- 服务除了可以基于指定的镜像，还可以基于一份`Dockerfile`，在使用up启动时执行构建任务，构建标签是build，可以指定Dockerfile所在文件夹的路径。Compose将会利用Dockerfile自动构建镜像，然后使用镜像启动服务容器。
```yaml
# 也可以是相对路径，只要上下文确定就可以读取到Dockerfile。
build: /path/to/build/dir　
```
```yaml
# 设定上下文根目录，然后以该目录为准指定Dockerfile。
build: ./dir
```
```yaml
# build都是一个目录，如果要指定Dockerfile文件需要在build标签的子级标签中使用dockerfile标签指定。
# 如果同时指定image和build两个标签，那么Compose会构建镜像并且把镜像命名为image值指定的名字。
build:
    context: ../
    dockerfile: path/of/Dockerfile
```
3. context
- context选项可以是Dockerfile的文件路径，也可以是到链接到git仓库的url，当提供的值是相对路径时，被解析为相对于撰写文件的路径，此目录也是发送到Docker守护进程的context
```yaml
build:
    context: ./dir
```

4. dockerfile
- 使用dockerfile文件来构建，必须指定构建路径
```yaml
build:
    context: .
    dockerfile: Dockerfile-alternate
```

5. commond
- 使用command可以覆盖容器启动后默认执行的命令。
```yaml
command: bundle exec thin -p 3000
```

6. container_name
- Compose的容器名称格式是：<项目名称><服务名称><序号>
- 可以自定义项目名称、服务名称，但如果想完全控制容器的命名，可以使用标签指定：
```yaml
container_name: app
```

7. depends_on
- 在使用Compose时，最大的好处就是少打启动命令，但一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。例如在没启动数据库容器的时候启动应用容器，应用容器会因为找不到数据库而退出。depends_on标签用于<u>解决容器的依赖、启动先后的问题</u>
```yaml
version: '2'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

8. PID
- 将PID模式设置为主机PID模式，跟主机系统共享进程命名空间。容器使用pid标签将能够访问和操纵其他容器和宿主机的名称空间。
```yaml
pid: "host"
```

9. ports
- ports用于映射端口的标签。
- 使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口。
```yaml
ports:
    - "3000"
    - "8000:8000" #宿主机端口:容器端口
    - "49100:22"
    - "127.0.0.1:8001:8001"
 # 当使用HOST:CONTAINER格式来映射端口时，如果使用的容器端口小于60可能会得到错误得结果，因为YAML将会解析xx:yy这种数字格式为60进制。所以建议采用字符串格式。
```

10. extra_hosts
- 添加主机名的标签，会在/etc/hosts文件中添加一些记录。
```yaml
extra_hosts:
    - "somehost:162.242.195.82"
    - "otherhost:50.31.209.229"
```

11. volumes
- 挂载一个目录或者一个已存在的数据卷容器，可以直接使用 [HOST:CONTAINER]格式，或者使用[HOST:CONTAINER:ro]格式，后者对于容器来说，数据卷是只读的，可以有效保护宿主机的文件系统。
Compose的数据卷指定路径可以是相对路径，使用 . 或者 .. 来指定相对目录。
数据卷的格式可以是下面多种形式
```yaml
volumes:
    # 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
    - /var/lib/mysql
    # 使用绝对路径挂载数据卷
    - /opt/data:/var/lib/mysql
    # 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
    - ./cache:/tmp/cache
    # 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
    - ~/configs:/etc/configs/:ro
    # 已经存在的命名的数据卷。
    - datavolume:/var/lib/mysql
    # 如果不使用宿主机的路径，可以指定一个volume_driver。
    # volume_driver: mydriver
```

12. volumes_from
- 从另一个服务或容器挂载其数据卷：
```yaml
volumes_from:
    - service_name   
    - container_name
```

13. dns
- 自定义DNS服务器。可以是一个值，也可以是一个列表。
```yaml
dns：8.8.8.8
dns：
    - 8.8.8.8   
    - 9.9.9.9
```

14. expose
- 暴露端口，但不映射到宿主机，只允许能被连接的服务访问。仅可以指定内部端口为参数，如下所示：
```yaml
expose:
    - "3000"
    - "8000"
```

15. links
- 链接到其它服务中的容器。使用服务名称（同时作为别名），或者“服务名称:服务别名”（如 SERVICE:ALIAS），例如：
```yaml
links:
    - db
    - db:database
    - redis
```

16. net
- 设置网络模式。
```text
net: "bridge"
net: "none"
net: "host"
```

17. env_file
- 环境变量文件，可以在yml文件引用这些变量
```yaml
env_file: .env
env_file:
    - ./common.env
    - ./apps/web.env
    - /opt/runtime_opts.env
```

18. extends
- 扩展当前文件或其他文件中的另一个服务(可选覆盖配置)。
```yaml
extends:
    file: common.yml
    service: webapp
```

19. restart
- no是默认的重启策略，它在任何情况下都不会重启容器。当always总是指定时，容器总是重新启动。如果退出码表明出现了on-failure错误，则on-failure策略将重新启动容器。
```yaml
restart: no
restart: always
restart: on-failure
restart: unless-stopped
```

20. environment
- 用于设置环境变量
```yaml
environment:
    - xxx
    - yyy
```


### 命令帮助
```bash
docker-compose --h
```
```text
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file(指定yml文件的名字)
                              (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
  --verbose                   Show more output
  --log-level LEVEL           Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --no-ansi                   Do not print ANSI control characters
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the
                              name specified in the client certificate
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)
  --compatibility             If set, Compose will attempt to convert keys
                              in v3 files to their non-Swarm equivalent

Commands:
  build              Build or rebuild services
  bundle             Generate a Docker bundle from the Compose file
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
```
