---
title: postgresql编译安装
date: 2023-02-17 17:33:00
categories:
- Postgresql
tags:
- 编译
- 二进制
- 源码
---

### postgresql编译安装

#### 准备

需要提前安装`postgresql`依赖`gcc g++ zlib perl openssl readline `。

在官方网站https://vault.centos.org/7.6.1810/os/x86_64/Packages/ 查询依赖需要的rpm安装包，将所需要的依赖包都放在同一个文件，避免循环依赖问题

```shell
rpm  -ivh  *.rpm --nodeps --force
```

安装完成确认依赖是否安装完成 

```shell
gcc -v
g++ -v
perl -version
```

其他涉及依赖根据编译提示错误问题依次安装，若碰到依赖循环使用强制安装`--nodeps --force`忽略二次依赖问题。

#### `postgresql`安装

##### 编译安装

准备源码 `postgresql-12.5.tar.bz2`

```shell
# 解压
tar  jxf postgresql-12.5.tar.bz2
# 预编译指定目录
./configure --prefix=/usr/local/pgsql
# 预编译没有问题执行
make && make install
```

##### 更改配置

源码自带启动脚本，位于**/contrib/start-scripts**下，将`linux`文件复制到 `/etc/init.d/pgsql`下，更改数据库和启动脚本所属用户和用户组为`postgres`。

```shell
vi /etc/profile
```

添加如下配置

```shell
PGDATA=/usr/local/pgsql/data
export PGDATA
PGHOME=/usr/local/pgsql
export PGHOME
PATH=$PATH:$PGHOME/bin:$PGDATA
export PATH PGDATA
```

路径位置根据实际安装路径配置，预编译指定了`--prefix=/usr/local/pgsql`安装路径。

```shell
source /etc/profile
```

##### 初始化

```shell
su postgres
initdb
```

以上`postgresql`安装完成

### 编译安装postgis

安装postgis前需要安装需要的依赖

- `PostgreSQL`  ——  `PostGIS`构建于`PostgreSQL`之上，所以`PostgreSQL`必须要安装。
- `GNU Make`（`gmake`或`make`）——  这个也是用于编译源码。
- `Proj4`  ——  `Proj4` 重投影库用于在`PostGIS`中提供坐标重投影功能。
- `GEOS`  ——  `GEOS`几何图形库，用于支持`PostGIS`中的几何信息处理、分析等功能，也可以直接认为`GEOS`是一个几何算法库。
- `LibXML2`  ——  `LibXML2`目前用于`PostGIS`中的一些导入函数，比如`ST_GeomFromGML()`和`ST_GeomFromKML()`。
- `JSON-C`  ——  目前使用`JSON-C`通过`ST_GeomFromGeoJSON()`函数导入`GeoJSON`格式的数据
- `GDAL`  ——  用于`PostGIS`对栅格数据的支持。

* `SFCGAL`  ——  用于提供额外的二维和三维的高级分析功能。允许对一些函数使用基于SFCGAL的实现，而不是使用基于GEOS的实现（例如ST_Intersection()和ST_Area()函数）
* `protobuf`和`protobuf-c`

#### `cmake`安装

解压`cmake-3.9.0-rc2-Linux-x86_64.tar.gz`重命名为`cmake`，路径为`/usr/local/cmake`，配置对应的环境变量。

```shell
CMAKE_HOME=/usr/local/cmake
PATH=$PATH:$CMAKE_HOME/bin
export PATH
```

> 编译安装

```shell
./boostrap
make && make install
# 查看cmake版本
cmake -version
```

#### geos安装

```shell
#解压源码
tar jxf geos-3.8.0.tar.bz2
# 进入编译目录
cd geos-3.8.0/src
# 编译安装
make && make install
```

#### proj4安装

```shell
tar zxf proj-4.9.3.tar.gz 
cd proj-4.9.3/
./configure 
make && make install
```

#### json-c 安装

编译安装前，需要安装json-c的两个强依赖包

```shell
rpm -ivh  json-c-devel-0.11-4.el7_0.x86_64.rpm
rpm -ivh  json-c-0.11-4.el7_0.x86_64.rpm
```

若出现依赖问题，使用`--nodeps --force`强制安装

```shell
# 解压源码
unzip mirrors-json-c-master.zip 
cd json-c/
# 创建编译目录
mkdir build
cd build/
# 编译安装
cmake ../
make && make install
```

#### libxml2安装

下载libxml2对应的rpm包，强制安装

```
rpm -ivh libxml2-2.9.1-6.el7_2.3.x86_64.rpm
rpm -ivh libxml2-devel-2.9.1-6.el7_2.3.x86_64.rpm
rpm -ivh libxml2-static-2.9.1-6.el7_2.3.x86_64.rpm
```

#### `protobuf`和`protobuf-c`安装

```shell
tar -zxvf protobuf-2.6.1.tar.gz
cd protobuf-2.6.1
./configure
make && make install
```

安装完成后，修改 /etc/profile，配置protobuf的路径。

```shell
export PROTOBUF_HOME=/usr/local/protobuf
export PKG_CONFIG_PATH=/usr/local/protobuf/lib/pkgconfig
```

source /etc/profile 使配置生效。

> 安装`protobuf-c`

```shell
tar -zxvf protobuf-c-1.3.1.tar.gz
cd protobuf-c-1.3.1
./configure
make && make install
```

#### SFCGAL安装

安装`sfcgal`需要依赖`boost`、`cgal`、`mpfr`、`gmp`。

将`boost`、`mpfr`、`gmp`所需要的rpm包都放在同一个文件夹内

```shell
gmp-6.0.0-15.el7.x86_64
gmp-devel-6.0.0-15.el7.x86_64
mpfr-3.1.1-4.el7.x86_64
mpfr-devel-3.1.1-4.el7.x86_64
## boost 下的所有包
boost-1.53.0-27.el7.x86_64
boost-chrono-1.53.0-27.el7.x86_64
boost-devel-1.53.0-27.el7.x86_64
```

`rpm  -ivh  *.rpm --nodeps --force`安装该目录下的所有依赖。

>`cgal`编译安装

```shell
cd cgal-releases-CGAL-4.13/
mkdir build
cd build
cmake ../
make &&make install
```

>`sfcgal`安装

```shell
cd SFCGAL-1.3.6/
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/sfcgal  ../
make &&make install
```

需要指定`sfcgal`的安装路径

#### gdal安装

```shell
tar xf gdal-2.1.3.tar.xz
cd gdal-2.1.3/
./configure --prefix=/usr/local/gdal
make && make install
```

需要指定编译路径，方便postgis预编译查询依赖。

#### postgis安装

预编译

```shel
./configure --prefix=/usr/local/postgis \
 --with-gdalconfig=/usr/local/gdal/bin/gdal-config \
 --with-pgconfig=/usr/local/pgsql/bin/pg_config  \
 --with-protobufdir=/usr/local/protobuf-c \
 --with-sfcgal=/usr/local/sfcgal/bin/sfcgal-config
```

没有问题的话，`make && make install`