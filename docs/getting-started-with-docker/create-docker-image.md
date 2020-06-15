# 构建镜像

## 手动构建

1. 创建一个容器

    ```sh
    ~ >>> docker create --name nginx-base -p 80:80 nginx
    877171d5bdd46aee1d20365081706914f6689ec60b16163162c08bd32827de57
    ~ >>>
    ```

1. 查看容器

    ```sh
    ~ >>> docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
    877171d5bdd4        nginx               "/docker-entrypoint.…"   20 seconds ago      Created                                        nginx-base
    ```

1. 启动容器

    ```sh
    ~ >>> docker start nginx-base
    nginx-base
    ~ >>>
    ```

    这时浏览器访问 <http://127.0.0.1/> 就可以看到nginx的 "Welcome to nginx" 页。

1. 修改容器

    编写一个 index.html 文件并复制到容器中

    ```html
    <html>
        <head>
            <title>Hello World</title>
        </head>
        <body>
            <h1>Hello World!</h1>
        </body>
    </html>
    ```

    运行命令：

    ```sh
    ~ >>> docker cp index.html nginx_base:/use/share/nginx/html/index.html
    ```

    运行后访问 <http://127.0.0.1/> 地址就变成上面的 Hello world 页面。

1. 从容器创建镜像

    ```sh
    ~/nginx-data >>> docker commit nginx-base
    sha256:d6c2dc3823af2a7b4cc3b3b29ec79c25d68735e11ca12a630b142d50f7572266
    ~/nginx-data >>>  
    ```

    查看创建出来的镜像：

    ```sh
    ~/nginx-data >>> docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    <none>              <none>              d6c2dc3823af        48 seconds ago      132MB
    nginx               latest              4392e5dad77d        29 hours ago        132MB
    centos              latest              470671670cac        4 months ago        237MB
    ~/nginx-data >>>
    ```

    可以看到新创建出来的镜像，但是 repository 和 TAG 都是 <none>, 可以用 tag 命令来标记，以便于查找：

    ```sh
    ~/nginx-data >>> docker tag d6c2dc3823af hello-workd-nginx
    ~/nginx-data >>>
    ```

    再查看镜像列表就变成下面这样：

    ```sh
    ~/nginx-data >>> docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    hello-workd-nginx   latest              d6c2dc3823af        3 minutes ago       132MB
    nginx               latest              4392e5dad77d        29 hours ago        132MB
    centos              latest              470671670cac        4 months ago        237MB
    ~/nginx-data >>>
    ```

1. 删除原来的镜像

    使用 `docker ps -l` 命令可以看到刚刚用来创建镜像的容器还在运行中：

    ```sh
    ~/nginx-data >>> docker ps -l
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
    877171d5bdd4        nginx               "/docker-entrypoint.…"   17 minutes ago      Up 16 minutes       0.0.0.0:80->80/tcp   nginx-base
    ~/nginx-data >>>
    ```

    停止并删除容器：

    ```sh
    ~/nginx-data >>> docker stop 877171d5bdd4
    877171d5bdd4
    ~/nginx-data >>> docker rm nginx-base
    nginx-base
    ~/nginx-data >>>
    ```

1. 创建时设定作者身份

    inspect命令查看作者身份：

    ```sh
    ~ >>> docker inspect d6c2dc3823af | grep Author
        "Author": "",
    ```

    可以发现作者是空的， 现在可以这样设置：

    ```sh
    ~ >>> docker commit --author jianxin_yang nginx-base authored-nginx-base
    ```

1. 创建时指定描述信息

    ```sh
    docker commit --message 'This is a basic nginx image' nginx-base mmm
    ```

    通过 history 命令查看描述信息：

    ```sh
     >>> docker commit --message 'this is a basic nginx image' nginx-base mmm
    sha256:25565b041b1818d2b917b5ecb3fb7a70272d401a4d1cec10eae5f51dd650cbd5
    ~ >>> docker history mmm
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    25565b041b18        7 seconds ago       nginx -g daemon off;                            1.12kB              this is a basic nginx image
    4392e5dad77d        4 days ago          /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
    <missing>           4 days ago          /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B
    <missing>           4 days ago          /bin/sh -c #(nop)  EXPOSE 80                    0B
    <missing>           4 days ago          /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
    <missing>           4 days ago          /bin/sh -c #(nop) COPY file:cc7d4f1d03426ebd…   1.04kB
    <missing>           4 days ago          /bin/sh -c #(nop) COPY file:b96f664d94ca7bbe…   1.96kB
    <missing>           4 days ago          /bin/sh -c #(nop) COPY file:d68fadb480cbc781…   1.09kB
    <missing>           4 days ago          /bin/sh -c set -x     && addgroup --system -…   62.9MB
    <missing>           4 days ago          /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B
    <missing>           4 days ago          /bin/sh -c #(nop)  ENV NJS_VERSION=0.4.1        0B
    <missing>           4 days ago          /bin/sh -c #(nop)  ENV NGINX_VERSION=1.19.0     0B
    <missing>           3 weeks ago         /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
    <missing>           3 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B
    <missing>           3 weeks ago         /bin/sh -c #(nop) ADD file:7780c81c33e6cc5b6…   69.2MB
    ```

    commit 命令有个副作用就是会**暂停**运行中的容器，如果不希望暂停，在commit命令后面加上 `--pause=false` 参数

1. 修改配置
    docker commit 支持一个 `-c/--change` 标识，它支持的设置有以下这些：

    - CMD
    - ENTRYPOINT
    - ENV
    - EXPOSE
    - LABEL
    - ONBUILD
    - USER
    - VOLUME
    - WORKDIR

    以 Nginx 镜像的 dockerfile 内容为例：

    ```yaml
    CMD [“nginx”, “-g”, “daemon off;”]
    ENV NGINX_VERSION 1.15.3
    EXPOSE 80
    ```

    这样修改 CMD 配置：

    ```sh
    docker commit --change='CMD ["nginx", "-T"]' nginx-base conf_dump
    ```

    启动新制作的镜像：

    ```sh
    docker run --name dumper -p 80:80 conf_dump
    ```

