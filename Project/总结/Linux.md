# Linux

## 1. 安装

### 1.1 硬件和系统方面：

bios设置：

​	Intel Virtualization Technology 、Vt-x、Hyper-V（高级 > CPU设置）

系统设置：

​	在`启用或关闭Windows功能`中勾选如下几个选项：

- Hyper-V
- 适用于Linux的Windows子系统
- 虚拟机平台

### 1.2 通过WSL安装Ubuntu

准备工作

>wsl --set-default-version 2

通过wsl -l -v查看已经安装好的ubuntu使用wsl版本
如果使用的是wsl 1这个版本，可以通过这个指令进行升级
>wsl --set-version Ubuntu-22.04 2

如果是安装过程中 ，没有出现任何输入用户名和密码的地方，则是以root账号初始化的，安装完毕以后可以通过如下指令设置密码（后面通过远程方式登录，必须要密码的）

```shell
# sudo 表示以管理员的方式运行该指令，如果已经是管理员身份登录了，可以省略
sudo passwd root
```

## 2. SSH

首先修改SSH的配置文件 `/etc/ssh/sshd_config`

```shell
# 查看文件内容
cat sshd_config

# 使用VI工具
sudo vi sshd_config
```

在配置文件中找到如下2项，进行修改：

```shell
# 允许root用户登录
PermitRootLogin yes
# 认证方式为密码认证
PasswordAuthentication yes
```

启动SSH服务

```shell
service ssh start

#如果启动的时候提示
# Starting OpenBSD Secure Shell server sshd 
# sshd: no hostkeys available -- exiting.

# 可以通过如下指令生成key
sudo ssh-keygen -A

# 设置ssh开机自动启动
systemctl enable ssh
```

## 3. 进程查询

```shell
# 查询进程及相应的编号
ps -ef | grep ssh

# 查询相应端口的占用情况
sudo -lsof -i:22

# 杀死进程
kill -9 进程编号（PID）
```

## 4. JDK的安装

在控制台输入某一个指令回车，我们的OS会首先在当前目录下寻找对应的可执行文件，如果没有找到的话，会在当前系统的PATH环境下指定的目录下寻找

在`Linux`编辑 `~/.profile`

```shell
export JAVA_HOME=/home/think/Program/jdk1.8.0_202
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib
```

让`Linux`内核重新加载配置文件，使其生效

```shell
source .profile
```

可以通过如下指令查看是否生效

```shell
echo $PATH
echo $CLASSPATH
```

## 5. 部署项目

首先在`IDEA`中使用`maven`对项目进行打包

```shell
mvn package
```

> 在打包之前，根据`OS`修改`application.yml`中的DB的访问路径，如果是访问`windows`下的DB，需要关闭`windows`的防火墙

## 6. 杂项准备工作

由于在默认情况下，MySQL只允许在本地访问，所以需要使用MySQL官方的客户端工具，对访问权限进行设置：

`Administrator -> Users & Privileges`设置能访问的域为%（10.41.250.%），表示任意机器都能访问



设置Ubuntu的时区和Windows保持一致：

```shell
# 查看当前系统的时区信息
date -R
# 设置时区
timedatectl set-timezone Asia/Shanghai
```



## 7. Nginx安装

windows和Linux的安装， 直接解压即可

```shell
# 启动
nginx
# 退出
nginx -s quit
# 重新加载
nginx -s reload
```

### 7.1 简单安装方式

```shell
# 切换到管理员用户
sudo su
# 通过Linux的软件源直接安装
apt-get install nginx
```

nginx与正向代理（代理的是客户端）的区别，就在于它是代理的它身后的各个服务器

### 7.2 通过官方源码编译&安装

```shell
sudo apt-get install build essential
sudo apt-get install zlib1g zlib1g-dev
sudo apt-get install libpcre3 libpcre3-dev
sudo apt-get install openssl

# 上述编译所依赖的库安装好以后就可以执行源码编译
make
# 然后再执行安装
sudo make install

# 安装好以后，可以通过如下指令查看安装nginx的版本
nginx -v
```

## 8. 部署前端项目

```shell
# 首先在vscode执行打包命令
npm run build

# 把项目打包成zip格式上传到Ubuntu，比如/home/think/Project/dist.zip （根据各自的环境设置）

# 安装unzip解压工具
sudo apt-get install unzip

# 解压刚才上传的前端项目文件
unzip dist.zip

# 配置nginx
sudo vi /etc/nginx.conf

# 主要设置如下信息
server {
	listen 80;

	location / {
		# 设置项目的静态资源目录
		root    /home/think/Project/dist;
		# 设置首页文件名称
		index   index.html;
	}

}

# 保存，退出，然后重新加载nginx配置文件
sudo nginx -s reload
```

