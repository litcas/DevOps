# Linux安装MySQL5.7

下载地址：https://dev.mysql.com/downloads/mysql/5.7.html#downloads

**解压**
```
tar -xvf mysql-5.7.26-linux-glibc2.12-x86_64.tar 
```
**再移动并重命名一下**

```
mv mysql-5.7.26-linux-glibc2.12-x86_64 /usr/local/mysql
```
**创建mysql用户组和用户并修改权限**

```
groupadd mysql
useradd -r -g mysql mysql
```

**创建数据目录并赋予权限**

```
mkdir -p  /data/mysql              #创建目录
chown mysql:mysql -R /data/mysql   #赋予权限
```

**配置my.cnf**

`vim /etc/my.cnf`

```
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
#character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
```

**初始化数据库**
进入mysql的bin目录

```
cd /usr/local/mysql/bin/
```
**初始化**
```
yum install -y libaio 

./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize
```

**查看密码**

```
cat /data/mysql/mysql.err
```

#### 启动mysql，并更改root 密码

```
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql

service mysql start

ps -ef|grep mysql
```
**用SET PASSWORD命令**
 
 ```
格式：mysql> set password for 用户名@localhost = password('新密码'); 
例子：mysql> set password for root@localhost = password('123'); 
 ```
 
查看当前mysql的大小写敏感配置

```
show global variables like '%lower_case%';
lower_case_file_system
表示当前系统文件是否大小写敏感，只读参数，无法修改。
ON  大小写不敏感 
OFF 大小写敏感 

进入docker的MySQL容器，编辑/etc/mysql/mysql.conf.d/mysqld.cnf文件，在[mysqld]下添加如下：
[mysqld] 
lower_case_table_names=1

保存，退出容器；
```

## Mysql 远程登录及常用命令

### mysql服务的启动和停止
```
   net stop mysql

   net start mysql
```
### 登陆mysql
```
   语法如下： mysql -u用户名 -p用户密码

   键入命令mysql -uroot -p， 回车后提示你输入密码，输入12345，然后回车即可进入到mysql中了，mysql的提示符是：

mysql>
```
注意，如果是连接到另外的机器上，则需要加入一个参数-h机器IP

### 开启远程访问权限

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'XXXXXXX'  
```
### 增加新用户

   格式：grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码"

   如，增加一个用户user1密码为password1，让其可以在本机上登录， 并对所有数  据库有查询、插入、修改、删除的权限。首先用以root用户连入mysql，然后键入以下命令：
```
   grant select,insert,update,delete on *.* to user1@localhost Identified by "password1";
```
如果希望该用户能够在任何机器上登陆mysql，则将localhost改为"%"。

如果你不想user1有密码，可以再打一个命令将密码去掉。
```
grant select,insert,update,delete on mydb.* to user1@localhost identified by "";
```
 

###  操作数据库

   登录到mysql中，然后在mysql的提示符下运行下列命令，每个命令以分号结束。

1、 显示数据库列表。
```
show databases;
```
缺省有两个数据库：mysql和test。 mysql库存放着mysql的系统和用户权限信息，我们改密码和新增用户，实际上就是对这个库进行操作。

2、 显示库中的数据表：
```
   use mysql;

   show tables;
```
3、 显示数据表的结构：
```
    describe 表名;
```
4、 建库与删库：
```
   create database 库名;

   drop database 库名;
```
5、 建表：
```
   use 库名;

  create table 表名(字段列表);

  drop table 表名;
```
6、 清空表中记录：
```
  delete from 表名;
```
7、 显示表中的记录：
```
  select * from 表名;
```
 
### 快速复制数据库

创建新的数据库

```
mysql -u root -p123456

mysql>CREATE DATABASE `newdb` DEFAULT CHARACTER SET UTF8 COLLATE UTF8_GENERAL_CI;
```
复制数据库，使用mysqldump及mysql的命令组合，一次性完成复制

```
mysqldump db1 -u root -p123456 --add-drop-table | mysql newdb -u root -p123456
```
不在同一个mysql服务器上

```
mysqldump db1 -uroot -p123456 --add-drop-table | mysql -h 192.168.1.22 newdb -u root -p123456
```

### 导出和导入数据

1. 导出数据：
```
   mysqldump --opt test > mysql.test