## 使用 Dockerfile 构建

### 常用指令

|类型|命令|
|--|--|
|基础镜像|FROM|
|维护者信息|MAINTAINER|
|镜像操作指令|RUN,COPY,ADD,EXPOSE,WORKDIR,ONBUILD,USER,VOLUMN...|
|容器启动时执行指令|CMD,ENTRYPOINT|

1. FROM: 指定基础镜像

    构建镜像可以理解为基于一个基础镜像进行修改。因此一个Dockerfile中FROM是必须要配置的指令，并且必须是第一条指令。如：指定ubuntu的18版本为基础镜像

    ```dockerfile
    FROM ubunt:18
    ```

1. RUN: 执行命令

    RUN指令在新镜像内部执行的命令，如：执行某些动作、安装系统软件、配置系统信息等等，格式有如下两种：

    - shell格式：`RUN < command >`, 就像直接在命令行中输入命令一样，如

        ```dockerfile
        RUN echo "hello" > /etc/nginx/html/index.html
        ```

    - exec格式：`RUN ["可执行文件","参数1","参数2"]`，如在镜像中用yum安装nginx:

        ```dockerfile
        RUN ["yum", "install", "nginx"]
        ```

    **注意：** 多行命令不要写多个 `RUN`，因为dockerfile中每一条指令都会建立一层镜像层，容易造成镜像臃肿、多层，增加构建部署时间，也容易出错， `RUN` 书写的换行符是 `\`.

1. COPY 复制文件

    将宿主机器上的文件复制到镜像内，如果目的位置不存在，Docker会自动创建。但宿主机要复制的目录必须和Dockerfile文件同级目录下。
    格式：

    ```dockerfile
    COPY [--chown=<user>:<group>] <愿路径>...<目标路径>
    COPY [--chown=<user>:<group>] ["<愿路径>",..."<目标路径>"]
    ```

1. CMD: 容器启动时执行的命令

    CMD在Dockerfile中只能出现一次，多次的话，只有最后一个有效。它的作用就是在启动容器时提供一个默认命令项。如果用户执行`docker run`时提供了命令项，就会覆盖这个命令。

    格式：

    shell 格式：

    ```dockerfile
    CMD <命令>
    ```

    exec 格式：

    ```dockerfile
    CMD ["可执行文件", "参数1", "参数2"...]
    ```

1. MAINTAINER: 指定作者

    ```dockerfile
    MAINTAINER <name> <email>
    ```

1. EXPOSE: 暴露端口

    设置容器对外映射的端口号，相当于指定 `docker run -p` 的-p参数

    ```dockerfile
    EXPOSE <端口1> [<端口2>...]
    ```

1. WORKDIR: 配置工作目录

    为RUN、CMD、ENTRYPOINT指令配置工作目录，类似于 cd 命令，但是WORKDIR 会自动创建不存在的目录

    ```dockerfile
    WORKDIR <path>
    ```

1. ENTRYPOINT: 容器启动执行命令

    用法和 CMD 一样，但有不同点：
    1. CMD的命令会被docker run的命令覆盖，而ENTRYPOINT不会
    2. CMD和ENTRYPOINT都存在时，CMD指令变成了ENRTYPOINT的参数，并且此CMD提供的参数会被docker run后面的命令覆盖

1. VOLUME

    创建一个从主机或其它容器挂载的挂载点

    ```dockerfile
    VOLUME ["path"]
    ```

1. USER

    指定当前往下执行的用户

1. ADD

    同COPY

1. ONBUILD

    配置当前所创建的镜像作为其它新创建镜像的基础镜像时所执行的操作指令。

    ```dockerfile
    ONBUILD [INSTRUCTION]
    ```

1. ENV

    设置环境变量，变量以 `key=value` 的形式存在，在容器内被脚本或者程序调用，容器运行的时候这个变量也会保留

    ```dockerfile
    # 一个
    ENV <key> <value>
    # 多个
    ENV <key>=<value> <key2>=<value2>
    ```

    注意：

    1. 具有传染性，当前镜像被当作其它镜像的基础镜像时，新镜像会拥有这个基础镜像所有环境变量
    2. ENV定义的环境变量可以在dockerfile被后面的所有指令中使用，但不能被docker run的命令参数引用。

### Dockerfile 编写

基于centos构建一个nginx镜像

```dockerfile
FROM centos
MAINTAINER vgerbot vgerbot@gmail.com
# 测试网络环境
RUN ping -c 1 www.baidu.com
RUN yum -y install gcc make pcre-devel zlib-devel tar zlib
ADD nginx-1.15.8.tar.zg /usr/src/
RUN cd /usr/src/nginx-15.8 \
    && mkdir /usr/local/nginx \
    && ./configure --prefix=/usr/local/nginx && make && make install \
    && ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/ \
    && nginx

RUN rm -rf /usr/src/nginx-nginx-1.15.8
EXPOSE 80
CMD ['nginx', '-g', 'daemon off;']
```

运行 `docker build` 构建

```sh
docker build [OPTIONS] PATH | URL | -
```

OPTIONS有如下几个常用的：

```md
* --build-arg=[]: 设置镜像创建时的变量
* -f: 指定要使用的Dockerfile路径
* --force-rm: 设置镜像过程中删除中间容器
* --rm: 设置镜像成功后删除中间容器
* --tag,-t:镜像的名字以及标签，通常 name:tag 或者 name 格式; 
```

