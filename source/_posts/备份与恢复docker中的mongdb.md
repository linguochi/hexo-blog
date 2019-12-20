---
title: docker中MongoDB的备份与恢复
author: 林国池
abbrlink: ea08259b
date: 2019-12-11 20:40:38
tags:
---

> 使用docker方式启动了MongoDB，虽然指定了宿主机中的volume数据卷，但是在做服务器迁移时，关于数据库的迁移还是用数据库本身的导入导出比较安全。

<!--more-->

### 备份

#### 查看MongoDB的容器名
```shell script
docker ps
# a2c8638c62eb mongo:3.4   "docker-entrypoint.s…"   4 days ago    Up 2 minutes   0.0.0.0:27017->27017/tcp   one-view-server_mongodb_1
```

#### 进入MongoDB容器,使用/bin/bash命令行
```shell script
docker exec -it one-view-server_mongodb_1 /bin/bash
# bash 有问题也可以用sh
# docker exec -it one-view-server_mongodb_1 sh 
```

#### 查看容器中的数据库
```shell script
> mongo  # 进入mongo命令行
>show dbs
#admin    0.000GB
#local    0.000GB
#oneview  0.000GB
```

#### 使用mongodump命令进行数据库备份
```shell script
# 在bash命令下面执行
mongodump -h 127.0.0.1 --port 27017 -u=用户名 -p=密码 -d test(数据库名字) -o /dump
# 容器里的MongoDB安装时未指定用户密码的
# mongodump -h 127.0.0.1 --port 27017 -d oneview -o /dump
```

#### 打包备份文件夹
```shell script
cd /dump/oneview
tar -zcvf oneviewdb.tar.gz
```

#### 将文件夹复制到宿主机
```shell script
exit # 退出到宿主机命令行
# docker cp <你的MongodDB容器名>:/dump/test.tar.gz /宿主机路径 
docker cp one-view-server_mongodb_1:/dump/oneviewdb.tar.gz ~/oneviewdb
```


### 恢复
```shell 
# 将文件拷贝到容器里面，解压；运行恢复命令
docker cp  ~/oneviewdb/oneviewdb.tar.gz one-view-server_mongodb_1:/
# 进入到容器
docker exec -it one-view-server_mongodb_1 /bin/bash
# 解压oneviewdb.tar.gz
tar -zxvf /dump/oneviewdb.tar.gz
# pwd 查看 oneviewdb文件夹路径
# 恢复 --drop清空原有数据
mongorestore -h 127.0.0.1 --port 27017 --drop -d oneview /dump/dump/oneview
```
