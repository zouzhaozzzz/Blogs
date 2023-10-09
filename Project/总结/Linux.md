

# Linux

# 1. 安装

## 1.1 硬件和系统方面：

bios设置：

​	Intel Virtualization Technology 、Vt-x、Hyper-V（高级 > CPU设置）

系统设置：

​	在`启用或关闭Windows功能`中勾选如下几个选项：

- Hyper-V
- 适用于Linux的Windows子系统
- 虚拟机平台

## 1.2 通过WSL安装Ubuntu

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

# 2. SSH

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

# 3. 进程查询

```shell
# 查询进程及相应的编号
ps -ef | grep ssh

# 查询相应端口的占用情况
sudo -lsof -i:22

# 杀死进程
kill -9 进程编号（PID）
```

# 4. JDK的安装

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

# 5. 部署项目

首先在`IDEA`中使用`maven`对项目进行打包

```shell
mvn package
```

> 在打包之前，根据`OS`修改`application.yml`中的DB的访问路径，如果是访问`windows`下的DB，需要关闭`windows`的防火墙

# 6. 杂项准备工作

由于在默认情况下，MySQL只允许在本地访问，所以需要使用MySQL官方的客户端工具，对访问权限进行设置：

`Administrator -> Users & Privileges`设置能访问的域为%（10.41.250.%），表示任意机器都能访问



设置Ubuntu的时区和Windows保持一致：

```shell
# 查看当前系统的时区信息
date -R
# 设置时区
timedatectl set-timezone Asia/Shanghai
```



# 7. Nginx安装

windows和Linux的安装， 直接解压即可

```shell
# 启动
nginx
# 退出
nginx -s quit
# 重新加载
nginx -s reload
```

## 7.1 简单安装方式

```shell
# 切换到管理员用户
sudo su
# 通过Linux的软件源直接安装
apt-get install nginx
```

nginx与正向代理（代理的是客户端）的区别，就在于它是代理的它身后的各个服务器

## 7.2 通过官方源码编译&安装

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

# 8. 部署前端项目

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

# 9. 部署后端项目

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

## 配置负载均衡

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



# 10.命令

## 查看端口:netstat -anp |grep port

## 文件系统 容量 已用 可用 已用占比  挂载点 (列标题)  df -h 

## 文件系统 fdisk blkid

## 内存

- free -h

- ps aux --sort -rss

- top

## 解压

​	tar -zxvf 				 tar -zcvf

| 参数 | 含义                                                |
| ---- | --------------------------------------------------- |
| tar  | Linux压缩/解压缩命令                                |
| -z   | 代表gzip，使用gzip工具进行压缩或解压                |
| -x   | 代表extract，解压文件（压缩文件是-c）               |
| -v   | 代表verbose，显示解压过程（文件列表）               |
| -f   | 代表file，指定要解压的文件名（or 要压缩成的文件名） |

压缩单个文件

```bash
zip [包名].zip [文件]
```

压缩目录

```bash
zip -r [包名].zip [文件]
```

解压

```bash
unzip [包名].zip

unzip [包名].zip -d 目录  #解压到指定目录
```



## 清除Buff/Cache

~~~markdown
1）清理pagecache（页面缓存）
echo 1 > /proc/sys/vm/drop_caches或sysctl -w vm.drop_caches=1
 
2）清理dentries（目录缓存）和inodes
echo 2 > /proc/sys/vm/drop_caches或sysctl -w vm.drop_caches=2
 
3）清理pagecache、dentries和inodes
echo 3 > /proc/sys/vm/drop_caches或sysctl -w vm.drop_caches=3
~~~







## mysql docker

~~~shell
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
~~~



## kafka docker

~~~shell
docker pull wurstmeister/zookeeper
docker pull wurstmeister/kafka
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper

docker run  \
-d --name kafka  \
-p 9092:9092 -e KAFKA_BROKER_ID=0  \
-e KAFKA_ZOOKEEPER_CONNECT=192.168.247.128:2181/kafka  \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.247.128:9092  \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092  \
-e JVM_XMS=512m \
-e JVM_XMX=512m  \
wurstmeister/kafka

docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=172.24.20.219:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.24.20.219:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka

docker run  \
-d --name kafka  \
-p 9092:9092 -e KAFKA_BROKER_ID=0  \
-e KAFKA_ZOOKEEPER_CONNECT=172.27.89.168:2181/kafka  \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.27.89.168:9092  \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092  \
-e JVM_XMS=512m \
-e JVM_XMX=512m  \
wurstmeister/kafka

#进入容器
docker exec -it ${CONTAINER ID} /bin/bash
cd opt/bin
\#单机方式：创建一个主题
kafka-topics.sh --create --zookeeper 192.168.247.128:2181/kafka --replication-factor 1 --partitions 1 --topic mykafka
\#查看主题
kafka-topics.sh --zookeeper 192.168.247.128:2181/kafka --list
kafka-topics.sh --zookeeper 192.168.247.128:2181/kafka --describe --topic mykafka
\#查看consumer Group列表
kafka-consumer-groups.sh --bootstrap-server 192.168.247.128:9092 --list
kafka-consumer-groups.sh --bootstrap-server 192.168.247.128:9092 --describe --group consumer-group-mykafka

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
~~~



~~~shell
docker run -d --name zookeeper_sasl -p 2181:2181 --rm -it -e SERVER_JVMFLAGS="-Djava.security.auth.login.config=/opt/zookeeper-3.4.13/secrets/server_jaas.conf" -v /usr/xxx/kafka-sasl/conf:/opt/zookeeper-3.4.13/conf -v /usr/xxx/kafka-sasl/conf:/opt/zookeeper-3.4.13/secrets/ wurstmeister/zookeeper
~~~







