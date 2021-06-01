## 一：容器内部结构

1, 在容器中执行命令

docker exec [it] 容器id 

exec在对应容器中执行命令

it 采用交互方式执行命令

docker exec it xxxx /bin/bash

## 二：容器声明周期

![](D:\MyWork\MarkDownPicture\docker\生命周期.png)

## 三：使用docker镜像

1， 获取镜像 docker pull 默认下载的是最新的镜像

docker image pull name[:tag]

name 是镜像仓库的名称，tag是标签，往往用来表示版本信息，不指定通常表示最新的版本

### 查看镜像信息

docker images

docker image ls

### 使用

