# 环境准备  
## 开发机器
Linux系统版本：  

    chrisyu@ubuntu:~$ lsb_release -a
    No LSB modules are available.
    Distributor ID:	Ubuntu
    Description:	Ubuntu 18.04.5 LTS
    Release:	18.04
    Codename:	bionic
## 源码下载
1. git clone https://github.com/postgres/postgres.git  
  
2. 源码下载完成之后，切换到分支REL_12_STABLE  
![切换分支](https://github.com/HeShuP/HeShuP.github.io/raw/gh-pages/_posts/images/postgresql/branch.png)

## 源码结构
    heshupei@ubuntu:~/pg/postgres$ tree -L 1
    .
    ├── aclocal.m4
    ├── config
    ├── configure
    ├── configure.in
    ├── contrib
    ├── COPYRIGHT
    ├── doc
    ├── GNUmakefile.in
    ├── HISTORY
    ├── Makefile
    ├── README
    ├── README.git
    └── src
    
    4 directories, 9 files

# 源码编译、安装

1. 环境检查，构建makefile文件  

        ./configure --with-pgport=5432  --prefix=/home/heshupei/pg/debug/  --without-readline  --enable-debug  

    参数解释：  
        --prefix：指定程序安装路径  
        --enable-debug：编译程序为可调试  

2. 编译  
   
        make -j4

3. 安装  

        make install 

4. 效果  
  
        heshupei@ubuntu:~/pg/debug$ tree -L 1
        .
        ├── bin
        ├── include
        ├── lib
        └── share
        
        4 directories, 0 files

# 运行PostgreSQL 

## 创建数据集簇  
    ./bin/initdb -D data -U system
![initdb](https://github.com/HeShuP/HeShuP.github.io/raw/gh-pages/_posts/images/postgresql/initdb.png) 

## 启动数据库
    ./bin/pg_ctl -D data/ start
![start](https://github.com/HeShuP/HeShuP.github.io/raw/gh-pages/_posts/images/postgresql/startdb.png) 

## 连接数据库
    ./bin/psql -p 5432 -U system -d postgres  

其中，-p参数为PostgreSQL的连接端口号；-U为连接用户名，-d是要连接的数据库名称。  
![psql](https://github.com/HeShuP/HeShuP.github.io/raw/gh-pages/_posts/images/postgresql/psql.png)   


至此，完成了PostgreSQL的源码下载、编译、和启动、连接，数据库已经为可以使用状态。