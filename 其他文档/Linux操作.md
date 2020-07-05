# Linux 操作笔记

[TOC]



## 基本信息

```
> 账号密码 
 -> root
 -> zzg19950824
 
> vm net 范围 
 -> 192.168.158.2
 -> CentOS 静态 ip : 192.168.158.20
```



## 常用命令 

```java
> 查看 IP 信息 --- ifconfig
> 访问网络 --- telnet 
  -> 查看 telnet 服务 --- rpm -qa telnet-serverc
> 重启网络 --- service network restart
> 查看路由 --- route

// 创建目录
> 创建目录 --- mkdir ~/temp
> 创建路径上所有的目录 --- mkdir -p dir1/dir2/dir3/dir4/

// 系统
> 查看磁盘使用情况 --- df -k
> 比例显示磁盘 --- df -h
> 显示文件系统类型 --- df -T
> 查看当前工作目录 --- pwd
> 输出环境变量 --- export | grep ORCALE 
> 终止进程 --- kill -9 7243
    // -9 表示强制终止指定进程

// 权限控制
> 

// 查看
> 查看多个文件的内容 --- cat filename
> 显示最后10行数的文本 --- tail filename
> 显示指定行数的文本 --- tail -n N filename.txt 
	: tail -n 1000 ESC-CFG_20000.log
> 查看内存 --- free -m
        

// 搜索
> 文件中查找字符串 --- grep -i "the" demo_file
> 查找指定文件名的文件 --- find -iname "MyProgram.c"
> 对查找的文件执行命令 --- find -iname "MyProgram.c" -exec md5sum {} \;
> 查找所有的空文件 --- find ~ -empty 

// 挂载
> 挂载系统 --- mount /dev/sdb1 /u01

// copu
> 拷贝文件，保持权限 --- cp -p file1 file2
> 拷贝覆盖 --- cp -i file1 file2


// 压缩
> 创建一个tar 文件 --- tar cvf archive_name.tar dirname/
> 解压 tar 文件：tar xvf archive_name.tar
> 查看 tar 文件：tar tvf archive_name.tar

    
// 关机
shutdown -h now   
// 重启    
shutdown -r now
```



## 操作流程

### 更改阿里yum

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.tencentyun.com/centos


> 更新缓存
yum makecache

> 更新系统
yum -y update

> 腾讯开源镜像网站 -> https://mirrors.cloud.tencent.com/

http://mirrors.ustc.edu.cn/mysql-ftp/Downloads/

http://uni.mirrors.163.com/mysql/Downloads/
```



### VM 使用

```
> 安装  VMTools
 -> 查看 VMTools 是否安装成功 --- rpm VMWareTools

> 编辑文档
 -> vi filename
 -> 按 i 进入编辑
 -> vm 中 shift + Esc 退出
 -> :wq 

> 联网 ： 
  -> vi /etc/sysconfig/network-scripts/ifcfg-ens33
  -> ip addr
  
> 安装 yum
  -> 查看 yum 版本 --- rpm –qa|grep yum
  
  
> 安装 telnet
  ->  检查 是否安装 --- rpm -qa telnet-server
  ->  安装 telnet --- yum install telnet-server 
  
>  修改DNS  
nameserver 192.168.158.1
nameserver 8.8.8.8
nameserver 114.114.114.114
```

### 更新 yum

```
1 > yum update
2 > yum install -y yum-utils  --- 添加 yum 工具
3 >  yum-config-manager --add-repo xxx
	: yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
4 > 更新索引
	: yum clean all
	: yum makecache
	: yum makecache fast
	
```

### Linux 安装 Java

```
> 查看 JAVA 版本
rpm -qa |grep java

> 查看 yum Java 版本
yum list java*

> 安装 JDK
yum -y install java-1.8.0-openjdk

> 安装 JDK
  -> 检查是否已经安装 JDK --- yum list installed | grep java
  -> 查看 JDK 安装列表 --- yum search java | grep -i --color jdk 
  
