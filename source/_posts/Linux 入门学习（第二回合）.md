# Linux 入门学习（第二回合）

`2018年12月18日  周二  晴`

**今日学习要点：**



[TOC]

## Linux后半程

### 一、Linux系统配置

#### 1.主机名配置：

> vim  /etc/sysconfig/network

![1545214876278](C:\Users\ADMINI~1\AppData\Local\Temp\1545214876278.png)

配置完成之后需要重启机器才能生效

> reboot

#### 2.DNS配置

> # 查看DNS服务器的地址
>
> cat  /etc/resolv.conf
>
> # 修改DNS服务器地址
>
> 方式一：vim  /etc/sysconfig/network.scripts/ifconfig-eth0
>
> ​             在配置网关时，配置DNS1=114.114.114.114（不推荐，江苏南京的IP）
>
> 方式二：vim   /etc/resolv.conf    （用本地网关解析）
>
> ​              nameserver   192.168.198.0   ( 此为虚拟机中的网关地址)

#### 3.环境变量

> 配置系统环境变量，使得某些命令在执行时，系统可以找到命令对应的执行程序，命令才能正常执行。

查看系统一共在哪些目录里寻找命令对应的程序

> 命令：echo $PATH

![1545215894631](C:\Users\ADMINI~1\AppData\Local\Temp\1545215894631.png)

`注意`：路径之间有冒号隔开，系统会从左往右依次寻找对应的程序

​             一般命令会存放在  bin目录，或sbin目录

配置全局环境变量：

> vim  /etc/profile
>
> 在文件中：
>
> PATH=$PATH:(命令所在目录)
>
> 退出文件编辑后：
>
> source  /etc/profile  
>
>  (重新加载资源，有的可能需要重启机器，这不适用于实际状况)



配置局部环境变量：（推荐，限当前登录用户使用）

> 查看所有文件(root目录下)
>
> ls  -a    (发现隐藏文件    .bash.profile)
>
> vim  ~/ bash_profile
>
> 在文件中：
>
> export  PATH =$PATH:(命令所在目录)

#### 4.拍快照

（保存当时计算机所出状态的各种配置和资源，适度使用）

> 选中指定虚拟计算机------鼠标右击-----选中“快照” ------“拍摄快照‘----在页面中找到”拍摄快照“，并添加名称和描述
>
> 也可以删除，找到页面中的删除按钮

### 二、服务操作

#### 1、查询操作系统

在每一个执行等级中会执行哪些系统服务，其中包括各类常驻服务。

> 命令：chkconfig

![1545219619458](C:\Users\ADMINI~1\AppData\Local\Temp\1545219619458.png)

```txt
各数字代表的系统初始化级别：
    0：停机状态
　　1：单用户模式，root账户进行操作
　　2：多用户，不能使用net file system，一般很少用
　　3：完全多用户，一部分启动，一部分不启动，命令行界面
　　4：未使用、未定义的保留模式
　　5：图形化，3级别中启动的进程都启动，并且会启动一部分图形界面进程。
　　6：停止所有进程，卸载文件系统，重新启动(reboot)
```

> 1、2、4很少用，0、3、5、6常用，3级别和5级别除了桌面相关的进程外没有什么区别，推荐都用3级别；
>
> linux默认级别为3；
>
> 不要把 /etc/inittab  中 initdefault 设置为0 和 6； 

#### 2、服务操作

> service 服务名 start/stop/status/restart

举例：对防火墙服务进行操作

> 防火墙的服务名为：iptables

查看防火墙服务运行状态

> service  iptables status

关闭防火墙

> service  iptables stop

开启防火墙

> service  iptables start

永久开启/关闭防火墙

> chkconfig iptables on/off

#### 3、服务初执行等级更改

> chkconfig --level 2345 `name` off|on
>                                    （ `服务名`）
>
> 举例：防火墙
>
> chkconfig --level 2345 iptables   off

若不加级别，默认是2345级别

> 命令：chkconfig `name` on|off
>                          （`服务名`）

### 三、linux进程操作

#### 1、查看所有进程

> 命令： ps  -aux

```txt
    -a 列出所有
	-u 列出用户
	-x 详细列出，如cpu、内存等
     -e select all processes    相当于-a
     -f does full-format listing   将所有格式详细列出来      
```

