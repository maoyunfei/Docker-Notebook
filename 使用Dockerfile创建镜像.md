# 使用Dockerfile创建镜像

Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile来快速创建自定义的镜像。

## 基础结构

Dockerfile由一行行命令语句组成，并且支持以`#`开头的注释行。

一般而言，Dockerfile分为四部分：**基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令**。

其中，一开始必须指明所基于的镜像名称，接下来一般是说明维护者信息。后面则是镜像操作指令，例如`RUN`指令，`RUN`指令将对镜像执行跟随指令。每运行一条`RUN`指令，镜像就添加一层，并提交。最后是`CMD`指令，用来指定运行容器时的操作命令。

## 指令说明

|    指令    | 说明 |
| ---------- | --- |
| FROM |  指定所创建镜像的基础镜像 |
| MAINTAINER |  指定维护者信息 |
| RUN |  运行命令 |
| CMD |  指定容器启动时默认执行的命令 |
| LABEL |  指定生成镜像的元数据标签信息 |
| EXPOSE |  声明镜像内服务所监听的端口 |
| ENV |  指定环境变量 |
| ADD |  复制指定的<src>路径下的内容到容器中的<dest>路径下，<src>可以为URL；<br/>如果为tar文件，会自动解压到<dest>路径下 |
| COPY |  复制本地主机的<src>路径下的内容到镜像中的<dest>路径下；<br/>一般情况下推荐使用COPY，而不是ADD |
| ENTRYPOINT |  指定镜像的默认入口 |
| VOLUME |  创建数据卷挂载点 |
| USER |  指定运行容器时的用户名或UID |
| WORKDIR |  配置工作目录 |
| ARG |  指定镜像内使用的参数 |
| ONBUILD |  配置当所创建的镜像作为其他镜像的基础镜像时，<br/>所执行的创建操作指令 |
| STOPSIGNAL |  容器退出的信号值 |
| HEALTHCHECK |  如何进行监控检查 |
| SHELL |  指定使用shell时的默认shell类型 |

### FROM

格式为`FROM <image>`或`FROM <image>:<tag>`或`FROM <image>@<digest>`。

### MAINTAINER

格式为`MAINTAINER <name>`。

例如：

```docker
MAINTAINER image_creator@docker.com
```
### RUN

格式为`RUN <command>`或`RUN ["executable", "param1", "param2"]`。注意后一个指令会被解析为Json数组，因此必须用双引号。

前者默认将在shell终端中运行命令，即`/bin/sh -c`；后者则使用exec执行，不会启动shell环境。

当命令较长时可以使用`\`来换行。

例如：

```docker
RUN apt-get update \
	&& apt-get install -y libsnappy-dev zlib1g-dev libbz2-dev \
	&& rm -rf /var/cache/apt
```

### CMD

支持三种格式：

* `CMD ["executable", "param1", "param2"]`使用`exec`执行，是推荐使用的方式
* `CMD command param1 param2`在`/bin/sh`中执行，提供给需要交互的应用
* `CMD ["param1", "param2"]`提供给ENTRYPOINT的默认参数

每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时手动指定了运行的命令(作为run的参数)，则会覆盖掉CMD指定的命令。

### LABEL

格式为`LABEL <key>=value <key>=value <key>=value ...`。

例如：

```docker
LABEL version="1.0"
```

### EXPOSE

格式为`EXPOSE <port> [<port>...]`。

例如：

```docker
EXPOSE 22 80 8443
```
注意，该指令只是起到声明作用，并不会自动完成端口映射。

### ENV

格式为`ENV <key><value>`或`ENV <key>=<value>...`。

例如：

```docker
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
```

### ADD

格式为`ADD <src> <dest>`。

例如：

```docker
ADD *.c /code/
```

### COPY

格式为`COPY <src> <dest>`。

当使用本地目录为源目录时，推荐使用COPY。

### ENTRYPOINT

支持两种格式：

* `ENTRYPOINT ["executable", "param1", "param2"]`(exec调用执行)
* `ENTRYPOINT command param1 param2`(shell中执行)

每个Dockerfile中只能有一个ENTRYPOINT，当指定多个时，只有最后一个有效。在运行时，可以被`--entrypoint`参数覆盖掉。

### VOLUME

格式为`VOLUME ["/data"]`。

可以从本地主机或其他容器挂载数据卷，一般用来存放数据库和需要保存的数据等。

### USER

格式为`USER daemon`。

### WORKDIR

格式为`WORKDIR /path/to/workdir`。

可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。

例如：

```docker
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
则最终路径为`/a/b/c`。

### ARG

格式为`ARG <name>[=<default value>]`。

在执行`docker build`命令时才以`--build-arg<varname>=<value>`格式传入。

### ONBUILD

格式为`ONBUILD [INSTRUCTION]`。

### STOPSIGNAL

格式为`STOPSIGNAL signal`。

### HEALTHCHECK

支持两种格式：

* `HEALTHCHECK [OPTION] CMD command`(根据执行命令返回值是否为0来判断)
* `HEALTHCHECK NONE`(禁止镜像中的健康检查)

OPTION支持：

* `--interval=DURATION`(默认为：30s)：过多久检查一次
* `--timeout=DURATION`(默认为：30s)：每次检查等待结果的超时
* `--retries=N`(默认为：3)：如果失败了，重试几次才最终确认失败

### SHELL

格式为`SHELL ["executable", "parameters"]`。

默认值为`["/bin/bash", "-c"]`。

## 创建镜像

格式为`docker build [选项] 内容路径`。

该命令将读取指定路径下的Dockerfile，并将该路径下的所有内容发送给Docker服务端，由服务端来创建镜像。因此除非生成镜像需要，否则一般建议放置Dockerfile的目录为空目录。

有两点经验：

* 如果使用非内容路径下的Dockerfile，可以使用`-f`选项。
* 要指定生成镜像的标签信息，可以使用`-t`选项。


例如：

```docker
docker build -t build_repo/first_image /tmp/docker_builder/
```

## 使用`.dockerignore`文件

可以通过`.dockerignore`文件来让Docker忽略匹配模式路径下的目录和文件。

例如：

```
# comment
    */tmep*
    */*/temp*
    tmp?
    ~*
```


