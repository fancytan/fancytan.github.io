

# Nginx入门学习（第一回合）

[^应用]: 大型网站高并发处理
[^开发者]:  由俄罗斯的程序设计师Igor Sysoev所开发



[TOC]



## 一、产生背景

1.巨大流量—海量的并发访问

2.单台服务器资源和能力有限

引发服务器宕机而无法提供服务

## 二、负载均衡(Load Balance)

### 1、高并发

（1）、高（大量的）

（2）、并发就是可以使用多个线程或者多个进程，同时处理（就是并发）不同的操作

（3）、简而言之就是每秒内有多少个请求同时访问。

### 2、负载均衡

（1）、将请求/数据【均匀】分摊到多个操作单元上执行

（2）、关键在于【均匀】,也是分布式系统架构设计中必须考虑的因素之一。

### 3、互联网分布式架构

常见，分为客户端层、反向代理nginx层、站点层、服务层、数据层。只需要实现“将请求/数据 均匀分摊到多个操作单元上执行”，就能实现负载均衡。

## 三、对Nginx的基本了解

### 1、什么是Nginx？  

```txt
  一款轻量级的Web 服务器/反向代理服务器【后面有介绍】及电子邮件（IMAP/POP3）代理服务器。
```

​     `特点`

```
  *是占有内存少，CPU、内存等资源消耗却非常低，
  *运行非常稳定并发能力强，nginx的并发能力确实在同类型的网页服务器中表现非常好。
```



### 2、Nginx   VS   Apache

# （1）、nginx相对于apache的优点：

```txt
*轻量级，同样起web 服务，比apache 占用更少的内存及资源高并发，
*nginx 处理请求是异步非阻塞（如前端ajax）的，而apache 则是阻塞型的，
*在高并发下nginx能保持低资源低消耗高性能高度模块化的设计，编写模块相对简单
*Nginx 配置简洁, Apache 复杂
```



# （2）、apache 相对于nginx 的优点：

```txt
 * Rewrite重写 ，比nginx 的rewrite 强大模块超多，
 *基本想到的都可以找到少bug ，nginx 的bug 相对较多。（出身好起步高）
```

## 四、安装Nginx

`这里以安装tengine为例`

### 1、安装之前准备

配置依赖 gcc openssl-devel pcre-devel zlib-devel

 安装：

> yum install gcc openssl-devel pcre-devel zlib-devel -y

### 2、下载

（目前最新版）：[tengine-2.2.3.tar](http://tengine.taobao.org/download.html)

### 3、 解压缩

> tar  -zvxf   tengine-2.2.3.tar 

### 4、安装Nginx

> 在Nginx解压的目录下运行：
>
> ./configure
>
> make && make install

默认安装目录：
/usr/local/nginx

### 5、配置Nginx为系统服务，以方便管理

#### （1）、在/etc/rc.d/init.d/目录中建立文本文件nginx

####  （2）、在文件中粘贴下面的内容：

```txt
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
 
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
 
lockfile=/var/lock/subsys/nginx
 
make_dirs() {
   # make required directories
   user=`nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

#### （3）、修改nginx文件的执行权限

> chmod +x nginx

#### （4）、添加该文件到系统服务中去

> chkconfig --add nginx

查看是否添加成功

> chkconfig --list nginx

启动，停止，重新装载

> service nginx start|stop

## 五、Nginx配置

### 1、查看配置

> cd   /usr/local/nginx/conf
>
> vim   nginx.conf

### 2、配置解析

```txt
#进程数，建议设置和CPU个数一样或2倍
worker_processes  2;

#日志级别
error_log  logs/error.log  warning;(默认error级别)

# nginx 启动后的pid 存放位置
#pid        logs/nginx.pid;

events {
	#配置每个进程的连接数，总的连接数= worker_processes * worker_connections
    #默认1024
    worker_connections  10240;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
#连接超时时间，单位秒
keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost                 
        #默认请求
  		location / {
    				 root  html;   #定义服务器的默认网站根目录位置
   				  index  index.php index.html index.htm;  #定义首页索引文件的名称
        }
	    #定义错误提示页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```

### 3、负载均衡配置

安装Tomcat，参考 `Tomcat配置`

#### 多负载均执行一下操作：

 多负载的情况下，打开指定虚拟机器

> open  node01    
>
> node01  为指定虚拟机器的别名，在hosts文件中配置的

启动Tomcat

> 在Tomcat解压的目录下       ./startup.sh  

注意： 记得关闭虚拟机器的防火墙

> service  iptables  stop

浏览器访问

> 虚拟机器IP地址：8080

默认负载均衡配置

> ```
> http { 
>     upstream shsxt{   
>     # 以下均为实际执行服务的服务器
>     #只有当hosts文件中给ip地址配置了别名，这里server后面才能用别名，
>     #否则跟IP地址
>         server node01; 
>         server node02; 
>         server node03; 
>     } 
> 
>     server { 
>     #指定访问端口为80 ，那么Tomcat服务器端的port也要改为80
>         listen 80;   
> 	    server_name  localhost;
>         location / {
>             proxy_pass http://shsxt;    
>             # shsxt  是指定的代理服务器
>         }
>     } 
> }
> ```

配置文件编辑结束后，启动nginx服务

> service  nginx  start

#### （1）、轮询负载均衡（默认）

```
 - 对应用程序服务器的请求以循环方式分发
```

#### （2）、加权负载均衡

> 通过使用服务器权重，还可以进一步影响nginx负载均衡算法，
>
> 谁的权重越大，分发到的请求就越多。
>
> 权重总数：10

在nginx.conf文件中修改：

```
 upstream shsxt {
        server srv1.example.com weight=3;
        server srv2.example.com;
        server srv3.example.com;
  }
```

配置修改之后，重启

> service  nginx  restart

#### （3）、最少连接负载平衡

> 在连接负载最少的情况下，nginx会尽量避免将过多的请求分发给繁忙的应用程序服务器，
>
> 而是将新请求分发给不太繁忙的服务器，避免服务器过载。

在nginx.conf文件中修改：

```
upstream shsxt {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }
```

#### （4）、会话持久性------ip-hash负载平衡机制

`特点`：保证相同的客户端总是定向到相同的服务;

(此方法可确保来自同一客户端的请求将始终定向到同一台服务器，除非此服务器不可用。)

在nginx.conf文件中修改：

```
upstream shsxt{
    ip_hash;
    server （IP地址|别名）;
    server （IP地址|别名）;
    server （IP地址|别名）;
}
```

#### (5)、Nginx的访问控制

> Nginx还可以对IP的访问进行控制，allow代表允许，deny代表禁止.

```
location / {
deny 192.168.2.180;
allow 192.168.78.0/24;
allow 10.1.1.0/16;
allow 192.168.1.0/32;
deny all;
proxy_pass http://shsxt;
}
```

```
从上到下的顺序，匹配到了便跳出。
如上的例子先禁止了1个，
接下来允许了3个网段，
其中包含了一个ipv6，
最后未匹配的IP全部禁止访问
```