查看所有进程里CMD是ssh 的进程信息（包括pid 进程号）

> 命令： ps  - ef | grep ssh      
>
> （|  管道符  ：前一个输出，变为后一个的输入）
>
> 举例：
>
> ps -ef | grep redis

#### 2、杀死进程

`kill`

> 命令：kill pid 
>
> -9    强制杀死
>
> 用法：用ps 命令先查出对应程序的PID或PPID ，然后用kill杀死掉进程。

### 四、其他常用命令

#### 1、**yum**

| 基于[RPM](https://baike.baidu.com/item/RPM)包管理            |
| ------------------------------------------------------------ |
| 能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装 |

跟换yum下载源（默认是到国外网站下载）

第一步：备份你的原镜像文件，以免出错后可以恢复

> cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

第二步：下载新的CentOS-Base.rep到/etc/yum.repos.d/

> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

下载完之后，查看一下文件内容

> vim  /etc/yum.repos.d/CentOS-Base.repo

第三步：生成缓存

> 运行yum makecache

查看当前源

> yum list | head -50

#### 2、 **wget**

| 一个从网络上自动下载文件的自由工具                           |
| ------------------------------------------------------------ |
| 支持通过 HTTP、HTTPS、FTP 三个最常见的 TCP/IP协议，可以使用 HTTP 代理 |

安装：

> yum install wget  –y

用法：

> wget  [option] 网址  -O	指定下载保存的路径
>
> 举例：
>
> wget  www.baidu.com  -O baidu.html

#### 3、**tar**

```txt
    -z	gzip进行解压或压缩，带.gz需要加，压缩出来.gz也需要加
	-x	解压
	-c	压缩
	-f	目标文件，压缩文件新命名或解压文件名
	-v	解压缩过程信息打印
```

> 解压命令：tar  -zvxf  xxxx.tar.gz

> 压缩命令：tar -zcf 压缩包命名 压缩目标
> 举例：
>
> tar -zcf  tomcat.tar.gz  apache-tomcat-7.0.61 
> 将 apache-tomcat-7.0.61 目录压缩成tomcat.tar.gz包

#### 4、**man**

作用：用于查看指定命令的具体解释

安装

> yum install  man  -y
>
> (下载并安装man  并确认)

使用

> man  ps

### 五、JDK部署

#### 1、准备JDK安装包：

（这是使用   .rpm  格式的安装包）

官网下载：http://www.oracle.com/technetwork/java/javase/downloads/index.html

云盘资源：  [jdk-8u191-linux-x64.rpm](https://pan.baidu.com/s/1LzeQbOnG9PROZrtlbYHlDg)  ：

​     根据用户喜好放到虚拟机器的文件目录中

#### 2、解压并安装，展示编译过程

> rpm   -ivh  jdk-8u191-linux-x64.rpm

安装放到了 /usr 目录下，有/java目录

#### 3、配置环境变量

> vim  ~/.bash_profile
>
> 在文件中：
>
> JAVA_HOME=(jdk文件所在的路径+jdk文件名)
>
> export  PATH=$PATH:$JAVA_HOME/bin

注意：

新的path路径必须要包含旧的PATH路径，且每个路径之间以冒号隔开，而不是分号

> 配置完成，退出编辑框后
>
> source  ~/.hash_profile

#### 4、测试：

> java  -version
>
> 或
>
> echo  $JAVA_HOME

`echo   标准输出，打印`

### 六、Tomcat部署

#### 1、官网下载

http://tomcat.apache.org/

云盘资源：[apache-tomcat-7.0.61.tar](https://pan.baidu.com/s/1e7OSIf3K9YpFCzXncMj_9Q)

#### 2、上传并解压

> tar -zvxf  apache-tomcat-7.0.61.tar

#### 3、启动tomcat

在tomcat的bin目录下有个startup.sh 脚本可以直接启动tomcat服务

> ./startup.sh

#### 4、关闭tomcat服务

方式一：可以用shutdown.sh命令

方式二：ps -ef | grep tomcat 查看出tomcat进程号后，用kill命令

#### 5、验证

先把防火墙关了（service iptables stop），然后访问虚拟机IP的8080端口