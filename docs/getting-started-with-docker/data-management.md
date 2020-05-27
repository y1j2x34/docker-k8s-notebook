# Docker 数据管理

## 容器的数据卷

### 什么是数据卷？

Docker 数据卷是经过特殊设计的目录，可以绕过联合文件系统（UFS），为一个或多个容器提供访问。
数据卷设计的目的在于数据的永久化，它完全独立与容器的生命周期，所以，容器删除时，容器挂载的数据卷不会被删除，同时也不存在垃圾收集机制处理容器引用的数据。

### 数据卷的特点

- docker 数据卷独立于 docker 存在，与 docker 容器的生命周期分离；
- 存在于宿主机（docker host）中；
- docker数据卷，可以是目录也可以是文件；
- docker容器可以利用数据卷技术与宿主机/其它容器进行数据共享和重用。
- 可以直接修改数据卷里的内容
- 数据卷的变化不会影响镜像的更新
- 即使挂载数据卷的容器被删除，卷也会一直存在

### 数据卷的使用方法

```sh
# 为容器添加数据卷 -- 使用-v选项
docker run -v ~/container_data:/data -it <IMAGE> /bin/bash
```

### 数据卷添加访问权限

默认挂载的数据卷为读写权限。也可以根据需求，设置为只读。在宿主机上操作如下：

```sh
# ro: 指定为只读
docker run -d -v ~/container_data:/data:ro centos /bin/bash
```

### 用 Dockerfile 创建包含数据卷的镜像

```sh
# Dockerfile指令：
VOLUME ["/data"]
```

在 Dockerfile 中，VOLUME 指令创建的挂载点，无法指定主机上对应的目录，是自动生成的

#### Dockerfile

```sh
# Version 0.0.1
FROM centos:latest
VOLUME ["/data/volume1","/data/volume/2"]
RUN yum -y install nginx
RUN echo 'hello world' > /usr/share/nginx/html/index.html
EXPOSE 80
```

构建镜像

```sh
docker build -t="mynginx2"
```

启动容器

```sh
docker run -it --name mynginx-test mynginx2
```

查看数据卷

```sh
[root@b9d0bcce8330 /]# ls
bin  data  dev	etc  home  lib	lib64  lost+found  media  mnt  opt  proc  root	run  sbin  srv	sys  tmp  usr  var
[root@b9d0bcce8330 /]# cd data
[root@b9d0bcce8330 data]# ls
volume1 volume2
[root@b9d0bcce8330 data]#
```

查看主机挂载点

```sh
docker inspect
```

## 数据卷容器