> 查看版本信息
java-version
  
```



### Linux 安装 Mysql

```java
> 安装包 , 需要的 , 别省略
: wget http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
rpm -ivh mysql57-community-release-el7-11.noarch.rpm

> yum 安装 mysql
: yum -y install mysql57-community-release-el7-11.noarch.rpm

> 查看安装成果
> yum repolist enabled | grep mysql.*

> 安装 mysql 服务器
> yum install mysql-community-server
// 安装过慢 --> 
	1 http://mirrors.ustc.edu.cn/mysql-ftp/Downloads/MySQL-5.7/
	2 找到对应的rpm
	3 上传到指定目录 --> /var/cache/yum/x86_64/7/mysql57-community/packages
       

> 启动 mysql 服务
> systemctl start  mysqld.service

> 查看状态
> systemctl status mysqld.service

> 查看初始化密码
> grep "password" /var/log/mysqld.log

> 登录
mysql -uroot -p

> 修改初始化密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Zzg!@123456';

ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
ALTER USER USER() IDENTIFIED BY '123456';
: 旧版 SET PASSWORD = PASSWORD('123456'); 
> 查看密码策略
SHOW VARIABLES LIKE 'validate_password%'; 
> 修改密码策略等级(0=low -> 长度8位以上)
set global validate_password_policy=0;
set global validate_password_policy=LOW;
select @@validate_password_length;  -- 查看密码长度
set global validate_password_length=1; -- 修改密码长度

> 数据库授权
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;


> 刷新
FLUSH PRIVILEGES;

> 设置自启动ng
systemctl enable mysqld

systemctl daemon-reload

> 开启端口
firewall-cmd --zone=public --add-port=389/tcp --permanent
firewall-cmd --permanent --zone=public --add-port=3306/tcp
```

### Linux 安装 zoopkeeper

```
1 > 官网下载 zookeeper , apache-zookeeper-3.5.6-bin.tar.gz

2 > 解压缩
	tar zxvf apache-zookeeper-3.5.6-bin.tar.gz -C /usr/local/
	
3 > 修改配置文件
	cd /usr/local/apache-zookeeper-3.5.6-bin/conf
	cp zoo_sample.cfg zoo.cfg
	
4 > 修改配置文件

# 客户端与zk服务端的心跳时间
tickTime=2000
initLimit=10
syncLimit=5
clientPort=2181
dataDir=/usr/local/apache-zookeeper-3.5.6-bin/data
dataLogDir=/usr/local/apache-zookeeper-3.5.6-bin/log

# 集群配置
server.11=hu-hadoop1:2888:3888
server.12=hu-hadoop2:2888:3888
server.13=hu-hadoop3:2888:3888

5 > 环境配置
vi /etc/profile
/usr/local/apache-zookeeper-3.5.6-bin/bin/zkEnv.sh
: 添加 -- export ZOOKEEPER_HOME=/usr/local/apache-zookeeper-3.5.6-bin/


: 修改 log
if [ "x${ZOO_LOG4J_PROP}" = "x" ]
then
    ZOO_LOG4J_PROP="INFO,ROLLINGFILE"
fi
if [ -f "${ZOOCFGDIR}/zookeeper-env.sh" ]; then
  . "${ZOOCFGDIR}/zookeeper-env.sh"
fi

6 > 修改 log
/usr/local/apache-zookeeper-3.5.6-bin/conf/log4j.properties
zookeeper.root.logger=INFO, ROLLINGFILE
log4j.appender.ROLLINGFILE=org.apache.log4j.DailyRollingFileAppender

7 > zookeeper 启动
1. 启动ZK服务:       sh bin/zkServer.sh start
2. 查看ZK服务状态:   sh bin/zkServer.sh status
3. 停止ZK服务:       sh bin/zkServer.sh stop
4. 重启ZK服务:       sh bin/zkServer.sh restart

