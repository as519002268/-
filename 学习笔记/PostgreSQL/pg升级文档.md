# 一、  环境信息
```
Linux发行版本：CentOS release 6.7 (Final)
旧pg数据库版本：9.5.0
旧版本bin目录：/usr/local/pgsql/bin
旧版本data目录：/alidata1/data
旧版本WAL block size大小:8192
旧版本端口：5432
插件信息：pg_stat_statements V1.3 on schema basisdata
```
# 二、  安装pg10.3版本

## 1、下载源码文件至服务器

## 2、解码
`tar jxvf postgresql-10.0.tar.bz2`

## 3、编译（设置安装目录和安装选项）
```
cd postgresql-10.0
./configure --prefix=/usr/local/pgsql10 --with-pgport=5432 --with-wal-blocksize=8
```
若提示安装失败，原因是缺少GCC、readline和zlib，先查看本机是否安装，然后再根据情况自行安装

```
rpm -qa | grep gcc
yum search gcc
yum install gcc
readline和zlib安装方式相同
```
## 4、安装
```
gmake world
gmake install -world
```
## 5、创建data文件夹

`mkdir /usr/local/pgsql/data`

## 6、创建postgres用户和密码

```
adduser postgres
passwd postgres
```
## 7、修改data目录权限，将其赋给postgres

```
chown -R postgres:postgres /usr/local/pgsql10/data/
chmod -R 700 /usr/local/pgsql10/data/
```
## 8、设置环境变量

修改系统环境变量文件

`vim /etc/profile`

添加环境变量

```
LD_LIBRARY_PATH=/usr/local/pgsql10/lib --是psql命令可用
export  LD_LIBRARY_PATH
PGDATA=/usr/local/pgsql10/data
export  PGDATA
PATH=$PATH:/usr/local/pgsql10/bin --pg_ctl命令可用
export  PATH --使自己定义的变量生效，类似于全局变量
```
保存后，读取profile文件

`source profile`
##9、切换至postgres账号，初始化数据库

```
su - postgres
/usr/local/pgsql10/bin/initdb -E UTF8 -D /usr/local/pgsql10/data --locale=en_US.UTF-8 -U postgres -W
```
## 10、开启postgres服务
```
/usr/local/pgsql10/bin/pg_ctl start
/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data >logfile 2>&1 &
```
# 三、升级

## 1、新版本初始化成功以后，安装pg_upgrade插件
```
cd contrib 
gamke 
gmake install
```
## 2、修改配置文件postgresql.conf和pg_hba.conf
### 2.1、修改pg_hba.conf

因为升级需要多次连接新老集群数据库实例, 所以修改为使用本地trust认证

###2.2、修改postgresql.conf
将老版本的postgresql.conf文件拷贝至新版本的相应目录下，然后修改新版本的端口为5434（与编译时已经指定的端口相同）

## 3、停老库

`/usr/local/pgsql95/bin/pg_ctl stop`

## 4、创建upgrade文件夹，并赋权
```
su - root   
cd /usr/local/pgsql10
mkdir upgrade  
chown postgres：postgres upgrade   
su - postgres 
cd /usr/local/pgsql10/upgrade
```
## 5、修改profile文件，LD_LIBRARY_PATH指向新库
```
cd /etc
vim profile
LD_LIBRARY_PATH=/usr/local/pgsql95/lib
export LD_LIBRARY_PATH
```
## 6、检验更新

`/usr/local/pgsql10/bin/pg_upgrade -c -b /usr/local/pgsql95/bin -B /usr/local/pgsql10/bin -d /usr/local/pgsql95/data -D /usr/local/pgsql10/data -p 5432 -P 5434 -U postgres`

解决完上述问题后，重新检验，出现如下结果

说明新老数据库兼容，可以进行升级

##7、升级

升级有两种方式

一是：缺省的通过拷贝数据文件到新的data目录下，速度较慢，但原库可用

二是：创建硬链接，速度较快，但原库不可用，添加--link命令即可执行

由于数据量较大，本次测试是采用硬链接的方式

`/usr/local/pgsql10/bin/pg_upgrade --link -b /usr/local/pgsql95/bin -B /usr/local/pgsql10/bin -d /usr/local/pgsql95/data -D /usr/local/pgsql10/data -p 5432 -P 5434`

# 四、遇到的问题：
## 1、有三张表的字段不合法：
```
biuse.temp_xcm_0402;
biuse.temp_xcm_0402_2;
biuse.temp_xcm_0402_3;
```
解决方案：删除这三张表





## 2、生产环境安装了pg_stat_statements插件

由于生产环境安装了pg_stat_statements插件，而在升级的时候，新版本并未安装该插件，所以报错
解决方案：

1、在升级的时候，在新版本源码包的该目录下，需要make & make install相应的插件，具体
目录如下

`postgresql-10.0/contrib/pg_stat_statements`

2、然后修改老版本的配置文件postgresql.conf，将pg_stat_statements对应到组件暂时注释掉，然后再将老版本的配置文件拷贝到10.3版本对应的目录下，才能启动。

3、存在以pg_开头的用户

10.0版本开始是不支持以pg_开头的用户的，因为这是系统的默认用户
解决方案：收回该用户的所有权限，并删除该用户

```
revoke all on schema public from pg_rd;
drop role pg_rd;
```

升级检测成功，可以进行升级



4、wal-blocksize问题，由于该参数新老版本对不上，导致检验失败，解决方法是删除数据库，然后重新编译，并在编译是指定wal-blocksize的大小，然后重新检验

5、locale参数对不上，解决方案是删除data目录，重新初始化，并修改locale参数的指定值