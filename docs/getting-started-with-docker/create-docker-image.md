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
