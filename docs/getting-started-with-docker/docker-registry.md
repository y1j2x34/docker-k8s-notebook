# Docker Hub 镜像加速器

## 配置加速地址

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://docker.mirrors.ustc.edu.cn",
        "http://f1361db2.m.daocloud.io",
        "https://registry.docker-cn.com"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```


## 验证是否生效

```sh
docker info
```

## Docker Hub 镜像加速器列表

|镜像加速器|镜像加速器地址|专属加速器？|其它加速？|
|--|--|--|--|
|Docker 中国官方镜像|https://registry.docker-cn.com|Docker Hub
|DaoCloud 镜像站|http://f1361db2.m.daocloud.io|可登录，系统分配|Docker Hub
|Azure 中国镜像|https://dockerhub.azk8s.cn|Docker Hub、GCR、Quay|
|科大镜像站|https://docker.mirrors.ustc.edu.cn|Docker Hub、GCR、Quay|
|阿里云|https://&lt;your_code>.mirror.aliyuncs.com|需登录，系统分配|Docker Hub
|七牛云|https://reg-mirror.qiniu.com|Docker Hub、GCR、Quay|
|网易云|https://hub-mirror.c.163.com|Docker Hub|
|腾讯云|https://mirror.ccs.tencentyun.com|Docker Hub|
