# [Redis](https://redis.io/documentation)

## 1. Getting Started

### 1.1. Introduction

> Redis is an advanced key-value store. It is similar to memcached but the dataset is not volatile, and values can be strings, exactly like in memcached, but also lists, sets, and ordered sets. All this data types can be manipulated with atomic operations to push/pop elements, add/remove elements, perform server side union, intersection, difference between sets, and so forth. Redis supports different kind of sorting abilities.

### 1.2. Intsall Redis

[download the latest stable version of Redis](https://redis.io/download)

本部分介绍如何在Linux上搭建Redis服务，包括Single、Replication、Sentinal、Cluster 四种模式，以及阐述为什么会出现这四种模式。

#### 1.2.1. Requirements

| GCC Version | Redis Version |
| :---------: | :-----------: |
|   GCC 9.+   |   Redis 6.+   |

#### 1.2.2. Upgrad GCC

获取系统默认的GCC版本：

```bash
$ gcc -v
# gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
# 如果上述命令不存在，尝试通过yum安装默认gcc
$ yum -y install gcc
$ yum -y install gcc-c++
$ gcc -v 
```

[download the gcc package](https://gcc.gnu.org/) or via devtools to install gcc [get the gcc devtools upgrade package ](http://mirrors.aliyun.com/centos/7/sclo/x86_64/rh/Packages/d/)  

**Devtools  upgrad GCC**

1. 下载升级GCC所需的devtools包:

- devtoolset-9-binutils-2.32-16.el7.x86_64.rpm  
- devtoolset-9-gcc-9.3.1-2.el7.x86_64.rpm
- devtoolset-9-gcc-c++-9.3.1-2.el7.x86_64.rpm 
- devtoolset-9-libstdc++-devel-9.3.1-2.el7.x86_64.rpm
- devtoolset-9-runtime-9.1-0.el7.x86_64.rpm  

2. 依次执行如下命令：

   ```bash
   # 若提示/usr/sbin/semanage is needed by devtoolset-9-runtime-9.1-0.el7.x86_64，则先安装semanage
   # $ yum provides semanage
   # $ yum -y install policycoreutils-python.x86_64
   # 若提示scl-utils >= 20120927-11 is needed by devtoolset-9-runtime-9.1-0.el7.x86_64，则安装scl-utils
   # $ yum -y install scl-utils
   $ rpm -ivh devtoolset-9-runtime-9.1-0.el7.x86_64.rpm
   $ rpm -ivh devtoolset-9-binutils-2.32-16.el7.x86_64.rpm
   $ rpm -ivh devtoolset-9-libstdc++-devel-9.3.1-2.el7.x86_64.rpm
   # 若提示glibc-devel版本过低时，则先升级glib-devel
   # $ yum install glibc-devel
   $ rpm -ivh devtoolset-9-gcc-9.3.1-2.el7.x86_64.rpm
   $ rpm -ivh devtoolset-9-gcc-c++-9.3.1-2.el7.x86_64.rpm
   ```

3. 配置环境变量：

   ```bash
   # 在当期会话中生效，退出则失效
   $ scl enable devtoolset-9 bash
   # 永久生效
   $ echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
   # 如果不生效则刷新下环境变量
   # $ source /etc/profile
   $ gcc -v
   # gcc version 9.3.1 20200408 (Red Hat 9.3.1-2) (GCC)
   ```

#### 1.2.2. Create a new user

我们建议您创建一个特定的新用户和用户组，用于管理应用软件。下述是一个创建用户的简单示例：

```bash
# 创建用户组`software`
$ groupadd software
# 创建用户`alen`并添加到组`software`，默认会在`home`目录下创建一个同名的用户主目录
$ useradd -g software alen
# 设置密码
$ passwdd alen
#查看用户信息
$ cat /etc/passwd	
```

#### 1.2.3. Standalone Redis

1. 登录服务器(用户`alen`)，解压redis软件包，并切换至该目录。 

   ```bash
   $ mkdir ~/software ~/programs
   $ tar -xvf ~/software/redis-6.2.1.tar.gz -C ~/programs/
   $ cd ~/programs/redis-6.2.1
   ```

2. 执行`make`编译redis文件。可使用`make distclean`清除编译的文件，这在编译出错或需重新编译时很有用！

   ```bash
   $ make
   ```
   
3. 切换至*src*目录，执行`make install`安装redis 服务。默认安装路径为`/usr/local/bin`，需保证您当前用户对该目录有足够的权限；可使用`PREFIX`指定安装路径。

   ```bash
   $ cd src
   # 此操作会在redis-6.2.1目录下生成一个bin目录，里面存放的是redis可执行文件
   $ make PREFIX=../ install 
   ```

4. 切换回*redis-6.2.1*目录，修改*redis.conf* 文件配置redis服务。

   ```properties
   # 绑定主机地址； 0.0.0.0表示绑定该主机上的所有IPV4地址，包括127.0.0.1
   bind 0.0.0.0
   # 绑定端口号；默认6379
   port 6379 
   # redis服务作为守护进程运行，否则将以前端方式启动
   daemonize yes 
   # 设置日志级别；debug (适用于开发或测试阶段); notice (适用于生产环境)
   loglevel debug
   # 设置日志文件存放路径及名称
   logfile "/home/alen/logs/redis/standalone.log"
   # 设置数据文件存放目录
   dir /home/alen/datas/redis
   # 关闭保护模式，允许远程连接；否则只能在主机内部访问，外部主机无法访问redis服务
   protected-mode no 
   # 开启aof持久化模式
   appendonly yes
   # aof持久化方式：no 不持久化；everysec 每秒一次；always 每次执行写操作时
   # always 最安全，其他都会发生丢失数据的可能，但同样也最消耗资源和影响性能，需要自行权衡
   appendfsync everysec
   # 设置redis服务密码
   requirepass alen@redis_aliyun@123zxc_com
   # 客户端链接超时时间；单位秒
   timeout 300
   ```

5. 切换至*bin*目录，执行`redis-server`并指定*redis.conf*作为配置文件启动redis服务。

   ```bash
   $ cd bin
   $ ./redis-server ../redis-conf
   # 查看redis服务进程
   $ ps -ef | grep redis
   ```

6. 执行`redis-cli`启动redis客户端验证。

   ```bash
   # 查看redis客户端帮助信息
   $ ./redis-cli --help
   # 当绑定ip地址为0.0.0.0，连接本机redis服务执行 ./redis-cli 即可
   $ ./redis-cli -h ip -p 6379
   # 也可使用 ./redis-cli -h ip -p 6379 -a <password>，但不安全，密码暴露在控制台
   127.0.0.1:6379> auth <passwd>
   127.0.0.1:6379> set number 10
   127.0.0.1:6379> get number
   # "10"
   127.0.0.1:6379> quit
   ```

## 2. Spring-Boot-Starter-Data-Redis

### 2.1. What Spring Data Redis?

Spring Data Redis 是基于Spring平台的一个用于操作Redis框架，提供了在Spring应用程序中轻松配置和访问Redis的功能，提供了用于与存储交互的低级和高级的抽象，消除了繁琐的配置和重复的模板代码！

### 2.2. Requirements

Spring Redis 需要Redis2.6+，Spring Data Redis 整合了 [Lettuce](https://github.com/lettuce-io/lettuce-core) 和 [Jedis](https://github.com/xetorthio/jedis)两种流行的操作Redis的开源库。

### 2.3. Getting Started

Using Maven：

```xml
<!--core: spring-data-redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.4.5</version>
</dependency>
```

Using gradle:

```groovy
dependencies {
  compile("spring-boot-starter-data-redis:2.4.5")
}
```

#### 2.2.1. Configuration

*org.springframework.boot.autoconfigure.data.redis.RedisProperties* 包函了Spring Redis可用的配置。

<table>
	<thead>
		<tr><th>属性名称</th><th>值</th><th>类型</th><th>描述</th></tr>
	</thead>
    <tbody>
    	<tr><td>database</td><td>0</td><td>int</td><td>redis数据库编号(0-15)</td></tr>
    	<tr><td>url</td><td>——</td><td>String</td><td>连接地址，该项会覆盖host、password、port配置; 
            <br/>格式：redis://user:password@example.com:6379</td></tr>
    	<tr><td>host</td><td>localhost</td><td>String</td><td>redis服务主机地址</td></tr>
    	<tr><td>username</td><td>——</td><td>String</td><td>登录redis服务的用户名</td></tr>
    	<tr><td>password</td><td>——</td><td>String</td><td>登录redis服务的用户名</td></tr>
    	<tr><td>port</td><td>6379</td><td>int</td><td>redis服务端口</td></tr>
    	<tr><td>ssl</td><td>false</td><td>boolean</td><td>是否开启ssl支持</td></tr>
    	<tr><td>timeout</td><td>——</td><td>Duration</td><td>读取超时时间</td></tr>
    	<tr><td>connect-timeout</td><td>——</td><td>Duration</td><td>客户端连接超时时间</td></tr>
        <tr><td>client-name</td><td>——</td><td>String</td><td>客户端名称</td></tr>
        <tr><td>client-type</td><td>LETTUCE/JEDIS</td><td>ClientType</td><td>客户端类型，默认会根据classpath下的依赖自动发现</td></tr>
        <tr><td>sentinel</td><td>——</td><td>Sentinel</td><td>Redis-Sentinel模式配置</td></tr>
        <tr><td>cluster</td><td>——</td><td>Cluster</td><td>Redis-Cluster模式配置</td></tr>
        <tr><td>jedis</td><td>——</td><td>Jedis</td><td>Jedis连接池配置</td></tr>
        <tr><td>lettuce</td><td>——</td><td>Lettuce</td><td>Lettuce连接池配置</td></tr>
    </tbody>
</table>

##### 2.2.1.1. Standalone Redis with Spring-Boot	

*application.yaml* 添加如下配置：

```yaml
spring:
  redis:
    client-name: alen
    host: 47.101.192.98
    password: alen@redis_aliyun@123zxc_com
    port: 6379
    timeout: 5s
    connect-timeout: 5m
    lettuce:
      pool:
        max-active: 8
        min-idle: 0
        max-idle: 8
      shutdown-timeout: 100ms
```

使用*org.springframework.data.redis.core.RedisTemplate* 操作Redis：

```java
@SpringBootTest
public class RedisApplicationTest {
    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Test
    public void giveValue_WhenStore_ThenGet(){
        String value = "10";
        redisTemplate.opsForValue().set("value", value);
        assertThat(redisTemplate.opsForValue().get("value")).isEqualTo(value);
    }
}
```
