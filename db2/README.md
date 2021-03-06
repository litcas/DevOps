# Db2 安装

[dbeaver](https://dbeaver.io/download/)

db2_developer_c

```
docker pull store/ibmcorp/db2_developer_c:11.1.4.4-x86_64
```


```
docker run -h db2server_developer_c --name db2server --restart=always  --detach --privileged=true -p 50000:50000 -p 55000:55000  --env-file /data/db2/.env_list  -v /data/db2/data/:/database store/ibmcorp/db2_developer_c:11.1.4.4-x86_64
```


```
docker exec -ti d4fc021e52aa bash -c "su - ${DB2INSTANCE}"
```



## db2创建数据库和导入导出数据库

### 1、切换用户

```shell
su - db2inst1

```
### 2、创建数据库

```shell
db2 create db databaseName using codeset utf-8 territory CN
```

注意：

`1.出现错误：SQL1004C there` `is` `not` `enough storage` `on` `the file` `to` `process the command``----物理空间不足`

`2.创建数据库失败（中间断掉或空间不足引起）无法重新创建，提示已存在，SQL1005N，此时需要删除掉重新创建解决方案：`

`查看是否存在系统数据库目录中`

```
db2 list db directory
```

`若存在则可以直接删除`
```
db2 drop db databasename
```
`注意系统创建的文件不能随意删除再复制回来需要修改权限否者出现SQL1036C  An I/O error occurred while accessing the` `database``.  SQLSTATE=58030`

`若不存在则需要添加进来再删除`
```
db2 catalog db databasename
```
`将数据库移除系统数据库目录中`
```
db2 uncatalog db databasename
```
### 3、连接新数据库
```
db2 connect to databaseName
```
### 4、创建BUFFERPOOL
```
db2 create BUFFERPOOL testBUFFER SIZE 1000 PAGESIZE 32K
```
### 5、创建TABLESPACE
```
db2 "create TABLESPACE testSpace PAGESIZE 32K MANAGED BY SYSTEM USING ('/home/db2inst1/test/ts') BUFFERPOOL testBUFFER"
```
### 6、创建临时表空间
```
db2 "create SYSTEM TEMPORARY TABLESPACE testBUF PAGESIZE 32 K MANAGED BY SYSTEM USING ('/home/db2inst1/epay/tts') BUFFERPOOL testBUFFER"
```
### 7、断开连接
```
db2 disconnect databaseName
```
### 8、使用db2inst1角色连接数据库
```
db2 connect to databaseName
```
### 9、创建用户和密码用于连接数据库（一个库下使用不同用户连接数据库管理各自的表（但是数据库名字是同一个，只需连接是指定用户名密码）类似oracle）

```
useradd username

passwd password
```

### 10、db2inst1进行授权

使用db2inst1连接数据库进行授权给指定用户后，该用户才可以有权访问表

```
db2 grant dbadm on database to user userName
```
### 11、使用创建用户重新连接数据库进行添加当前用户下的表

```
db2 connect to databaseName user username using password
```

### 12、导出数据库的所有表及数据（导出的文件是db2move.lst、export.out、tabxx.ixf、tabxx.msg）
```
`db2move <数据库名> export`
```
### 13、导入数据库的所有表及数据（需要修改db2move.lst中的第一个字段是用户名，管理需要导入的表，不需要导入的直接删除即可也可以修改用户）
```
`db2move <数据库名> import`
```
### 14、导出表创建语句
```
`db2look -d <数据库名> -u <用户> -e -o <脚本名称>.sql`
```
### 15、运行sql脚本
```
`db2 -tvf  <脚本名称>.sql`
```
### 16、导出单个表数据（只能导出一个表）
```
db2 export to test.txt of del select * from test
```
### 17、导入单个表数据

```
db2 import from test.txt of del insert into test
```
总结：

针对数据库操作千万不要手动删除数据库文件

db2可以创建多个库，也可以只创建一个数据库使用不用的用户进行登录，管理各自的表


```


db2 get db cfg|grep -i buff
list tablespaces
list tablespaces show detail
db2pd -d testdb -tablespaces
get snapshot for tablespaces on testdb
也可以查看sysibmadm.snaptbsp和sysibmadm.snapcontainer这两个视图

```

```sql
--重启数据库
FORCE APPLICATION ALL 
DB2STOP
DB2START 

--创建数据库
CREATE DATABASE mysdedb USING CODESET UTF-8 TERRITORY US COLLATE USING SYSTEM USER TABLESPACE MANAGED BY DATABASE USING (FILE 'd:\DB2\data\mysdedb\sdetbsp' 51200)

CONNECT TO mysdedb 

--创建缓冲池(使用32k的pagesize)
create bufferpool sdepool size 12800 pagesize 32K
create bufferpool sdepool1 size 12800 pagesize 32K

--创建表空间并使用32k的pagesize和自定义的缓冲池
CREATE REGULAR TABLESPACE regtbs PAGESIZE 32 K MANAGED BY DATABASE USING ( FILE 'C:\DB2\NODE0000\mysdedb\regtbs' 2g) bufferpool sdepool
CREATE REGULAR TABLESPACE idxtbs PAGESIZE 32 K MANAGED BY DATABASE USING ( FILE 'C:\DB2\NODE0000\mysdedb\idxtbs' 1g) bufferpool sdepool
CREATE LONG TABLESPACE lobtbs PAGESIZE 32 K MANAGED BY DATABASE USING ( FILE 'C:\DB2\NODE0000\mysdedb\lobtbs' 1g) bufferpool sdepool1
CREATE USER TEMPORARY TABLESPACE sdespace PAGESIZE 32 K MANAGED BY SYSTEM USING ('C:\DB2\NODE0000\mysdedb\sdespace' ) bufferpool sdepool1

--授权表空间给用户
grant use of tablespace 

--授权表空间
GRANT USE OF TABLESPACE regtbs TO PUBLIC  
GRANT USE OF TABLESPACE lobtbs TO PUBLIC 
GRANT USE OF TABLESPACE idxtbs TO PUBLIC  
GRANT USE OF TABLESPACE sdespace TO PUBLIC 

COMMENT ON TABLESPACE sdespace IS '' 

--优化数据库配置
update db cfg for mysdedb using APPLHEAPSZ 2048
update db cfg for mysdedb using APP_CTL_HEAP_SZ 2048
update db cfg for mysdedb using LOGPRIMARY 10
update db cfg for mysdedb using LOGFILSIZ 1000

--重启数据库
FORCE APPLICATION ALL 
DB2STOP FORCE 
DB2START


--授予sde用户DBADM权限
grant DBADM on database to user sde

--重启数据库
FORCE APPLICATION ALL 
DB2STOP FORCE 
DB2START

```

```
#创建pagesize为32K的bufferpool
db2 => create BUFFERPOOL bigbuffer SIZE 5000 PAGESIZE 32K
DB20000I  The SQL command completed successfully.

#创建pagesize为32K的tablespace，同时使用新创建的bufferpool
db2 => CREATE TABLESPACE bigtablespace PAGESIZE 32K BUFFERPOOL bigbuffer
DB20000I  The SQL command completed successfully.

```

现在再来看一下表空间的情况
```
db2 => select TBSPACE, OWNER, PAGESIZE from syscat.tablespaces
```
接下来再来创建这张表，就OK了
```
db2 => CREATE TABLE BOND_BASE_INFO ( SRNO INTEGER NOT NULL , ANCMNT_DATE DATE,  ...)
DB20000I  The SQL command completed successfully.
```
 不过，如果你不想要使用多个表空间的话，也可以将之前tablespace中的数据导入到新的tablespace中，这里DB2支持直接导入到指定的tablespace，db2 databasename import -ts tablespace_name。


# 1. 查看所有的表空间
db2 list tablespaces

# 2. 查看某个表空间的容器
db2 list tablespace containers for   [ show detail ]
如： db2 list tablespace containers for 3 show detail

# 3. 查看表空间的配置信息，使用情况及容器信息
db2pd -d -tablespaces
如：db2pd -d zdb -tablespaces

# 4. 通过snapshot 获取表空间信息
db2 get snapshot for tablespaces on


对于DMS( database manage ) 表空间，提供了几个方法更改表空间容器。
    1. Add 用来增加新的容器
    2. Drop 删除容器
    3. Extend 扩展已有容器大小
    4. Reduce 缩减已有容器大小
    5. Resize 重新设定容器大小
Add和Drop操作，表空间容器之间会发生数据重新平衡(rebalance)，Reduce和Resize操作，需要确保修改后的表空间容器有足够的空间，否则DB2会拒绝该操作。
日常运维中出现表空间满的情况时，根据对存储空间和对运维的影响，有以下三种方案：
1). 如果表空间对应的存储容器中还有未分配空间，可通过alter tablespace 的 extend 或 resize 选项扩展已有表空间容器的大小。如下面的例子是在每个容器上扩展了50GB：
    db2 "alter tablespace tbs_dat extend ( file '/db2tbsp/dat_1' 50G, file '/db2tbsp/dat_2' 50G) ";

2). 如果表空间容器对应的存储中没有剩下空间时，可以通过alter tablespace 的 add 选项增加新的表空间容器。需要注意的是：通过add增加容器会在容器之间进行数据Reblance，即数据重新平衡。如果数据量很大，reblance的时间会很长，对系统性能会造成很大的影响。下面是增加一个新的容器，表空间的不同容器内会发生reblance:
    db2 "alter tablespace tbs_dat add ( file '/db2tbsp/dat_2' 10G)"

3). 通过alter tablespace begin new stripe set 选项。begin new stripe set 选项是当已有容器使用完后，再使用新增加的容器。与方法2不同，该选项不会在容器之间做reblance，不会对系统造成性能影响，但它会造成数据偏移。如下：
    db2 "alter tablespace tbs_dat begin new stripe set (file '/db2tbsp/dat_2' 10G) "

# 删除表空间tbs_dat的容器dat_2
db2 "alter tablespace tbs_dat drop ( file '/db2tbsp/dat_2' )"


表空间容器的路径变化时，需要建立一个软链接(容器的原路径软链接到容器的新路径)，如：
ln -s /db2tbs/zdb/dat_1 /db2tbsp/dat_1

