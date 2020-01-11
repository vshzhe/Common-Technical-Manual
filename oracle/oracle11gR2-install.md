# Oracle 11gR2 Install Centos7

> 参考：[https://www.cnblogs.com/xibei666/p/5935219.html](https://www.cnblogs.com/xibei666/p/5935219.html)

## 创建运行oracle数据库的系统用户和用户组

用Root账号登录，运行下面指令，创建所需要用户和用户组

```shell
groupadd oinstall　　　　　　　　　　　　　#创建用户组oinstall
groupadd dba　　      　　　　　　　　　　 #创建用户组dba
useradd -g oinstall -G dba -m oracle　　 #创建oracle用户，并加入到oinstall和dba用户组
                                        #-g为指定用户的主要组为oinstall组
                                        #-G为指定用户的次要组为dba组
groups oracle  　　　　　　　　　　　　　　#查询用户组是否授权成功
passwd oracle　　     　　　　　　　　　　 #设置用户oracle的登陆密码，不设置密码，在CentOS的图形登陆界面没法登陆
id oracle                   　　　　　　  #查看新建的oracle用户
```


## 创建oracle数据库安装目录

```shell
mkdir -p /data/oracle　　#oracle数据库安装目录
mkdir -p /data/oraInventory　　#oracle数据库配置文件目录
mkdir -p /data/database　　#oracle数据库软件包解压目录
cd /data
ls　　#创建完毕检查一下
chown -R oracle:oinstall /data/oracle　　#设置目录所有者为oinstall用户组的oracle用户
chown -R oracle:oinstall /data/oraInventory
chown -R oracle:oinstall /data/database
```

## 修改OS系统标识

oracle默认不支持CentOS系统安装， 修改文件 /etc/RedHat-release 内容为RedHat-7

```shell
vi /etc/redhat-release#修改成红色部分文字
redhat-7
```

## 安装oracle数据库所需要的软件包

以下是按照需要依赖的安装包，通过 yum install {包名} 来验证是否安装，例如yum install binutils

```shell
binutils-2.23.52.0.1-12.el7.x86_64 
compat-libcap1-1.10-3.el7.x86_64 
gcc-4.8.2-3.el7.x86_64 
gcc-c++-4.8.2-3.el7.x86_64 
glibc-2.17-36.el7.i686 
glibc-2.17-36.el7.x86_64 
glibc-devel-2.17-36.el7.i686 
glibc-devel-2.17-36.el7.x86_64 
ksh
libaio-0.3.109-9.el7.i686 
libaio-0.3.109-9.el7.x86_64 
libaio-devel-0.3.109-9.el7.i686 
libaio-devel-0.3.109-9.el7.x86_64 
libgcc-4.8.2-3.el7.i686 
libgcc-4.8.2-3.el7.x86_64 
libstdc++-4.8.2-3.el7.i686 
libstdc++-4.8.2-3.el7.x86_64 
libstdc++-devel-4.8.2-3.el7.i686 
libstdc++-devel-4.8.2-3.el7.x86_64 
libXi-1.7.2-1.el7.i686 
libXi-1.7.2-1.el7.x86_64 
libXtst-1.2.2-1.el7.i686 
libXtst-1.2.2-1.el7.x86_64 
make-3.82-19.el7.x86_64 
sysstat-10.1.5-1.el7.x86_64
```

-- 使用下面指令，检查依赖软件包

```shell
yum install binutils-2.* compat-libstdc++-33* elfutils-libelf-0.* elfutils-libelf-devel-* gcc-4.* gcc-c++-4.* glibc-2.* glibc-common-2.* glibc-devel-2.* glibc-headers-2.* ksh-2* libaio-0.* libaio-devel-0.* libgcc-4.* libstdc++-4.* libstdc++-devel-4.* make-3.* sysstat-7.* unixODBC-2.* unixODBC-devel-2.* pdksh*
```

## 关闭防火墙和selinux

## 修改内核参数

```shell
vi /etc/sysctl.conf #要添加sysctl.conf内容

net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.rp_filter = 1
fs.file-max = 6815744 #设置最大打开文件数
fs.aio-max-nr = 1048576
kernel.shmall = 2097152 #共享内存的总量，8G内存设置：2097152*4k/1024/1024
kernel.shmmax = 2147483648 #最大共享内存的段大小
kernel.shmmni = 4096 #整个系统共享内存端的最大数
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500 #可使用的IPv4端口范围
net.core.rmem_default = 262144
net.core.rmem_max= 4194304
net.core.wmem_default= 262144
net.core.wmem_max= 1048576
```

## 对oracle用户设置限制，提高软件运行性能

```shell
vi /etc/security/limits.conf  #要添加到Limits.conf内容

oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
```

## 配置用户的环境变量

```shell
vi /home/oracle/.bash_profile  #要追加bash_profile内容部分

export ORACLE_BASE=/data/oracle #oracle数据库安装目录
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1 #oracle数据库路径
export ORACLE_SID=orcl #oracle启动数据库实例名
export ORACLE_TERM=xterm #xterm窗口模式安装
export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH #添加系统环境变量
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib #添加系统环境变量
export LANG=C #防止安装过程出现乱码
export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK  #设置Oracle客户端字符集，必须与Oracle安装时设置的字符集保持一致

运行source /home/oracle/.bash_profile时上述配置生效
```

## 获取安装包文件后解压安装包

获取安装包文件的方式，可通过ftp服务器，也可通过wget下载到指定目录，解压方式如下

```shell
unzip linux.x64_11gR2_database_1of2.zip -d /data/database/  #解压文件1
unzip linux.x64_11gR2_database_2of2.zip -d /data/database/  #解压文件2
chown -R oracle:oinstall /data/database/database/　　　　　　 #分配安装文件授权Oracle
```
