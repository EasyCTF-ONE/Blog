+++
date = '2025-01-07T21:01:51+08:00'
draft = false
title = 'docker'

+++

## 一 安装docker

阅读官方文档进行安装https://docs.docker.com/engine/install/ubuntu/

设置 Docker 的`apt`存储库

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```



配置镜像加速

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://rd4ovv3y.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



==安装完成后使用docker version验证安装是否成功==

## 二 docker的使用

### 1.下载镜像

> 检索：docker search
>
> 下载：docker pull
>
> 列表：docker images   <==>   docker image ls
>
> 删除：docker rmi      <==>   docker rm image
>
> 查看卷：docker volume ls
>
> 镜像名:标签（版本）



### 2.启动容器

> ==运行：docker run== 
> 查看：docker ps（运行中的容器）       docker ps -a 查看所有容器
> 停止：docker stop
> 启动：docker start
> 重启：docker restart
> 状态：docker stats
> 日志：docker logs
> ==进入：docker exec==
> 删除：docker rm

#### 2.1 run命令细节

```
docker run -d -p 88:80 --name mynginx nginx  
后台启动端口映射（主机端口:容器端口）并起别名mynginx
目录挂载：-v 主机目录:容器目录
docker run -d -p 88:80 -v /app/nghtml:/usr/share/nginx/html --name mynginx nginx
卷映射：-v ngconf:/etc/nginx
```

==卷的默认路径：/var/lib/docker/volumes/\<volume-name\>==

### 3.保存镜像

```
提交：docker commit -m "update" mynginx mynginx:v.10
保存：docker save -o mynginx.tar myginx:v.10
加载：docker load -i mynginx.tar
```

### 4.分享社区

```
登录：docker login
命名：docker tag	原来镜像名 用户名/改镜像名
推送：docker push
```

### 5.自定义网络

docker为每个容器都分配唯一ip，使用容器ip＋容器端口可以互相访问

ip由于各种原因可能会变化

docker0默认不支持主机域名

创建自定义网络，容器名就是域名

>docker network create 名字
>
>docker run --network 名字

## 三 docker compose

```
上线：docker compose up -d
下线：docker compose down

启动：docker compose start x1 x2 x2
停止：docker compose stop x1 x2 x3
扩容：docker compose scale x2=3
```

