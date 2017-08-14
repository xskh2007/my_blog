---
title: docker入门练习
date: 2017-08-14 10:38:16
categories:	docker
tags: 
	- docker
---

### 准备开始

Docker系统有两个程序：docker服务端和docker客户端。其中docker服务端是一个服务进程，管理着所有的容器。docker客户端则扮演着docker服务端的远程控制器，可以用来控制docker的服务端进程。大部分情况下，docker服务端和客户端运行在一台机器上。

### 安装
    yum install docker
    
### 修改成阿里云镜像
在文件/usr/lib/systemd/system/docker.service 添加 --registry-mirror=https://y3uxccn6.mirror.aliyuncs.com

    [root@localhost ~]# vim /usr/lib/systemd/system/docker.service
    ExecStart=/usr/bin/dockerd-current \
              --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current \
              --default-runtime=docker-runc \
              --exec-opt native.cgroupdriver=systemd \
              --userland-proxy-path=/usr/libexec/docker/docker-proxy-current \
              --registry-mirror=https://y3uxccn6.mirror.aliyuncs.com
              $OPTIONS \
              $DOCKER_STORAGE_OPTIONS \
              $DOCKER_NETWORK_OPTIONS \
              $ADD_REGISTRY \
              $BLOCK_REGISTRY \
              $INSECURE_REGISTRY

    
### 搜索
    [root@localhost ~]# docker search tutorial
    INDEX       NAME                                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    docker.io   docker.io/learn/tutorial                                                                          28                   
    docker.io   docker.io/georgeyord/reactjs-tutorial             This is the backend of the React comment b...   4                    [OK]
    docker.io   docker.io/egamas/docker-tutorial                  Funny manpages                                  2                    [OK]
    docker.io   docker.io/mhausenblas/kairosdb-tutorial           GitHub fetcher for KairosDB tutorial            1                    [OK]
    docker.io   docker.io/mjansche/tts-tutorial                   Software for a Text-to-Speech tutorial          1                    [OK]
    docker.io   docker.io/abarkai/aws-lambda-ecs-tutorial                                                         0                    
    docker.io   docker.io/activeeon/par-connector-tutorial        Do the par-connector tutorial with R. The ...   0                    [OK]
    docker.io   docker.io/biopython/biopython-tutorial            Biopython with Tutorial running on top of ...   0                    [OK]
    docker.io   docker.io/camphor/python-tutorial                 camphor-/python-tutorial                        0                    [OK]
    docker.io   docker.io/chris24walsh/flask-aws-tutorial         Runs a simple flask webapp demo, with the ...   0                    [OK]
    docker.io   docker.io/cloudboost/tutorial                                                                     0                    
    docker.io   docker.io/imiell/git-101-tutorial                                                                 0                    
    docker.io   docker.io/intrig/tutorial                                                                         0                    
    docker.io   docker.io/jbalexandre/docker-tutorial                                                             0                    
    docker.io   docker.io/kidikarus/concourse-tutorial-47-tasks                                                   0                    
    docker.io   docker.io/kobe25/docker-tutorial                  Docker Tutorial                                 0                    [OK]
    docker.io   docker.io/lmcluck/tutorial                        online tutorial example                         0                    
    docker.io   docker.io/lukasheinrich/quickana-tutorial         Image for the analysis code built from htt...   0                    
    docker.io   docker.io/michelesr/docker-tutorial               Docker Tutorial                                 0                    [OK]
    docker.io   docker.io/onekit/rest-tutorial                    REST API server-side tutorial. How to do i...   0                    [OK]
    docker.io   docker.io/paddledev/paddle-tutorial               images that paddle tutorials use.               0                    
    docker.io   docker.io/paulcos11/docker-tutorial               docker tutorial                                 0                    [OK]
    docker.io   docker.io/schwamster/docker-tutorial                                                              0                    
    docker.io   docker.io/starkandwayne/concourse-tutorial                                                        0                    
    docker.io   docker.io/starkandwayne/concourse-tutorial-ci                                                     0                    
    
### 下载容器镜像
    [root@localhost ~]# docker pull learn/tutorial
    Using default tag: latest
    Trying to pull repository docker.io/learn/tutorial ... 
    latest: Pulling from docker.io/learn/tutorial
    
    Digest: sha256:2933b82e7c2a72ad8ea89d58af5d1472e35dacd5b7233577483f58ff8f9338bd

### 在docker容器中运行hello world!
    [root@localhost ~]# docker run learn/tutorial echo "hello word"
    hello word

### 在容器中安装新的程序
    [root@localhost ~]# docker run learn/tutorial apt-get install -y ping
    Reading package lists...
    Building dependency tree...
    The following NEW packages will be installed:
      iputils-ping
    0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
    Need to get 56.1 kB of archives.
    After this operation, 143 kB of additional disk space will be used.
    Get:1 http://archive.ubuntu.com/ubuntu/ precise/main iputils-ping amd64 3:20101006-1ubuntu1 [56.1 kB]
    debconf: delaying package configuration, since apt-utils is not installed
    Fetched 56.1 kB in 1s (40.3 kB/s)
    Selecting previously unselected package iputils-ping.
    (Reading database ... 7545 files and directories currently installed.)
    Unpacking iputils-ping (from .../iputils-ping_3%3a20101006-1ubuntu1_amd64.deb) ...
    Setting up iputils-ping (3:20101006-1ubuntu1) ...

