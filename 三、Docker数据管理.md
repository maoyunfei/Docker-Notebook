# Docker数据管理

容器中管理数据主要有两种：

* 数据卷：容器内数据直接映射到本地主机环境；
* 数据卷容器：使用特定容器维护数据卷。

## 数据卷

数据卷是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于Linux中的mount操作。

数据卷可以提供很多有用的特性，如下所示：

* 数据卷可以在容器之间共享和重用，容器间传递数据将变得高效方便；
* 对数据卷内数据的修改会立马生效，无论容器内操作还是本地操作；
* 对数据卷的更新不会影响镜像，解耦了应用和数据；
* 卷会一直存在，直到没有容器使用，可以安全地卸载它。

### 在容器内创建一个数据卷

在用`docker run`命令时，可以使用`-v`来创建一个数据卷。从此重复使用`-v`可以创建多个数据卷。

使用training/webapp创建一个web容器，并创建一个数据卷挂载到容器的/webapp目录。
```docker
docker run -d -P --name web -v /webapp training/webapp python app.py
```

### 挂载一个主机目录作为数据卷(推荐)

使用`-v`标记也可以指定挂载本地的已有目录到容器中去作为数据卷。
```docker
docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```
**本地目录的路径必须是绝对路径，如果目录不存在，docker会自动创建。**

docker挂载数据卷的默认权限是读写(rw)，用户也可以通过ro指定为只读：

```docker
docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py
```
## 数据卷容器

如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。数据卷容器也是一个容器，但是它的目的是专门用来提供数据卷供其他容器挂载。

* 创建一个数据卷容器，并在其中创建一个数据卷挂载到/dbdata:

```docker
docker run -it -v /dbdata --name dbdata ubuntu
```

* 查看/dbdata目录：

```linux
$ ls
bin boot dbdata dev src tec ...
```

* 创建db1和db2两个容器，并从dbdata容器挂载数据卷：

```docker
docker run -it --volumes-from dbdata --name db1 ubuntu
docker run -it --volumes-from dbdata --name db2 ubuntu
```
此时，容器db1和db2都挂载同一个数据卷到相同的/dbdata目录。三个容器任何一方在该目录下的写入，其他容器都可以看见。

可以多次使用`--volomus-from`参数来从多个容器加载多个数据卷。还可以从其他已经挂载了容器卷的容器来挂载数据卷。

```docker
docker run -d --name db3 --volumes-from db1 training/postgres
```
**注意：** 使用`--volumes-from`参数所挂载的数据卷的容器自身并不需要保持在运行状态。
如果删除了挂载的容器，数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用`docker rm -v`命令来指定同时删除关联的数据卷。