## nacos

~~~shell
docker run \
-d --name nacos \
-p 8848:8848 \
--privileged=true \
--restart=always \
-e JVM_XMS=516m \
-e JVM_XMX=5 \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=172.24.20.219 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_DB_NAME=nacos \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=Zouzhao@0418 \
-e NACOS_AUTH_ENABLE=true \
-e NACOS_AUTH_TOKEN=U2VjcmV0S2V5MjAyMzA0MjcwOTIyem91emhhbzIwMjMwNDI3MDkyMg==  \
-v /root/docker/nacos/logs:/home/nacos/logs \
-v /root/docker/nacos/init.d/custom.properties:/etc/nacos/init.d/custom.properties \
-v /root/docker/nacos/data:/home/nacos/data \
nacos/nacos-server:v2.1.2
~~~

```sql
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
  `src_user` text,
  `src_ip` varchar(20) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE users (
 username varchar(50) NOT NULL PRIMARY KEY,
 password varchar(500) NOT NULL,
 enabled boolean NOT NULL
);

CREATE TABLE roles (
 username varchar(50) NOT NULL,
 role varchar(50) NOT NULL,
 constraint uk_username_role UNIQUE (username,role)
);

CREATE TABLE permissions (
    role varchar(50) NOT NULL,
    resource varchar(512) NOT NULL,
    action varchar(8) NOT NULL,
    constraint uk_role_permission UNIQUE (role,resource,action)
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```





## java

```
tar -zxvf jdk-8u202-linux-x64.tar.gz -C /usr/local/java
 
vim /etc/profile
 
export JAVA_HOME=/usr/local/java/jdk1.8.0_202
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export  PATH=${JAVA_HOME}/bin:$PATH


source /etc/profile
```





## mysql

~~~shell
下面安装mysql5.7

wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
yum localinstall mysql57-community-release-el7-8.noarch.rpm
yum install mysql-community-server --nogpgcheck
注意带上–nogpgcheck

设置开机启动mysql服务
systemctl enable mysqld
systemctl daemon-reload

查看mysql启动状态
systemctl status mysqld.service



查看MySQL初始密码,这个密码是随机生成的

grep 'A temporary password' /var/log/mysqld.log

// 重制密码
会有弱密码提示，建议设置复杂点或者自己配置关闭

SET PASSWORD FOR 'root'@'localhost' = PASSWORD('Zouzhao@0418');


保存重启mysql

systemctl restart mysqld.service

至此，root密码被修改了，但是root账户还不能远程登录

UPDATE mysql.user SET host = '%' WHERE user='root';
~~~



## nginx

~~~shell
1.安装依赖包

//一键安装上面四个依赖
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
2.下载并解压安装包

//创建一个文件夹
cd /usr/local
mkdir nginx
cd nginx
//下载tar包
wget http://nginx.org/download/nginx-1.13.7.tar.gz
tar -xvf nginx-1.13.7.tar.gz
3.安装nginx

//进入nginx目录
cd /usr/local/nginx
//进入目录
cd nginx-1.13.7
//执行命令 考虑到后续安装ssl证书 添加两个模块
./configure --with-http_stub_status_module --with-http_ssl_module
//执行make命令
make
//执行make install命令
make install
4.启动nginx服务

 /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
4.配置nginx.conf

打开配置文件

vi /usr/local/nginx/conf/nginx.conf

将端口号改成8089(随便挑个端口)，因为可能apeache占用80端口，apeache端口尽量不要修改，我们选择修改[nginx](https://so.csdn.net/so/search?q=nginx&spm=1001.2101.3001.7020)端口。

将localhost修改为你服务器的公网ip地址。

**5.重启nginx**

```python
/usr/local/nginx/sbin/nginx -s reload
```

**编译报错**

https://blog.csdn.net/succing/article/details/120526532


~~~



## elasticsearch

~~~shell
sudo docker pull elasticsearch:7.12.0
elasticsearch:7.12.0:我安装的版本是7.12.0，可以根据实际的情况安装

创建docker容器挂在的目录：
sudo mkdir -p /opt/elasticsearch/config
sudo mkdir -p /opt/elasticsearch/data
sudo mkdir -p /opt/elasticsearch/plugins

配置文件:
echo "http.host: 0.0.0.0" >> /opt/elasticsearch/config/elasticsearch.yml

异常一：文件夹未设置所有用户读写执行权限，处理：
sudo chmod -R 777 /opt/elasticsearch/

sudo docker run --name es -p 9200:9200  -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms84m -Xmx1024m" \
-v /opt/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /opt/elasticsearch/data:/usr/share/elasticsearch/data \
-v /opt/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.12.0


进入容器
docker exec -it es01 /bin/bash
重置密码
bin/elasticsearch-reset-password -u elastic
提示是否重置，输入y，控制台会打印新密码，请记住这个密码，稍后要用到
Password for the [elastic] user successfully reset.
New value: oN0YYrSffs76Pko9gQ=v

#elasticsearch.yml 文件添加
cluster.name: "docker-cluster-01"
network.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
# 此处开启xpack
xpack.security.enabled: true

#进入容器初始化密码
./bin/elasticsearch-setup-passwords interactive

~~~





## git

~~~shell
git config --global --list

git config --global user.name yaoc
git config --global user.ema yaoc
~~~