### 保存对容器的修改
    [root@localhost ~]# docker ps -l
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                          PORTS               NAMES
    af2aa74ff1d4        learn/tutorial      "apt-get install -y p"   About a minute ago   Exited (0) About a minute ago                       berserk_shannon
    
    docker commit af2aa74ff1d4 learn/ping

### 运行新的镜像
    [root@localhost ~]# docker run learn/ping ping www.baidu.com
    PING www.a.shifen.com (58.217.200.15) 56(84) bytes of data.
    64 bytes from 58.217.200.15: icmp_req=1 ttl=54 time=9.75 ms
    64 bytes from 58.217.200.15: icmp_req=2 ttl=54 time=12.4 ms
    64 bytes from 58.217.200.15: icmp_req=3 ttl=54 time=10.1 ms

### 检查运行中的镜像

现在你已经运行了一个docker容器，让我们来看下正在运行的容器。

使用docker ps命令可以查看所有正在运行中的容器列表，使用docker inspect命令我们可以查看更详细的关于某一个容器的信息。

    [root@localhost ~]# docker inspect learn/ping
    [
        {
            "Id": "sha256:e1c66d8e080650b256745febff36b9460f1a9cb47fadaefad05db95ce5571a34",
            "RepoTags": [
                "learn/ping:latest"
            ],
            "RepoDigests": [],
            "Parent": "sha256:a7876479f1aae32c0716d7a85b5151af26f533fe48efa086010105cba02f5163",
            "Comment": "",
            "Created": "2017-08-14T10:38:27.403206253Z",
            "Container": "af2aa74ff1d4e65e90dfdbb71d54031bdb8146274e914c39da6c4b545488787f",
            "ContainerConfig": {
                "Hostname": "af2aa74ff1d4",
                "Domainname": "",
                "User": "",
                "AttachStdin": false,
                "AttachStdout": true,
                "AttachStderr": true,
                "Tty": false,
                "OpenStdin": false,
                "StdinOnce": false,
                "Env": [],
                "Cmd": [
                    "apt-get",
                    "install",
                    "-y",
                    "ping"
                ],
                "Image": "learn/tutorial",
                "Volumes": {},
                "WorkingDir": "",
                "Entrypoint": null,
                "OnBuild": null,
                "Labels": {}
            },
            "DockerVersion": "1.12.6",
            "Author": "",
            "Config": {
                "Hostname": "",
                "Domainname": "",
                "User": "",
                "AttachStdin": false,
                "AttachStdout": false,
                "AttachStderr": false,
                "Tty": false,
                "OpenStdin": false,
                "StdinOnce": false,
                "Env": [],
                "Cmd": [
                    "apt-get",
                    "install",
                    "-y",
                    "ping"
                ],
                "Image": "",
                "Volumes": {},
                "WorkingDir": "",
                "Entrypoint": null,
                "OnBuild": null,
                "Labels": {}
            },
            "Architecture": "amd64",
            "Os": "linux",
            "Size": 139470427,
            "VirtualSize": 139470427,
            "GraphDriver": {
                "Name": "devicemapper",
                "Data": {
                    "DeviceId": "72",
                    "DeviceName": "docker-253:0-8471489-46df8901e0b6b7963ff68b92537694006f6a2e78da4d47b325d0d6962ac4f1cf",
                    "DeviceSize": "10737418240"
                }
            },
            "RootFS": {
                "Type": "layers",
                "Layers": [
                    "sha256:ee1ba0cc9b81862329c6aeab9cbc44adcc56b33905bee97515c24e918f3a58e1",
                    "sha256:a18c8c9eeeda2764953d2b01923ad9347b78c7e2314753dd16ccf44c53b60539"
                ]
            }
        }
    ]

### 发布自己的镜像到阿里云

    [root@localhost ~]# docker login --username=qiantu1986@126.com registry.cn-hangzhou.aliyuncs.com
    Password: 
    Login Succeeded
    [root@localhost ~]# docker images
    REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
    learn/ping                                     latest              e1c66d8e0806        5 minutes ago       139.5 MB
    learn/mytest                                   latest              0c6359071eea        2 hours ago         128 MB
    registry.cn-hangzhou.aliyuncs.com/qiantu/k8s   latest              fa6c646a6eae        2 hours ago         139.5 MB
    <none>                                         <none>              aeb9d4ef2dbf        3 hours ago         128 MB
    docker.io/learn/tutorial                       latest              a7876479f1aa        4 years ago         128 MB
    [root@localhost ~]# docker tag e1c66d8e0806 registry.cn-hangzhou.aliyuncs.com/qiantu/k8s:latest
    [root@localhost ~]# docker push registry.cn-hangzhou.aliyuncs.com/qiantu/k8s
    The push refers to a repository [registry.cn-hangzhou.aliyuncs.com/qiantu/k8s]
    d165e989ffd2: Layer already exists 
    ee1ba0cc9b81: Layer already exists 
    latest: digest: sha256:02c0349a6c73fbab31d9929a0be0309c3b87d78d523e79d7c03adb4e54d5d277 size: 740