# Docker镜像主要命令

[官网地址](https://docs.docker.com/engine/reference/commandline/docker/)

## 搜索镜像

使用`docker search`命令可以搜索远端仓库共享的镜像，默认是Docker Hub中的镜像。

命令格式：

```docker
docker search [options] TERM
```

参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--automated`	 |  `false` |  仅显示自动构建的镜像   |
| `--filter , -f`       |  | 根据指定条件过滤    |
| `--limit` | `25` | 搜索结果的最大条数    |
| `--no-trunc` | `false` | 输出信息不截断显示 |
| `--stars , -s` | `0` | 仅显示star大于指定星级的镜像，0表示输出所有镜像 |

示例：

```docker
docker search nginx
```
```docker
docker search -s 10 nginx
```
## 下载镜像

使用`docker pull`命令直接从Docker Hub下载镜像。

命令格式：

```docker
docker pull [options] NAME[:TAG]
```

其中，NAME是镜像仓库的名称，TAG是镜像的标签。通常情况下，描述一个镜像需要包括“名称+标签”信息。

参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--all-tags , -a`	 |  `false` |  下载所有标签的镜像   |
| `--disable-content-trust` | `false` | 忽略镜像内容校验  |

示例：

```docker
docker pull nginx
```
如果不指定TAG，默认下载镜像的最新版本。

```docker
docker pull myregistry.com/nginx:tag1
```
## 列出镜像

使用`docker images`命令可以列出本机上已有镜像的基本信息。

命令格式：

```docker
docker images [options] [REPOSITORY[:TAG]]
```
参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--all , -a` |  `false` |  列出所有镜像(包括隐藏中间层镜像)   |
| `--digests`  |  `false` | 显示摘要信息    |
| `--filter , -f` |  | 显示满足条件的镜像    |
| `--format` |  | 使用Go语言模板文件展示镜像 |
| `--no-trunc` | `false` | 输出信息不截断显示 |
| `--quite , -q` | `false` | 只显示镜像ID |

示例：

```docker
docker images
docker images nginx
docker images nginx:tag1
docker images --digests
docker images --filter "dangling=true"
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```
## 添加标签

使用`docker tag`命令为本地镜像添加新的标签

命令格式：

```docker
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```
示例：

```docker
docker tag 0e5574283393 fedora/httpd:version1.0
docker tag httpd fedora/httpd:version1.0
docker tag httpd:test fedora/httpd:version1.0.test
```

## 删除镜像

使用`docker rmi`命令可以删除指定镜像。

命令格式：

```docker
docker rmi [options] IMAGE [IMAGE...]
```
参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--force, -f`	 |  `false` |  强制删除   |
| `--no-prune` | `false` | 不移除该镜像的过程镜像，默认移除  |

示例：

```docker
docker rmi nginx
docker rmi ${ID}
```
删除所有镜像:

```docker
docker rmi $(docker images)
```
## 构建镜像

通过Dockerfile构建镜像。

命令格式：

```docker
docker build [options] path | url | -
```

参数：

| 参数 | 默认值 | 说明 |
| ---------- | --- | --- |
| `--file, -f`	 |   |  指定Dockerfile的名称，默认是‘PATH/Dockerfile’   |
| `--no-prune` | `false` | 不移除该镜像的过程镜像，默认移除  |
