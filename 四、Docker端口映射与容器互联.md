# Docker端口映射与容器互联

除了通过网络访问外，Docker还提供了两个很方便的功能来满足服务访问的基本需求：

* 一个是允许映射容器内应用的服务端口到本地宿主主机；
* 另一个是互联机制实现多个容器间通过容器名来快速访问。

## 端口映射实现访问容器

### 从外部访问容器应用

在启动容器时，如果不指定对应的参数，在容器外是无法通过网络访问容器内的网络应用和服务的。

当容器中运行一些网络应用，要让外部访问这些应用时，可以通过`-P`或`-p`参数来指定端口映射。当使用`-P`(大写)标记时，Docker会随机映射一个49000~49900的端口到内部容器开放的网络端口。

`-p`(小写)可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有`IP:HostPort:ContainerPort` | `IP::ContainerPort` | `HostPort:ContainerPort`。

### 映射所有接口地址

使用`HostPort:ContainerPort`默认会绑定所有接口上的所有地址，多次使用`-p`标记可以绑定多个端口。

```docker
docker run -d -p 5000:5000 -p 3000:80 training/webapp python app.py
```

### 映射到指定地址的指定端口

可以使用`IP:HostPort:ContainerPort`格式指定映射使用一个特定地址。

```docker
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

### 映射到指定地址的任意端口

可以使用`IP::ContainerPort`格式绑定指定地址任意端口到容器端口，本地主机会自动分配一个端口。

```docker
docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

### 查看映射端口配置

使用`docker port`命令可以查看当前映射的端口配置，也可以查看到绑定的地址：

```docker
docker port nostalgic_morse 5000
```

**注意：** 容器有自己的内部网络和IP地址，使用`docker inspect + 容器ID`可以获取容器的具体信息。

## 互联网机制实现便捷互访

容器的互联是一种让多个容器中应用进行快速交互的方式。它会在源和接收容器之间创建连接关系，接收容器可以通过容器名快速访问到源容器，而不用指定具体的IP地址。

### 自定义容器命名

连接系统依据容器的名称来执行，因此首先需要定义一个好记的容器名，虽然创建容器时系统会默认分配一个名字。

使用`--name`标记可以为容器自定义命名：

```docker
docker run -d -P --name web training/webapp python app.py
```
**注意：** 容器的名称是唯一的。如果已经命名了一个叫web的容器，当要再次使用web这个名称的时候，需要先用docker rm来删除之前创建的同名容器。

### 容器互联

使用`--link`参数可以让容器之间安全地进行交互。

`--link`参数的格式为`--link name:alias`，其中`name`是要连接的容器名称，`alias`是这个连接的别名。

新建一个web容器，并将它连接到db容器：

```docker
docker run -d -P --name web --link db:db training/webapp python app.py
```
使用docker ps可以看到db容器的names列有db，也有web/db，这表示web容器连接到db容器，这允许web容器访问db容器的信息。

**Docker相当于在两个互联的容器之间创建了一个虚机通道，而且不用映射它们的端口到宿主机上。**

### docker通过两种方式来为容器公开连接信息

* 更新环境变量
* 更新`/etc/hosts`文件

使用`env`命令来查看web容器的环境变量：

```docker
$ docker run -rm --name web --link db:db training/webapp env
...
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5000_TCP=tcp://172.17.0.5:5432
DB_PORT_5000_TCP_PROTO=tcp
...
```

其中DB_开头的环境变量是供web容器连接db容器使用的，前缀采用大写的连接别名。

除了环境变量之外，Docker还添加host信息到父容器的`/etc/hosts`文件。

下面是父容器web的hosts文件：

```docker
$ docker run -rm --name web --link db:db training/webapp /bin/bash
$ cat /etc/hosts
172.17.0.7 aed84ee21bde
...
172.17.0.5 db
```

这里有两个hosts信息，第一个是web容器，web容器用自己的id作为默认主机名，第二个是db容器的IP和主机名。

用户可以连接多个子容器到父容器。