## 9. 部署后端项目

首先修改`application.yml`中的url为MySQL所在服务器的IP

然后重新上传至Ubuntu服务器

```shell
java -jar springxxxxx.jar --server.port=8090
java -jar springxxxxx.jar --server.port=8100
```

重新调整前端项目的baseURL

```javascript
// 设置为真实接收请求的服务器地址
// 添加一个约定好的前缀，例如/api
// 对于nginx服务器来说，它不一定是转发所有的请求，它在当前项目中只需要转发指定的前端发送的请求就行了，而这些请求都是可以通过/api来进行区分的
const domain = "http://10.41.250.122/api";
//const port = 8090;


export const http = axios.create({
    // 设置了异步请求的基本路径，最终的请求路径为baseURL + path
    baseURL: `${domain}`
});

```

重新打包，上传

配置负载均衡

```shell
        upstream movie {
                server localhost:8090 weight=5;
                server localhost:8100 weight=1;
        }

        server {
                listen 80;

                # 配置前端项目
                location / {
                        root    /home/think/Project/dist;
                        index   index.html;
                }

                # 配置反向代理的请求转发
                # 注意movie后面一定要加"/"
                location /api/ {
                        proxy_pass http://movie/;
                }

        }
```

> 注意：当前项目在认证token的时候，还是会存在一个问题：

- 当`8090`这个端口的服务器给前端签发了一个`token`的时候，如果在后续的请求中，有一个请求是被其它端口（例如`8100`）的应用程序接收的时候，那么`8100`这台服务器走到`JwtFilter`中进行`token`校验的时候，会校验失败，因为两台服务器的用来签发和校验的私钥不一致
- 关于这个问题，后面我们会统一使用公共的中间件（例如`Redis`）来共享私钥或者`token`



## 10.配置

### 操作

- 查看端口:netstat -anp |grep port

- 文件系统 容量 已用 可用 已用占比  挂载点 (列标题) df -h 

- 文件系统 fdisk blkid

- 内存

  free -h 		ps aux --sort -rss

  

### mysql docker

mkdir -p /usr/local/docker/mysql5.7/conf 

mkdir -p /usr/local/docker/mysql5.7/data 

mkdir -p /usr/local/docker/mysql5.7/log



- docker安装：

docker run -p 3306:3306 --name mysql5.7 \
-v /usr/local/docker/mysql5.7/conf:/etc/mysql/mysql.conf.d \
-v /usr/local/docker/mysql5.7/log:/var/log/ \
-v /usr/local/docker/mysql5.7/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root\
-d mysql:5.7



### kafka docker

docker pull wurstmeister/zookeeper
docker pull wurstmeister/kafka
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.247.128:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.247.128:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka

\#进入容器
docker exec -it ${CONTAINER ID} /bin/bash
cd opt/bin
\#单机方式：创建一个主题
[kafka-topics.sh](http://kafka-topics.sh/) --create --zookeeper 192.168.247.128:2181/kafka --replication-factor 1 --partitions 1 --topic mykafka
\#查看主题
[kafka-topics.sh](http://kafka-topics.sh/) --zookeeper 192.168.247.128:2181/kafka --list
[kafka-topics.sh](http://kafka-topics.sh/) --zookeeper 192.168.247.128:2181/kafka --describe --topic mykafka
\#查看consumer Group列表
[kafka-consumer-groups.sh](http://kafka-consumer-groups.sh/) --bootstrap-server 192.168.247.128:9092 --list
[kafka-consumer-groups.sh](http://kafka-consumer-groups.sh/) --bootstrap-server 192.168.247.128:9092 --describe --group consumer-group-mykafka

\#查看消息
[kafka-console-consumer.sh](http://kafka-console-consumer.sh/) --bootstrap-server 192.168.247.128:9092 --topic mykafka123 --from-beginning

\#运行一个生产者
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka
\#运行一个消费者
bin/kafka-console-consumer.sh --zookeeper zookeeper:2181 --topic mykafka --from-beginning

\#分区数量的作用：有多少分区就能负载多少个消费者，生产者会自动分配给分区数据，每个消费者只消费自己分区的数据，每个分区有自己独立的offset
\#进入kafka容器
vi opt/kafka/config/server.properties
修改run.partitions=2
\#退出容器
ctrl+p+q
\#重启容器
docker restart kafka
​
\#修改指定topic
./kafka-topics.sh --zookeeper localhost:2181 --alter --partitions 3 --topic topicname
