# Redis

## 简介

`Redis`本质上是一个基于内存来存储、索引数据的数据库（NoSQL），一般我们部署Redis的服务器，内存都会堆的非常高，而磁盘和CPU相对次一点，并且它们会组成一个集群，常年开机，提供内存的服务。

所以对于一般的Java系统，我们存储的高频数据（热点数据），就不再存储在本地内存，而是存储在其它机器的内存。

NoSQL：[https://baike.baidu.com/item/NoSQL/8828247?fr=aladdin](https://baike.baidu.com/item/NoSQL/8828247?fr=aladdin)

Redis就是一种典型的非关系型数据库，除此之外，还有MongoDB。对于Redis而言，存储数据主要以key-value的形式保存。

适合使用Redis的合适场景：

验证码？用后即弃，阅后即焚，所以严格来说，并不是完全符合使用Redis的场景

- 本身带有键值对的特性
- 体积不能过大
- 高频使用的热点数据（最好这些数据相对静态，不要频繁修改，比如类似0-男 1-女字典数据）
- 客户端对于读取这样的数据，要求响应速度一定要快
- 多台系统想要共享的数据（token、secret-Key、乐观锁中用到的版本号）

## Redis在Linux环境下的安装

官网下载地址 [https://download.redis.io/releases/redis-6.2.7.tar.gz](https://download.redis.io/releases/redis-6.2.7.tar.gz)

```bash
# 上传文件至ubuntu，然后解压文件
tar -zxvf redis.6.2.7.jar

# 进入解压后的redis目录
cd redis-6.2.7

# 安装编译环境(build-essential里面集成了构建C、C++所需要的必要的库)
sudo apt-get install build-essential

# 编译
make
# 安装（由于要在系统文件夹下生成一些文件，所以需要管理员权限）
sudo make install

# 启动redis服务(默认是使用单机模式standalone)
redis-server

# 通过redis-cli关闭redis服务
redis-cli shutdown
kill -9 进程编号（可以使用sudo lsof -i:6379 查看哪个进程使用6379端口）

# 启动安装包中自带的简单命令行工具
redis-cli

# 进入cli以后，可以使用quit退出cli

# 开启可以远程访问的模式启动服务
redis-server --protected-mode no
```

配置文件常识：

- 默认受保护模式启动，要修改的话，需要设置`protected-mode no`
- 端口 `port 6379`
- `logfile ""` 默认输出到/dev/null
- `daemonize no` 默认以非后台方式运行
- `databases 16` 一共有16个数据库实例，索引从0~15，可以通过`select index`来进行切换，默认使用的0号数据库实例

以配置文件的方式启动`Redis`服务：

```bash
# 在redis目录下有redis.conf文件，首先进行备份，留待以后使用
cp redis.conf redis_backup.conf

# 编辑
vi redis.conf

# 修改如下几个内容
############分割线##############
# 限定哪些ip的机器能访问当前redis
# bind 127.0.0.1 -::1

# 关闭受保护模式
protected-mode no

# 以后台方式启动
daemonize yes

# 指定日志文件
logfile "/home/think/logs/redis.log"
############分割线##############

# 在启动的时候，指定具体的配置文件
redis-server redis.conf

# 以追加的方式查看运行日志
tail -f redis.log
```

## API

针对常见几种数据类型的操作（详见文档）：

- string
- hash
- list
- set
- zset

## Jedis

添加依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.3.0</version>
</dependency>
```

模板代码：

```java
try (
    // 创建连接Redis的连接池
    JedisPool pool = new JedisPool("10.41.250.140", 6379);
    // 从pool中获取一个连接，当作客户端接口使用
    Jedis jedis = pool.getResource()
) {
    String msg = jedis.ping();
    System.out.println(msg);
}
```

练习：

```bash
// 编写一个测试案例，测试redis的平均响应速度
// 使用set方法先初始化若干数据，例如k1=v1 k2=v2....k10=v10
// 然后循环10次，调用get方法随机访问刚才初始化好的数据，求出平均响应时间
```

## SpringBoot整合Redis



## 生产案例