```

### linux 按照 Jenkins

```
> 修改源(可以不用)
yum install yum-fastestmirror -y

> 添加jenkins源
wget -O /etc/yum.repos.d/jenkins.repo http://jenkins-ci.org/redhat/jenkins.repo
rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key

> yum 安装
yum install jenkins

--> http://mirrors.ustc.edu.cn/jenkins/redhat/
	-> kins-2.226-1.1.noarch.rpm
	-> /var/cache/yum/x86_64/7/jenkins/packages

> 查看是否开启
systemctl status jenkins

> 查看防火墙
firewall-cmd --list-all

> 开放防火墙
systemctl start firewalld.service

> mask 防火墙
systemctl unmask firewalld.service

> 开放防火墙8080 端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-port=18088/tcp --permanent
> 重启防火墙
systemctl restart firewalld.service


> 启动 Jenkins
service jenkins start
service jenkins restart
service jenkins stop

// 失败可通过 systemctl status jenkins 检查愿意  ,细读 , 比如 java 环境
> 配置文件路径
/etc/sysconfig/jenkins
```

### Linux 安装 Maven

```
wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo

yum -y install apache-maven

sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
sudo yum install -y maven
 
mvn -version


vi /etc/profile

export MAVEN_HOME="/opt/apache-maven-3.5.4"
export PATH="$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH"
```

### Linux 安装 Nexus-3 OSS

```java
> 下载 -->  wget https://www.sonatype.com/oss-thank-you-tar.gz

> 解压 
 tar -zxvf nexus-3.21.1-01-unix.tar.gz

> 修改路径
mv nexus-3.21.1-01 nexus

> 修改环境变量
 /etc/profile
export NEXUS_HOME=/opt/nexus 

    
// ------- root 账户无法启动
> 创建nexus用户
useradd nexus
> 为用户的启动目录设置 -R （读取）权限即可
chown -R nexus:nexus /opt/nexus
chown -R nexus:nexus /opt/sonatype-work
> 切换用户
su nexus
> cd到解压目录,进行启动
./nexus start
    
 // ---------- 配置 : 以nexus账号运行Nexus
sudo adduser nexus
sudo chown -R nexus:nexus /opt/nexus

修改/opt/nexus/bin/nexus.rc，编辑内容为run_as_user="nexus"

> 开启端口
firewall-cmd --zone=public --add-port=8081/tcp --permanent


> 以服务方式运行Nexus  -> 创建文件 /etc/systemd/system/nexus.service
[Unit]
Description=nexus service
After=network.target
  
