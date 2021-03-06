
## 卸载原有的安装

`yum remove mysql`

## 安装依赖包

`yum -y install autoconf automake libtool cmake ncurses-devel openssl-devel lzo-devel zlib-devel gcc gcc-c++`

## 下载源码包
```
wget http://downloads.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.12.tar.gz
```

## 解压依赖包
```
tar zxvf boost_1_59_0.tar.gz
tar zxvf mysql-5.7.12.tar.gz
```

## 进入MySQL源码路径

`cd mysql-5.7.12`

## 编译
```
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/root/boost_1_59_0
```

## 安装
```
make && make install
cd /usr/local/mysql/bin
```

# 添加MySQL用户和密码

```
adduser mysql
passwd mysql
```

## 初始化数据库
```
./mysqld --initialize --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql --explicit_defaults_for_timestamp
```

## 修改MySQL配置文件

`vim /etc/my.cnf`

## 修改MySQL文件夹的权限

`chown -R mysql:mysql /usr/local/mysql/`

## 添加环境变量
```
echo "export PATH=$PATH:/usr/local/mysql/bin" >> /etc/profile
source /etc/profile
```

## 添加防火墙
```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

## 设置开机启动MySQL

`chkconfig mysqld on`

## 启动MySQL服务

`service mysqld start`

# 进入MySQL
```
mysqladmin -u root
set password =password('Dashu0701');
\q
```
