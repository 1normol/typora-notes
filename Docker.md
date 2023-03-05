# Docker



## 安装Docker(Centos)

### 卸载Docker

#### 0.1 卸载旧版本

```sh
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 0.2 卸载 Docker

```sh
#删除安装包： 
yum remove docker-ce
#删除镜像、容器、配置文件等内容：
rm -rf /var/lib/docker
```

### 1. 安装 Docker Engine-Community

#### 1.1 设置仓库

```sh
#安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。
  sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

#### 1.2 切换国内镜像

```sh
#这里以阿里云镜像为例
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### 1.3 安装Docker Engine-Community

#### 1.3.1安装最新版本

```sh
#安装最新版本的 Docker Engine-Community 和 containerd，或者转到下一步安装特定版本：
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
#安装过程中会提示是否接受GPG秘钥，选择yes
```

#### 1.3.2安装指定版本

```sh
 
 #1.列出列出并排序您存储库中可用的版本。此示例按版本号（从高到低）对结果进行排序
yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable

#2.通过其完整的软件包名称安装特定版本，该软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。例如：docker-ce-18.09.1。
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

#### 1.4 启动Docker

```sh
 #启动Dokcer服务
 sudo systemctl start docker
 #设置Docker开机自启动
 sudo systemctl enable docker
```



## 安装Docker-Compose

```
#官方文档
https://docs.docker.com/compose/install/
```

### 1.1 通过Github安装Docker-Compose

```sh
sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

#若Github访问太慢，可以使用daocloud
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

### 1.11通过pip安装

```sh
sudo pip install docker-compose
```

### 1.2 添加可执行权限

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

$ docker-compose --version

docker-compose version 1.16.1, build 1719ceb