# 安装Oracle 11g2 一些错误的处理

安装前，按照Oracle官方文档进行准备工作，主要是安装一些必要的软件包。64位系统，没有装i686或i386的包，只装了x86_64的。

## ins_ctx.mk

安装过程中，在`link binaries`阶段出现2个错误，第一个是关于`ins_ctx.mk`，`log`显示：
`/lib64/libstdc++.so.5: undefined reference to 'memcpy@GLIBC_2.14'`

原因据说是由于本机的`glibc`版本高于`2.14`（实际为`2.17`）。解决方法：

* yum install glibc-static

该软件包包含一个静态链接库：`/usr/lib64/libc.a`
修改`/app/oracle/product/11.2.0/db_1/ctx/lib/ins_ctx.mk`，将

```vim
将
vim /app/oracle/product/11.2.0/db_1/ctx/lib/ins_ctx.mk
ctxhx: $(CTXHXOBJ)
$(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK)
修改为：
ctxhx: $(CTXHXOBJ)
-static $(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK) /usr/lib64/stdc.a

点击Retry即可
```

## ins_emagent.mk

是`Error in invoking target 'agent nmhs' of makefile'app/oracle/product/11.2.0/db_1/sysman/lib/ins_emagent.mk.'`

解决方法，在`makefile`中添加链接`libnnz11`库的参数：

```vim
vim /app/oracle/product/11.2.0/db_1/sysman/lib/ins_emagent.mk

将
$(MK_EMAGENT_NMECTL)
修改为：
$(MK_EMAGENT_NMECTL) -lnnz11

点击Retry即可
```

## ORA-39006, ORA-39065错误解决办法

执行expdp命令时，出现如下现象

```vim
ORA-39006: internal error  
ORA-39065: unexpected master process exception in DISPATCH  
ORA-01403: no data found  
ORA-39097: Data Pump job encountered unexpected error 100
```

**解决方案**

执行如下脚本重新生成DATAPUMP API用到的视图和表即可

```sql
sqlplus / as sysdba
SQL>@$ORACLE_HOME/rdbms/admin/catmeta.sql 
SQL>@$ORACLE_HOME/rdbms/admin/catmet2.sql 
SQL>@$ORACLE_HOME/rdbms/admin/utlrp.sql
```

## ORA-00845错误解决方法

启动实例时，出现如下现象

```sql
SQL> startup
ORA-00845: MEMORY_TARGET not supported on this system

[root@oracle11g ~]# df -k /dev/shm
Filesystem 1K-blocks Used Available Use% Mounted on
tmpfs 3072000 1374176 1697824 45% /dev/shm
```

解决方案

`Oracle`在`metalink`的文档：`Doc ID: Note:460506.1`中进行了说明。解决这个问题只有两个方法，一种是修改初始化参数，使得初始化参数中SGA的设置小于`/dev/shm`的大小，另一种方法就是调整`/dev/shm`的大小。

1. 通过修改`/dev/shm`的大小可以通过修改`/etc/fstab`文件来实现
2. 修改文件,启动数据库

    ```shell
    $ vim /etc/fstab

    $ tmpfs /dev/shm tmpfs defaults,size=3000m 0 0 #不存在$ 的话，可以新增
    $ umount /dev/shm # 修改/etc/fstab，重新mount /dev/shm
    $ mount /dev/shm
    ```