[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort
  
[Install]
WantedBy=multi-user.target

> 启动 (注意奇幻账号)
systemctl daemon-reload
systemctl enable nexus
systemctl start nexus


> 状态
systemctl status nexus -l
```



### Linux 安装 MyCat

```
> 下载 MyCat , 官网下载
http://www.mycat.io/

> tar zxvf Mycat-server-1.6.7.1-release-20190627191042-linux.tar.gz -C /usr/local/

> 服务器启动和配置
- 添加环境变量 --- vi /etc/profile
	: MYCAT_HOME=/usr/local/mycat
- 刷新
	: source /etc/profile
	
> 启动 (bin 目录下)
./mycat start
```



### Linux 安装 Java

```
 > 查看Java 状态
 rpm -qa|grep java  
 ? rpm -ev
 
 > 下载Java JDK 
 https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
 
 > 解压缩
 tar -zxvf jdk-8u231-linux-x64.tar.gz -C /usr/local/src/
 
 > 配置环境变量
 vim /etc/profile
 export ZOOKEEPER_HOME=/usr/local/src/jdk1.8.0_231
export JAVA_HOME=/usr/local/src/jdk1.8.0_231
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib

> 刷新
source /etc/profile

> 查看版本
java -version


```



### Linux 安装 Wget

```
> yum -y install wget
```

### Linux 按照 npm

```
> yum install nodejs
	|- npm -v

```



## centOS 使用

```
> 
```



## 常见问题

##### yum doesn't have enough cached data to continue

```
1  >  将/etc/yum.repos.d/epel.repo中的mirrorlist改为baseurl
2  >   /etc/resolv.conf文件中增加nameserver 144.144.144.144

[root@ec-cache ~]# vi /etc/resolv.conf

# Generated by NetworkManager
nameserver 192.168.1.2
nameserver 144.144.144.144
```

##### Linux 可以连接内网无法连接外网

```
-> NAT 模式
-> 查询 /etc/sysconfig/network-scripts/ifcfg-ens33 是否配置

-> 设置虚拟DNS --- vi /etc/NetworkManager/NetworkManager.conf
  -> 最后添加一行 --- dns=none

-> 重新启动网络管理 --- systemctl restart NetworkManager.service
 

```

<https://blog.csdn.net/weixin_42859280/article/details/89281862>

## 附录

### yum 命令

```

#查看软件包
  yum list all                     ##列出yum源仓库里面的所有可用的安装包 
  yum list installed               ##列出所有已经安装的安装包  
  yum list available               ##列出没有安装的安装包
 #安装软件
  yum install softwarename         ##安装指定的软件
  yum reinstall softarename        ##重新安装指定的软件
  yum localinstall 第三方software   ##安装第三方文件并且会解决软件的依赖关系
  yum remove  softwarename         ##卸装指定的软件
 #查找软件的信息  
  yum info software                ##查看软的信息
  yum search keywords              ##根据关键字查找到相关安装包软件的信息
  yum whatprovides filename        ##查找包含指定文件的相关安装包
 #对于软件组
   yum groups list             ##列出软件组
   yum groups install         ##安装一个软件组
   yum group remove           ##卸载一个软件组
   yum groups info            ##查看一个软件组的信息

> 使用日志
yum history info

yum repolist
```



#### Linux ifcfg 各项参数

```java
TYPE=Ethernet                # 网卡类型：为以太网
PROXY_METHOD=none            # 代理方式：关闭状态
BROWSER_ONLY=no                # 只是浏览器：否
BOOTPROTO=dhcp                # 网卡的引导协议：DHCP[中文名称: 动态主机配置协议]
DEFROUTE=yes                # 默认路由：是, 不明白的可以百度关键词 `默认路由` 
IPV4_FAILURE_FATAL=no        # 是不开启IPV4致命错误检测：否
IPV6INIT=yes  --- # IPV6是否自动初始化: 是[不会有任何影响, 现在还没用到IPV6]
IPV6_AUTOCONF=yes --- # IPV6是否自动配置：是[不会有任何影响, 现在还没用到IPV6]
IPV6_DEFROUTE=yes            # IPV6是否可以为默认路由：是[不会有任何影响, 现在还没用到IPV6]
IPV6_FAILURE_FATAL=no        # 是不开启IPV6致命错误检测：否
IPV6_ADDR_GEN_MODE=stable-privacy   --- IPV6地址生成模型：stable-privacy [这只一种生成IPV6的策略]
NAME=ens33                    # 网卡物理设备名称
UUID=f47bde51-fa78-4f79-b68f-d5dd90cfc698    # 通用唯一识别码, 每一个网卡都会有, 不能重复, 否两台linux只有一台网卡可用
DEVICE=ens33                    # 网卡设备名称, 必须和 `NAME` 值一样
ONBOOT=no  
IPADDR=”192.168.0.101” #192.168.59.x, x为3~255. 
NETMASK=”255.255.255.0” #子网掩码 
GATEWAY=”192.168.66.2” #网关IP


// 个人
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=ae352fd1-1117-4087-8310-809f972936b4
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.158.20
NETMASK=255.255.255.0
GATEWAY=192.168.158.2
DNS1=192.168.158.2
    
// 注意拼写不要出现问题
```

