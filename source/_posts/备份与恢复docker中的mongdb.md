---
title: docker中MongoDB的备份与恢复
---

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
cd /
tar -zcvf oneviewdb.tar.gz /dump/oneview
```


#### 将文件夹复制到宿主机
```shell script
exit # 退出到宿主机命令行
# docker cp <你的MongodDB容器名>:/dump/test.tar.gz /宿主机路径 
docker cp one-view-server_mongodb_1:/dump/oneviewdb.tar.gz ~/oneviewdb
```
