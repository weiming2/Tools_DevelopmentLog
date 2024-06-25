### 1. 安装docker
[docker配置官方教程](https://github.com/autowarefoundation/autoware_ai_documentation/wiki/docker-installation)  
我这边按照官网教程配置是能够成功的，这里抄一遍简要流程：
- 删除旧版docker(如果有的话)，更新apt，并添加密钥
  ```
  $ sudo apt-get remove docker docker-engine docker.io
  $ sudo apt-get update
  $ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
  $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  $ sudo apt-key fingerprint 0EBFCD88
  $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  ```
- 安装docker，并从docker官网拉取hello-world镜像运行
  ```
  $ sudo apt-get update
  $ sudo apt-get install docker-ce
  $ sudo docker run hello-world
  ```
- 查看安装完的版本
  ```
  egbert@egbert:~$ docker --version
  Docker version 23.0.1, build a5ee5b1
  ```
- 将用户添加到docker组中   
当安装完docker后，基本所有命令都需要加上sudo，非常麻烦，这是因为当前用户没有这个权限，只需要把我们当前用户添加到docker组中即可
  ```
  $ sudo gpasswd -a <your username> docker   # 将用户添加到docker组中
  $ newgrp docker   # 更新docker组
  ```
  ~~如果上述这条不行，可以试试`sudo usermod -aG docker username`~~(上述由弘毅测试没问题)，**运行完重启才能生效**

### 2. 安装nvidia-docker2
nvidia-docker2是对docker的封装，提供一些必要的组件可以很方便的在容器中使用GPU资源执行代码，nvidia-docker2共享了宿主机的CUDA Driver  
通过命令行直接安装
```
$ sudo apt install nvidia-docker2
```
但有可能报软件定位失败错误`E:Unable to locate package nvidia-docker2`  
可以通过如下命令解决  
```
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
$ sudo apt-get update
$ sudo apt-get install -y nvidia-docker2
```  
安装完毕后重启docker使之生效  
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

写在前面，记录一些docker命令的操作:
- 容器使用
    - 获取镜像: `$ docker pull ubuntu`，这将从docker的公共镜像进行下载(如果本地不存在)，若不指定版本，默认下载latest版本；否则可以通过':version'指定版本
    - 启动容器: `$ docker run -it ubuntu /bin/bash`
        - -i：交互式操作
        - -t：生成一个伪终端
        - ubuntu：镜像名，这里使用刚刚获取的镜像ubuntu
        - /bin/bash：在启动的容器里执行的命令，**由于我们一般希望有个交互式shell，因此用的是/bin/bash**   
        - 也可以添加-d来指定容器的运行模式，如一般情况下我们希望docker在后台运行，则`$ docker run -itd --name ubuntu-test ubuntu /bin/bash`， 在添加-d后默认不会进入容器，但已经创建了一个新的正在运行的容器，要想进入容器需要使用指令**docker exec(后面介绍)**  
        - 删除镜像: `$ docker rmi -f <images ID>`
    启动后会进入容器环境`egbert@egbert` -> `root@8406493241c0`
    - 查看所有容器: `$ docker ps -a`，结果如上图所示，我们可以看到id为840...的容器正在运行中，而其他两个容器已经停止
    - 启动一个容器: `$ docker start 840`: 其中840为容器的id，支持模糊搜索
    - 停止一个容器: `$ docker stop <容器ID>`: 同样支持模糊搜索
    - 重启一个容器: `$ docker restart <容器ID>`
    - 进入容器: 推荐使用`exec`命令而不使用`attach`命令，后者退出容器会导致容器的停止。`docker exec -it <容器ID> /bin/bash`
    - **导出和导入容器**，实现在现有镜像(如ubuntu基础镜像)上搭建环境，进行二次封装
        - 导出容器: `docker export 840 > ubuntu.tar`，这将导出ID为840的容器快照到本地文件ubuntu.tar
        - 导入容器: 可以使用`docker import`将容器快照导入为镜像，如`$ cat <your_path>/ubuntu.tar | docker import - test/ubuntu:v1`，则将刚刚导出的快照导入到镜像test/ubuntu:v1
    
    - 删除容器: `$ docker rm -f <容器ID>`
