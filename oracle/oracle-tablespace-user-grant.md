# Oracle创建表空间、用户、分配权限语句

> 参考：[https://segmentfault.com/a/1190000006704150](https://segmentfault.com/a/1190000006704150)

以Oracle用户登录，命令：`sqlplus / as sysdba `，连接数据库

## 创建临时表空间

```sql
create temporary tablespace User_Temp tempfile '/u01/app/oracle/oradata/orcl/user_temp.dbf' size 200m autoextend on next 100m maxsize 20480m extent management local;

注:此步创建的是临时表空间，可以多个数据公用一个临时表空间，注意创建的大小即可，名称随意。
```

## 创建数据表空间

```sql
create tablespace User_Data logging datafile '/u01/app/oracle/oradata/orcl/user_data.dbf'  size 200m autoextend on next 100m maxsize 20480m extent management local;

注:此步注意数据表空间的名称最好与导出的备份文件所用的表空间名称一致，不一致容易报错！
```

## 创建用户并指定表空间

```sql
create user User_Name identified by "Passwd" default tablespace user_data temporary tablespace user_temp;

注:用户名最好与导出的备份文件的数据库名保持一致！
```

## 给用户授予权限

```sql
grant connect,resource,dba to User_Name;
```

## 修改用户密码

```sql
alter user [username] identified by [password];
```

## 删除用户

```sql
drop user User_Name cascade;  
```

## 删除表空间

```sql
DROP TABLESPACE 表空间名 INCLUDING CONTENTS AND DATAFILES;
```

## 清空某张表

```sql
TRUNCATE TABLE tableName #tableName是要清空表的表名
```