```
即将数据库test数据库导出到mysql.test文件，后者是一个文本文件
```
如：mysqldump -u root -p123456 --databases dbname > mysql.dbname

就是把数据库dbname导出到文件mysql.dbname中。
```
2. 导入数据:
```
   mysqlimport -u root -p123456 < mysql.dbname。
```
不用解释了吧。

3. 将文本数据导入数据库:

文本数据的字段数据之间用tab键隔开。
```
   use test;

   load data local infile "文件名" into table 表名;
```
1:使用SHOW语句找出在服务器上当前存在什么数据库：
```
   mysql> SHOW DATABASES;
```
2:2、创建一个数据库MYSQLDATA
```
  mysql> CREATE DATABASE MYSQLDATA;
```
3:选择你所创建的数据库
```
  mysql> USE MYSQLDATA; (按回车键出现Database changed 时说明操作成功!)
```
4:查看现在的数据库中存在什么表
```
  mysql> SHOW TABLES;
```
5:创建一个数据库表
```
  mysql> CREATE TABLE MYTABLE (name VARCHAR(20), sex CHAR(1));
```
6:显示表的结构：
```
  mysql> DESCRIBE MYTABLE;
```
7:往表中加入记录
```
  mysql> insert into MYTABLE values ("hyq","M");
```
8:用文本方式将数据装入数据库表中(例如D:/mysql.txt)
```
  mysql> LOAD DATA LOCAL INFILE "D:/mysql.txt" INTO TABLE MYTABLE;
```
9:导入.sql文件命令(例如D:/mysql.sql)
```
  mysql>use database;

  mysql>source d:/mysql.sql;
```
10:删除表
```
  mysql>drop TABLE MYTABLE;
```
11:清空表
```
  mysql>delete from MYTABLE;
```
12:更新表中数据
```
  mysql>update MYTABLE set sex="f" where name='hyq';

```
13：备份数据库
```
  mysqldump -u root 库名>xxx.data
```
14：例2：连接到远程主机上的MYSQL
```
假设远程主机的IP为：110.110.110.110，用户名为root,密码为abcd123。则键入以下命令：

  mysql -h110.110.110.110 -uroot -pabcd123                       // 远程登录
```
(注:u与root可以不用加空格，其它也一样)

3、退出MYSQL命令： exit (回车)

### 行锁分析

* 执行 SQL 分析命令

```
show status like 'innodb_row_lock%';
```

* Variable_name 说明

  *  Innodb_row_lock_current_waits：当前正在等待锁定的数量。

  *  Innodb_row_lock_time：从系统启动到现在锁定的时长。

  *  Innodb_row_lock_time_avg：每次等待锁所花平均时间。

  *  Innodb_row_lock_time_max：从系统启动到现在锁等待最长的一次所花的时间。

  *  Innodb_row_lock_waits：系统启动后到现在总共等待锁的次数。

### mysql 忽略大小写配置

1.新安装mysql5.7版本后，linux环境下默认是大小写敏感的。可以在客户端执行以下命令:

```
SHOW VARIABLES LIKE '%case%'
```
可以看到 lower_case_table_names 的值是 0, 我们要做的就是把它设置成 1. 具体步骤如下:

2. 使用 vi /etc/mysql/my.cnf, 打开mysql 的配置文件, 在 mysqld 这个节点下, 加入:

```
lower_case_table_names=1
```
注意: 一定要放在 mysqld 节点下 , 放在其他节点下, 会不生效 !!!!

设置完后, 保存重启mysql 服务,再执行`SHOW VARIABLES LIKE '%case%'`.

在重启mysql 服务时, 会用到 restart 命令

```
service mysqld restart 

```

## mysql 默认链接数是100个 最大是16384。

查看最大连接数:
show variables like '%max_connections%';

显示当前状态：mysql> show status

Threads_running    1 ##（当前连接数）

设置新的最大连接数为200：mysql> set GLOBAL max_connections=200

需要重启数据库
修改 my.conf 
```
max_connection = 1000;
```

### 慢查询

```
[mysqlld] 
long_query_time=2 
#5.0、5.1等版本配置如下选项 
log-slow-queries="mysql_slow_query.log" 
#5.5及以上版本配置如下选项 
slow-query-log=On 
slow_query_log_file="mysql_slow_query.log" 
log-query-not-using-indexes 
```
