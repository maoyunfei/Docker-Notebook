# Docker容器主要命令

[官网地址](https://docs.docker.com/engine/reference/commandline/docker/)

## 新建并启动容器

使用`docker run`命令可以新建并启动一个容器。

命令格式：

```docker
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `-d`	 |   |  后台运行   |
| `-P` |  | 随机端口映射    |
| `-p` |  | 指定端口映射 (格式：-p hostPort:containerPort)  |
| `--network` |  | 指定网络模式  |

 
示例：

```docker
docker run java /bin/echo 'Hello World'
```
```docker
docker run -d -p 91:80 nginx
```
**提醒：**使用docker run命令创建容器时，会先检查本地是否存在指定镜像。如果本地不存在该名称的镜像，Docker就会自动从Docker Hub下载镜像并启动一个Docker容器。

## 列出容器

使用`docker ps`命令列出运行中的容器。

命令格式：

```docker
docker ps [options]
```

参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--all , -a`	 |  `false` |  列出所有容器(包括未运行的)   |
| `--filter , -f` |  | 根据条件过滤  |
| `--last , -n` | `-1` | 显示最近创建的n个容器(无论状态)  |
| `--no-trunc` | `false` | 不截断输出  |
| `--quite , -q` | `false` | 静默模式，只显示容器ID  |
| `--size , -s` | `false` | 显示总文件大小  |

示例：

```docker
docker ps -n 10
```
```docker
docker ps -a -q
```
## 停止容器

使用`docker stop`命令可以停止容器。

命令格式：

```docker
docker stop [options] CONTAINER [CONTAINER...]
```
参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--time , -t` |  `10` |  杀死容器前等待其停止的时间，单位是秒   |

示例：

```docker
docker stop 784fd3b294d7
docker stop nginx
```
## 强行停止容器

使用`docker kill`命令强行停止容器

命令格式：

```docker
docker kill [options] CONTAINER [CONTAINER...]
```

参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--siginal , -s` |  `KILL` |  向容器发送信号   |
示例：

```docker
docker kill 784fd3b294d7
docker kill nginx
```

## 启动已停止容器

使用`docker start`命令可以启动已停止的容器。

命令格式：

```docker
docker start [options] CONTAINER [CONTAINER...]
```

示例：

```docker
docker start 784fd3b294d7
docker start nginx
```
## 重启容器

使用`docker restart`命令可以重启容器。实际上是先执行了`docker stop`命令，然后再执行`docker start`命令。

命令格式：

```docker
docker restart [options] CONTAINER [CONTAINER...]
```

参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--time , -t` |  `10` |  杀死容器前等待其停止的时间，单位是秒   |

示例：

```docker
docker restart 784fd3b294d7
docker restart nginx
```

## 进入容器

有多种方式进入容器，最简单的方式是使用`docker exec`命令。

命令格式：

```docker
docker exec -it 容器ID /bin/bash
```
示例：

```docker
docker exec -it 784fd3b294d7 /bin/bash
```
## 删除容器

使用`docker rm`命令可以删除容器。

命令格式：

```docker
docker rm [options] CONTAINER [CONTAINER...]
```

参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--force , -f` |  `false` |  通过SIGKILL信号强制删除正在运行中的容器  |
| `--link , -l` |  `false` |  删除容器间的网络连接  |
| `--volumes , -v` |  `false` |  删除与容器关联的卷  |

示例：

```docker
docker rm 784fd3b294d7
docker rm -f nginx
```
删除所有的容器:

```docker
docker rm -f ${docker ps -a -q}
```


