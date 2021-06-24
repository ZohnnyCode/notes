---
title: 15000、docker学习
date: 2021-05-25 12:56:10
tags:
---

centos7.0++

#### 配置下载镜像的加速器

```js
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://li0q1630.mirror.aliyuncs.com"]
}
systemctl daemon-reload
systemctl restart docker
```

#### 命令

```js
pwd 查看当前处于那个目录
```

```js
docker version
docker info
docker --help

docker images 本地主机上的镜像
        -a // 列出所有镜像
        -q // 只列出镜像的id
        -qa // 列出全部镜像的id
        --digests
docker pull 名字
docker search 名字 // hub.docker.com
        -s 30 tomcat // 点赞数超过30

docker rmi -f id // 强制删除镜像---单个
docker rmi -f 名字:标签 名字:标签 // 强制删除镜像---多个
docker rmi -f $(docker images -qa) // 删除全部

镜像就像千层饼，一层一层的
```

```js
运行镜像获取容器
docker run [options] image [command][args...]
options参数：-it // 启动新终端
             -d // 守护式启动容器，没有前台后台也一直挂着
            --name= // 容器新名字


查看正在运行的容器
docker ps // 查看docker里面正在运行的进程有哪些（有哪些容器）
docker ps -l // 上一次运行的容器
docker ps -a // 所有容器
docker ps -n 3 // 前三次运行的容器

退出容器：exit：关闭容器并退出
        ctrl+p+q

启动容器
docker start 容器id或容器名

重启容器
docker restart 容器id或容器名

停止容器
docker stop 容器id或名字

强制停止容器
docker kill id或名字

删除已停止的容器
docker rm 容器id
rmi是删除镜像

删除多个容器
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm

查看某个容器的日志
docker logs -t(时间) -f（一直打印） --tail 数字（要几条） 容器id
```

```js
docker run -it 镜像 /bin/bash
ctrl+p+q
重新进去容器方式1：
docker attach 容器id
ls -l // 查看根目录下面内容
ls -l /tmp // 容器内tmp目录有啥

重新进去容器方式2：隔山打牛
docker exec -it 容器id ls -l /tmp  // 进去容器查看容器目录tmp，终端最后还是在宿主机
docker exec -it 容器id /bin/bash // 跟attach效果一样
```

```js
容器文件拷贝到宿主机上面
docker cp 容器id:/tmp/yum.log /root(目标路径)
```

#### linux 基本操作命令

```js
pwd
rm -rf 文件夹名

解压tar文件
tar -xvf xxx.tar

加压rar文件
tar -zxvf 文件名
```

#### 宿主机和镜像容器通信

```js
docker run -it -v /宿主机绝对目录:/容器目录 镜像名 // 可读写

docker run -it -v /宿主机绝对目录:/容器目录:ro 镜像名 // 只读

docker ps // 查看正在运行的容器
docker inspect 容器id  // 查看卷的关系
touch host.txt // 进入某个目录下新建一个文件
vi host.txt // 编辑这个文件
cat host.txt // 查看一个文件的内容

查看正在运行的容器：docker ps
查看上一个：docker l
重启容器：docker start 容器id
重新进入一个容器：docker attach 容器id
```

```js
运行启动程序代码
  docker run -it -v $(pwd)/myservices:/bot docker.io/wechaty/wechaty app.js
```
