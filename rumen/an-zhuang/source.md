# 目录
> * [依赖环境](#chapter-1)
> * [Tars C++开发环境](#chapter-2)
> * [Tars框架安装](#chapter-3)
> * [Tars-web说明](#chapter-4)

本安装文档描述在一台或多台服务器上安装搭建整个Tars框架的过程，目的是为了让用户对Tars框架的部署搭建、运行、测试等有个整体的认识。

如要用于线上环境，部署安装的原理是一样，不过需要更多考虑分布式系统下服务的部署需要有容错、容灾等的能力。

# 1. <a id="chapter-1"></a>依赖环境

软件 |软件要求
------|--------
linux内核版本:      |	2.6.18及以上版本（操作系统依赖）
gcc版本:          	|   4.8.2及以上版本、glibc-devel（c++语言框架依赖）
bison工具版本:      |	2.5及以上版本（c++语言框架依赖）
flex工具版本:       |	2.5及以上版本（c++语言框架依赖）
cmake版本：       	|   2.8.8及以上版本（c++语言框架依赖）
mysql版本:          |   4.1.17及以上版本（框架运行依赖）
rapidjson版本:      |   1.0.2版本（c++语言框架依赖）
nvm版本：           |   0.35.1及以上版本（web管理系统依赖, 脚本安装过程中自动安装）
node版本：          |   12.13.0及以上版本（web管理系统依赖, 脚本安装过程中自动安装）

运行服务器要求：1台普通安装linux系统的机器即可。

## 1.1. 编译包依赖下载安装介绍

源码编译过程需要安装:gcc, glibc, bison, flex, cmake

例如，在Centos7下，执行：
```
yum install glibc-devel gcc gcc-c++ bison flex cmake which
```

## 1.2. Mysql依赖库安装

Tars代码编译需要依赖mysql头文件和静态库, 依赖路径如下:

- 头文件: /usr/local/mysql/include
- 库路径: /usr/local/mysql/lib

如果系统已经存在mysql头文件和静态库, 则可跳过次步骤, 否则建议: 

```
rpm -ivh https://repo.mysql.com/mysql57-community-release-el7.rpm
yum install -y mysql-devel 
mkdir -p /usr/local/mysql && ln -s /usr/lib64/mysql /usr/local/mysql/lib && ln -s /usr/include/mysql /usr/local/mysql/include && echo "/usr/local/mysql/lib/" >> /etc/ld.so.conf && ldconfig 
```

## 1.3. Mysql客户端安装

Tars环境部署需要依赖mysql客户端

```
which mysql
```

如果mysql客户端不存在, 请执行:
```
rpm -ivh https://repo.mysql.com/mysql57-community-release-el7.rpm
yum install -y mysql 
```

## 1.4. Mysql安装

Tars框架安装需要在mysql中读写数据, 因此需要安装mysql, 如果你已经存在mysql, 可以忽略该步骤.

安装mysql请参考[mysql安装](mysql.md)


# 2. <a id="chapter-2"></a>Tars C++开发环境(源码安装框架必备)

**源码安装框架才需要做这一步, 如果只是用c++写服务, 只需要下载tarscpp代码即可**

下载TarsFramework源码

```
cd ${source_folder}
git clone https://github.com/TarsCloud/TarsFramework.git --recursive
```

然后进入build源码目录
```
cd ${source_folder}/TarsFramework/build
chmod u+x build.sh
./build.sh prepare
./build.sh all
```

**编译时默认使用的mysql开发库路径：include的路径为/usr/local/mysql/include，lib的路径为/usr/local/mysql/lib/**

若mysql开发库的安装路径不在默认路径需要修改CMakeLists文件中mysql开发库的路径。CMakeLists在`${source_folder}/TarsFramework/`和`${source_folder}/TarsFramework/tarscpp/` 目录下各有一个同名文件。
修改文件中上述路径为本机mysql开发库的路径
(参考路径："/usr/include/mysql"；"/usr/lib64/mysql")。


如果需要重新编译
```
./build.sh cleanall
./build.sh all
```

切换至root用户，创建安装目录
```
cd /usr/local
mkdir tars
chown ${普通用户}:${普通用户} ./tars/
```

安装
```
cd ${source_folder}/build
./build.sh install或者make install
```
**默认的安装路径为/usr/local/tars/cpp**

**如要修改安装路径:**
```
**需要修改tarscpp目录下CMakeLists.txt文件中的安装路径。**
**需要修改tarscpp/servant/makefile/makefile.tars文件中的TARS_PATH的路径**
**需要修改tarscpp/servant/script/create_tars_server.sh文件中的DEMO_PATH的路径**
```

# 3 <a id="chapter-3"></a>Tars框架安装

## 3.1. 框架安装模式

**框架有两种模式:**

- centos or ubuntu一键部署, 安装过程中需要网络从外部下载资源
- 制作成docker镜像来完成安装, 制作docker过程需要网络下载资源, 但是启动docker镜像不需要外网

**框架安装注意事项:**

- 安装过程中, 由于tars-web依赖nodejs, 所以会自动下载nodejs, npm, pm2以及相关的依赖, 并设置好环境变量, 保证nodejs生效.
- nodejs的版本目前默认下载的v12.13.0
- 如果你本机以及安装了nodejs, 最好卸载掉
- 如果使用腾讯云服务器请修改 linux-install.sh 中 MIRROR变量为 http://mirrors.tencentyun.com 否则安装node与pm2的时候出现失败的情况
**注意:需要完成TarsFramework的编译和安装**

下载tarsweb并copy到/usr/local/tars/cpp/deploy目录下(注意目录名是web, 不要搞错!):

```
git clone https://github.com/TarsCloud/TarsWeb.git
mv TarsWeb web
cp -rf web /usr/local/tars/cpp/deploy/
```

例如, 这是/usr/local/tars/cpp/deploy下的文件:
```
[root@vm-0-15-centos deploy]# ls -l
total 64
-rw-r--r--  1 root root  1922 Jan 10 21:44 centos7_base.repo
-rw-r--r--  1 root root  1229 Jan 10 21:44 Dockerfile
-rwxr-x---  1 root root  2959 Jan  8 21:46 docker-init.sh
-rwxr-x---  1 root root   215 Dec 31 15:37 docker.sh
drwxr-xr-x  4 root root  4096 Jan 10 21:41 framework
-rwxr-x---  1 root root  4876 Jan 10 21:38 linux-install.sh
-rw-r--r--  1 root root   565 Dec 31 15:37 README.md
-rw-r--r--  1 root root   539 Dec 31 15:37 README.zh.md
-rwxr-x---  1 root root  1157 Jan  7 16:38 tar-server.sh
-rwxr-x---  1 root root 12162 Jan 10 15:36 tars-install.sh
-rwxr-x---  1 root root   311 Dec 31 15:37 tars-stop.sh
drwxr-xr-x  2 root root  4096 Jan 10 21:41 tools
drwxr-xr-x 12 root root  4096 Jan  5 12:03 web
```

## 3.2. 框架部署说明

框架可以部署在单机或者多机上, 多机是一主多从模式, 通常一主一从足够了:

- 主节点只能有一台, 从节点可以多台
- 主节点默认会安装:tarsAdminRegistry, tarspatch, tarsweb, tarslog, 这几个服务在从节点上不会安装
- tarsAdminRegistry只能是单点(带有发布状态)
- tarslog也只能是单点, 否则日志会分散在多机上
- 原则上tarspatch, tarsweb可以是多点, 如果部署成多点, 需要把/usr/local/app/patchs目录做成多机间共享(可以通过NFS), 否则无法正常发布服务
- 可以后续把tarslog部署到大硬盘服务器上
- 实际使用中, 即使主从节点都挂了, 也不会影响框架上服务的正常运行, 只会影响发布
- 一键部署会自动安装好web(自动下载nodejs, npm, pm2等相关依赖), 同时开启web权限

部署完成后会创建5个数据库，分别是db_tars、db_tars_web、db_user_system、 tars_stat、tars_property。 

其中db_tars是框架运行依赖的核心数据库，里面包括了服务部署信息、服务模版信息、服务配置信息等等；

db_tars_web是web管理平台用到数据库

db_user_system是web管理平台用到的权限管理数据库

tars_stat是服务监控数据存储的数据库；

tars_property是服务属性监控数据存储的数据库；

无论哪种安装方式, 如果成功安装, 都会看到类似如下输出:

```
 2019-10-31 11:06:13 INSTALL TARS SUCC: http://xxx.xxx.xxx.xxx:3000/ to open the tars web. 
 2019-10-31 11:06:13 If in Docker, please check you host ip and port. 
 2019-10-31 11:06:13 You can start tars web manual: cd /usr/local/app/web; npm run prd 
```
打开你的浏览器输入: http://xxx.xxx.xxx.xxx:3000/ 如果顺利, 可以看到web管理平台

**注意: 执行完毕以后, 可以检查nodejs环境变量是否生效: node --version, 如果输出不是v12.13.0, 则表示nodejs环境变量没生效**
**如果没生效, 手动执行:  centos: source ~/.bashrc or ubuntu: source ~/.profile**

请参考[检查web的问题](web.md)中的检查web问题章节, 如果没有问题, 请检查机器防火墙
 
## 3.3. (centos or ubuntu)一键部署

进入/usr/local/tars/cpp/deploy, 执行:
```
chmod a+x linux-install.sh
./linux-install.sh MYSQL_HOST MYSQL_ROOT_PASSWORD INET REBUILD(false[default]/true) SLAVE(false[default]/true) MYSQL_USER MYSQL_PORT
```

MYSQL_HOST: mysql数据库的ip地址

MYSQL_ROOT_PASSWORD: mysql数据库的root密码(注意root不要有太特殊的字符, 例如!, 否则shell脚本识别有问题, 因为是特殊字符)

INET: 网卡的名称(ifconfig可以看到, 比如eth0), 表示框架绑定的本机IP, 注意不能是127.0.0.1

REBUILD: 是否重建数据库,通常为false, 如果中间装出错, 希望重置数据库, 可以设置为true

SLAVE: 是否是从节点

MYSQL_USER: mysql用户, 默认是root

MYSQL_PORT: mysql端口

举例, 安装两台节点, 一台数据库(假设: 主[192.168.7.151], 从[192.168.7.152], mysql:[192.168.7.153])

主节点上执行(192.168.7.151)
```
chmod a+x linux-install.sh
./linux-install.sh 192.168.7.153 tars2015 eth0 false false root 3306
```
主节点执行完毕后, 从节点执行:
```
chmod a+x linux-install.sh
./linux-install.sh 192.168.7.153 tars2015 eth0 false true root 3306
```

执行过程中的错误参见屏幕输出, 如果出错可以重复执行(一般是下载资源出错)

## 3.4. 制作成docker

目标: 将框架制作成一个docker, 部署时启动docker即可.

进入该目录, 执行生成docker:
```
chmod a+x docker.sh
./docker.sh v1
```
docker制作完毕: tar-docker:v1
```
docker ps
```

可以将docker发布到你的机器, 然后执行

```
docker run -d --net=host -e MYSQL_HOST=xxxxx -e MYSQL_ROOT_PASSWORD=xxxxx \
        -e MYSQL_USER=root -e MYSQL_PORT=3306 \
        -eREBUILD=false -eINET=enp3s0 -eSLAVE=false \
        -v/data/tars:/data/tars \
        -v/etc/localtime:/etc/localtime \
        tars-docker:v1
```

MYSQL_IP: mysql数据库的ip地址

MYSQL_ROOT_PASSWORD: mysql数据库的root密码

INET: 网卡的名称(ifconfig可以看到, 比如eth0), 表示框架绑定本机IP, 注意不能是127.0.0.1

REBUILD: 是否重建数据库,通常为false, 如果中间装出错, 希望重置数据库, 可以设置为true

SLAVE: 是否是从节点

MYSQL_USER: mysql用户, 默认是root

MYSQL_PORT: mysql端口

映射三个目录到宿主机
- -v/data/tars:/data/tars
  >- 包含了 tars应用日志,  tarsnode/data目录(业务服务的运行包, 保证docker重启, 发布到docker内部的服务不会丢失)
  >- 如果是主机则还包含: web日志, 发布包目录

**如果希望多节点部署, 则在不同机器上执行docker run ...即可, 注意参数设置!**

**这里必须使用 --net=host, 表示docker和宿主机在相同网络** 

## 3.5. mysql权限问题

上述安装mysql默认都是需要root权限, 但是在某些场景不具备root用户或者root用户必须交互式输入密码的情况下(比如腾讯云cdb), 你可以这样安装:

- 首先在mysql中创建用户(可能管理员分配给你的), 比如:admin
- admin用户具备以下权限(重点是创建用户的权限):
```
SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE
```
- 执行安装脚本

```
./linux-install.sh 192.168.7.153 tars2015 eth0 false true admin 3306

```

```
docker run -d --net=host -e MYSQL_HOST=xxxxx -e MYSQL_ROOT_PASSWORD=xxxxx \
        -eMYSQL_USER=admin -eMYSQL_PORT=3306 \
        -eREBUILD=false -eINET=enp3s0 -eSLAVE=false \
        -v/data/tars:/data/tars \
        -v/etc/localtime:/etc/localtime \
        tars-docker:v
```

## 3.6. 核心模块

无论是那种安装方式, 最终Tars Framework都是由几个核心模块组成的, 例如:

```
[root@VM-0-7-centos deploy]# ps -ef | grep app/tars | grep -v grep
root       368     1  0 09:20 pts/0    00:00:25 /usr/local/app/tars/tarsregistry/bin/tarsregistry --config=/usr/local/app/tars/tarsregistry/conf/tars.tarsregistry.config.conf
root      9245 32687  0 09:29 ?        00:00:13 /usr/local/app/tars/tarsstat/bin/tarsstat --config=/usr/local/app/tars/tarsnode/data/tars.tarsstat/conf/tars.tarsstat.config.conf
root     32585     1  0 09:20 pts/0    00:00:10 /usr/local/app/tars/tarsAdminRegistry/bin/tarsAdminRegistry --config=/usr/local/app/tars/tarsAdminRegistry/conf/tars.tarsAdminRegistry.config.conf
root     32588     1  0 09:20 pts/0    00:00:20 /usr/local/app/tars/tarslog/bin/tarslog --config=/usr/local/app/tars/tarslog/conf/tars.tarslog.config.conf
root     32630     1  0 09:20 pts/0    00:00:07 /usr/local/app/tars/tarspatch/bin/tarspatch --config=/usr/local/app/tars/tarspatch/conf/tars.tarspatch.config.conf
root     32653     1  0 09:20 pts/0    00:00:14 /usr/local/app/tars/tarsconfig/bin/tarsconfig --config=/usr/local/app/tars/tarsconfig/conf/tars.tarsconfig.config.conf
root     32687     1  0 09:20 ?        00:00:22 /usr/local/app/tars/tarsnode/bin/tarsnode --locator=tars.tarsregistry.QueryObj@tcp -h 172.16.0.7 -p 17890 --config=/usr/local/app/tars/tarsnode/conf/tars.tarsnode.config.conf
root     32695     1  0 09:20 pts/0    00:00:09 /usr/local/app/tars/tarsnotify/bin/tarsnotify --config=/usr/local/app/tars/tarsnotify/conf/tars.tarsnotify.config.conf
root     32698     1  0 09:20 pts/0    00:00:14 /usr/local/app/tars/tarsproperty/bin/tarsproperty --config=/usr/local/app/tars/tarsproperty/conf/tars.tarsproperty.config.conf
root     32709     1  0 09:20 pts/0    00:00:12 /usr/local/app/tars/tarsqueryproperty/bin/tarsqueryproperty --config=/usr/local/app/tars/tarsqueryproperty/conf/tars.tarsqueryproperty.config.conf
root     32718     1  0 09:20 pts/0    00:00:12 /usr/local/app/tars/tarsquerystat/bin/tarsquerystat --config=/usr/local/app/tars/tarsquerystat/conf/tars.tarsquerystat.config.conf
```

- 对于主机节点 tarsAdminRegistry  tarsnode  tarsregistry tars-web 必须活着, 其他tars服务会被tarsnode自动拉起
- 对于从机节点 tarsnode  tarsregistry 必须活着, 其他tars服务会被tarsnode拉起
- tars-web是nodejs实现的服务, 由两个服务组成, 具体参见后面章节
- 为了保证核心服务是启动的, 可以通过check.sh来控制, 在crontab 中配置

主机(add to contab):

```
* * * * * /usr/local/app/tars/check.sh master
```

从机(add to contab):
```
* * * * * /usr/local/app/tars/check.sh 
```

如果配置了check.sh, 就不需要配置后面章节中的tarsnode的监控了

# 4. <a id="chapter-4"></a>Tars-web说明

## 4.1 模块说明

Tars Framework部署好以后, 在主机节点上会安装tars-web(从机节点不会安装), tars-web采用nodejs实现, 由两个服务组成.

查看tars-web的模块:
```
pm2 list
```

输出如下:
```
[root@8a17fab70409 data]# pm2 list
┌────┬─────────────────────────┬─────────┬─────────┬──────────┬────────┬──────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ id │ name                    │ version │ mode    │ pid      │ uptime │ ↺    │ status   │ cpu      │ mem      │ user     │ watching │
├────┼─────────────────────────┼─────────┼─────────┼──────────┼────────┼──────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ 0  │ tars-node-web           │ 0.2.0   │ fork    │ 1602     │ 2m     │ 0    │ online   │ 0.1%     │ 65.1mb   │ root     │ disabled │
│ 1  │ tars-user-system        │ 0.1.0   │ fork    │ 1641     │ 2m     │ 0    │ online   │ 0.1%     │ 60.1mb   │ root     │ disabled │
└────┴─────────────────────────┴─────────┴─────────┴──────────┴────────┴──────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

**如果找不到pm2, 一般是环境变量没生效, 请先执行: centos: source ~/.bashrc or ubuntu: source ~/.profile, 安装过程中会写这个文件**

**tars-web由两个模块组成**
- tars-node-web: tars-web主页面服务, 默认绑定3000端口, 源码对应web目录
- tars-user-system: 权限管理服务, 负责管理所有相关的权限, 默认绑定3001端口, 源码对应web/demo目录

tars-node-web调用tars-user-system来完成相关的权限验证

**web采用nodejs+vue来实现, 最终的安装运行目录如下:**

```
/usr/local/app/web
```

如果pm2 list中查看模块启动不了, 可以进入改目录定位问题:

```
cd /usr/local/app/web/demo; npm run start
cd /usr/local/app/web; npm run start
```

npm run start 启动服务, 可以观察控制台的输出, 如果有问题, 会有提示.

**正式运行建议: pm2 start tars-node-web; pm2 start tars-user-system**

如果安装完成后web页面打不开, 请参考[web](web.md), 检查问题章节, 定位问题


