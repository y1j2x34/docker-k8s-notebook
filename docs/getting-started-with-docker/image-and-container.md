# Docker 镜像和容器

## Docker 镜像

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包括一些为运行时准备的一些配置参数（匿名卷、环境变量、用户==）。镜像不包含动态数据，其内容在构建后也不会被改变。

### Docker 镜像层

Docker 镜像是由镜像层构成，Dockerfile 的每一条指令都会对应了一个镜像层, 每个镜像层相互堆叠，每个层只存储差异信息。<!--（Docker 使用 UnionFS 将这些不同的层结合到一个镜像中去。）--> 当创建一个新的容器时，相当于在这些镜像层顶层添加一个可写层（容器层），除了这一层，其它镜像层都是只读的。
容器中所有操作（写或修改文件，删除文件）都在这个容器层中进行，结构如下图所示：

![镜像层|容器层](./container-layers.jpg)

### 镜像操作

- 搜索镜像：`docker search`
- 获取镜像：`docker pull`
- 查看镜像：`docker images`
- 删除镜像：`docker rmi`

## Docker 容器

容器和镜像最大的区别是在顶层的可写层。所有对数据的增删改操作都是在这个可写层进行。当容器被删除时，容器对应的可写层也会被删除，镜像则被保留下来。 每一个容器对应都一个可写层，有多个容器就多个可写层，这些层都可以同时指向同一个镜像，每个可写层数据相互独立。参考下面的图片：

![共享镜像层](./sharing-layers.jpg)

>
> **Note:** 如果需要多个镜像共享同一份数据的访问权，就需要用到 Docker volumn 并装载到容器中。
>

### 操作

- 启动镜像：`docker run centos /bin/echo "hello world"`， `docker run --name mycentos -it centos /bin/bash`
  - `-d` 后台运行容器，并返回容器ID
  - `-i` 以交互模式运行容器，通常与 `-t` 同时使用
  - `-p` 端口映射，格式为：*主机端口:容器端口* 或 *IP:主机端口:容器端口* 或 *IP:容器端口*
  - `-t` 为容器重新分配一个伪输入终端，通常与 `-i` 同时使用
  - `-e PARAM1="mario" -e PARAM2="XXX"` 设置环境变量
  - `--env-file=filepath` 从文件导入环境变量
  - `-m` 容器使用内存最大值
- 查看运行（或已停止）的容器：`docker ps -a`
- 启动已终止的容器：`docker start <container-id>`
- 进入运行中的容器：`docker attach <container-id>`(容器进入之后Ctrl+C或者exit会把容器终止了。)，应该用 `nsenter --target <pid> --mount --uts --ipc --net --pid`命令进入（需要安裝util-linux）, PID 通过`docker inspect --format "{{.State.Pid}}" <container-id|container-name>`命令获取
- 终止运行中的容器：`docker stop <container-id>`。
- 删除容器：`docker rm <container-id>`(容器必须先停止)
