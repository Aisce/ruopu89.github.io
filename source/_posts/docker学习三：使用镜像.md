---
title: docker学习三：使用镜像
date: 2018-09-20 14:26:06
tags: docker
categories: Container
---

### 获取镜像

```shell
* 命令格式
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
# 具体的选项可以通过docker pull --help命令看到
# Docker 镜像仓库地址：地址的格式一般是<域名/IP>[:端口号]。默认地址是 Docker Hub。
# 仓库名：这里的仓库名是两段式名称，即<用户名>/<软件名>。对于 Docker Hub,如果不给出用户名，则默认为library，也就是官方镜像。

* 例：
root@test:~# docker pull ubuntu:16.04
16.04: Pulling from library/ubuntu
18d680d61657: Pull complete 
0addb6fece63: Pull complete 
78e58219b215: Pull complete 
eb6959a66df2: Pull complete 
Digest: sha256:76702ec53c5e7771ba3f2c4f6152c3796c142af2b3cb1a02fce66c697db24f12
Status: Downloaded newer image for ubuntu:16.04
# 上面的命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub 获取镜像。而镜像名称是ubuntu:16.04，因此将会获取官方镜像library/ubuntu仓库中标签为16.04的镜像。
```

> <font color=red>从下载过程中可以看到我们之前提及的分层存储的概念，镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的sha256的摘要，以确保下载一致性。层 ID 以及
> sha256
> 的摘要不总是一样的。这是因为官方镜像是一直在维护的，有任何新的 bug，或者版本更新，都会进行修复再以原来的标签发布，这样可以确保任何使用这个标签的用户可以获得更安全、更稳定的镜像。</font>



#### 运行

```shell
root@test:~# docker run -it --rm ubuntu:16.04 bash
root@82e7f3542e12:/# cat /etc/os-release 
NAME="Ubuntu"
VERSION="16.04.5 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.5 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial
# 以上面的ubuntu:16.04为基础，使用上面命令进行交互式操作
# docker run是运行容器的命令
# -it：这是两个参数，一个是-i：交互式操作；一个是-t：终端。我们这里打算进入bash执行一些命令并查看返回结果，因此我们需要交互式终端。
# --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动docker rm。我们这里只是随便执行个命令，看看结果，因此使用--rm可以避免浪费空间。
# ubuntu:16.04：这是指用ubuntu:16.04镜像为基础来启动容器
# bash：放在镜像名后的是命令，这里我们希望有个交互式shell，因此用的是bash。
# 进入容器后，我们可以在 Shell 下操作，执行任何所需的命令。这里，我们执行了cat /etc/os-release，这是 Linux 常用的查看当前系统版本的命令，从返回的结果可以看到容器内是Ubuntu 16.04.4 LTS系统。最后我们通过exit退出了这个容器。
```



### 列出镜像

```shell
root@test:~# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               4a689991aa24        4 weeks ago         116MB
ubuntu              latest              4a689991aa24        4 weeks ago         116MB
hello-world         latest              4ab4c602aa5e        2 months ago        1.84kB
# image ls可以替换为images，也就是docker images，显示的效果与上面的命令相同
# 列出已经下载下来的镜像。列表包含了仓库名、标签、镜像 ID、创建时间以及所占用的空间。
# 镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个标签。因此，在上面的例子中，我们可以看到ubuntu:latest和ubuntu:16.04拥有相同的 ID，因为它们对应的是同一个镜像。
```



#### 镜像体积

> <font color=red>如果仔细观察，会注意到，这里标识的所占用空间和在 Docker Hub 上看到的镜像大小不同。比如，ubuntu:16.04
> 镜像大小，在这里是
> 116MB，但是在 Docker Hub 显示的却是
> 50
> 。这是因为 Docker Hub 中显示的体积是压缩后的体积。在镜像下载和上传过程中镜像是保持着压缩状态的，因此 Docker Hub 所显示的大小是网络传输中更关心的流量大小。而
> docker image ls
> 显示的是镜像下载到本地后，展开的大小，准确说，是展开后的各层所占空间的总和，因为镜像到本地后，查看空间的时候，更关心的是本地磁盘空间占用的大小。另外一个需要注意的问题是，
> docker image ls
> 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。 </font>

```shell
root@test:~# docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              2                   1                   116MB               116MB (99%)
Containers          2                   0                   0B                  0B
Local Volumes       0                   0                   0B                  0B
Build Cache         0                   0                   0B                  0B
# 使用此命令可以便捷的查看镜像、容器、数据卷所占用的空间。
```



#### 虚悬镜像

```shell
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>				<none>				00285df0df87		5 days ago			342 MB
# 使用docker image ls命令查看时，可能会出现上面的结果。这个镜像既没有仓库名，也没有标签，均为<none>。
# 这个镜像原本是有镜像名和标签的，随着官方镜像维护，发布新版本后，重新执行"docker pull 镜像名"命令时，这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了<none>。除了docker pull命令可能导致这种情况，docker build命令也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取走，从而出现仓库名、标签均为<none>的镜像。这类无标签镜像也被称为虚悬镜像(dangling image)

root@test:~# docker image ls -f dangling=true
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>				<none>				00285df0df87		5 days ago			342 MB
# 使用上面命令可以显示出虚悬镜像。一般虚悬镜像已经没有了存在的价值，可以随意删除。

root@test:~# docker image prune 
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
# 删除所有的虚悬镜像
```



#### 中间层镜像

> <font color=red>为了加速镜像构建、重复利用资源，Docker 会利用中间层镜像。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。默认的docker image ls列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像的话，需要加-a参数。</font>

```shell
root@test:~# docker image ls -a
# 这样会看到很多无标签的镜像，与之前的虚悬镜像不同，这些无标签的镜像很多都是中间层镜像，是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。实际上，这些镜像也没必要删除，因为之前说过，相同的层只会存一遍，而这些镜像是别的镜像的依赖，因此并不会因为它们被列出来而多存了一份，无论如何你也会需要它们。只要删除那些依赖它们的镜像后，这些依赖的中间层镜像也会被连带删除。
```



#### 列出部分镜像

```shell
root@test:~# docker image ls ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               4a689991aa24        4 weeks ago         116MB
# 根据仓库名列出镜像

root@test:~# docker image ls ubuntu:16.04
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               4a689991aa24        4 weeks ago         116MB
# 指定仓库名和标签，列出特定的某个镜像

root@test:~# docker image ls -f since=ubuntu:16.04
# 过滤器参数--filter，或者简写为-f。查询在ubuntu:16.04之后建立的镜像。

root@test:~# docker image ls -f before=ubuntu:16.04
# 将since改为before，可以查询在ubuntu:16.04之前建立的镜像。

root@test:~# docker image ls -f label=com.example.version=0.1
# 如果镜像构建时，定义了LABEL，还可以通过LABEL来过滤。
```



#### 以特定格式显示

```shell
root@test:~# docker image ls -q
4a689991aa24
4ab4c602aa5e
# 默认情况下，docker image ls会输出一个完整的表格，但是我们并非所有时候都会需要这些内容。比如，刚才删除虚悬镜像的时候，我们需要利用docker image ls把所有的虚悬镜像的 ID 列出来，然后才可以交给docker image rm命令作为参数来删除指定的这些镜像，这个时候就用到了-q参数。--filter配合-q产生出指定范围的 ID 列表，然后送给另一个docker命令作为参数，从而针对这组实体成批的进行某种操作的做法在 Docker 命令行使用过程中非常常见

root@test:~# docker image ls --format "{{.ID}}: {{.Repository}}"
4a689991aa24: ubuntu
4ab4c602aa5e: hello-world
# 我们可能只是对表格的结构不满意，希望自己组织列；或者不希望有标题，这样方便其它程序解析结果等，这就用到了 Go 的模板语法。上面的命令会直接列出镜像结果，并且只包含镜像ID和仓库名

root@test:~# docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY          TAG
4a689991aa24        ubuntu              16.04
4ab4c602aa5e        hello-world         latest
# 以表格等距显示，并且有标题行，和默认一样，不过列为自定义
```



### 删除本地镜像

```shell
* 命令格式
docker image rm [选项] <镜像1> [<镜像2> ...]
# 删除镜像

docker rm <containerID>
# 删除容器
```



#### 用 ID、镜像名、摘要删除镜像

```shell
* 使用ID删除镜像
root@test:~# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               4a689991aa24        4 weeks ago         116MB
hello-world         latest              4ab4c602aa5e        2 months ago        1.84kB
root@test:~# docker image rm 4ab
Error response from daemon: conflict: unable to delete 4ab4c602aa5e (must be forced) - image is being used by stopped container 641b04116b0a
# 我们可以通过镜像的ID号删除镜像，并且ID号可以只使用前几位。但上面删除时有报错，说明有容器在依赖此镜像，需要先停止这个容器才能删除镜像。
root@test:~# docker rm 641b04116b0a
641b04116b0a
# 删除容器
root@test:~# docker image rm 4ab
Error response from daemon: conflict: unable to delete 4ab4c602aa5e (must be forced) - image is being used by stopped container 41a0dff7c649
# 再次删除镜像时依然有此报错，但这次容器的ID变了
root@test:~# docker rm 41a0dff7c649
41a0dff7c649
root@test:~# docker image rm 4ab   
Untagged: hello-world:latest
Untagged: hello-world@sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Deleted: sha256:4ab4c602aa5eed5528a6620ff18a1dc4faef0e1ab3a5eddeddb410714478c67f
Deleted: sha256:428c97da766c4c13b19088a471de6b622b038f3ae8efa10ec5a37d6d31a2df0b
# 再次删除容器后就可以删除镜像了
# 我们可以用镜像的完整 ID，也称为长 ID，来删除镜像。使用脚本的时候可能会用长 ID，但是人工输入就太累了，所以更多的时候是用短 ID来删除镜像。docker image ls默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。

* 使用镜像名删除镜像
root@test:~# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               4a689991aa24        4 weeks ago         116MB
hello-world         latest              4ab4c602aa5e        2 months ago        1.84kB
root@test:~# docker image rm hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Deleted: sha256:4ab4c602aa5eed5528a6620ff18a1dc4faef0e1ab3a5eddeddb410714478c67f
Deleted: sha256:428c97da766c4c13b19088a471de6b622b038f3ae8efa10ec5a37d6d31a2df0b
# 用镜像名，也就是<仓库名>:<标签>，来删除镜像。

* 使用镜像摘要删除镜像
root@test:~# docker image ls --digests 
REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
ubuntu              16.04               sha256:76702ec53c5e7771ba3f2c4f6152c3796c142af2b3cb1a02fce66c697db24f12   4a689991aa24        4 weeks ago         116MB
hello-world         latest              sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788   4ab4c602aa5e        2 months ago        1.84kB
# 查询镜像摘要
root@test:~# docker image rm node@sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Error: No such image: node@sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
# 按镜像摘要删除镜像，但测试失败，提示没有这个镜像

* Untagged 和 Deleted
# 如果观察上面这几个命令的运行输出信息的话，你会注意到删除行为分为两类，一类是Untagged，另一类是Deleted。我们之前介绍过，镜像的唯一标识是其 ID 和摘要，而一个镜像可以有多个标签。
# 因此当我们使用上面命令删除镜像的时候，实际上是在要求删除某个标签的镜像。所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的Untagged的信息。因为一个镜像可以对应多个标签，因此当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像，如果是这种情况，那么Delete行为就不会发生。所以并非所有的docker image rm都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。 
# 当该镜像所有的标签都被取消了，该镜像很可能会失去了存在的意义，因此会触发删除行为。镜像是多层存储结构，因此在删除的时候也是从上层向基础层方向依次进行判断删除。镜像的多层结构让镜像复用变动非常容易，因此很有可能某个其它镜像正依赖于当前镜像的某一层。这种情况，依旧不会触发删除该层的行为。直到没有任何层依赖当前层时，才会真实的删除当前层。这就是为什么，有时候会奇怪，为什么明明没有别的标签指向这个镜像，但是它还是存在的原因，也是为什么有时候会发现所删除的层数和自己docker pull看到的层数不一样的原因。
# 除了镜像依赖以外，还需要注意的是容器对镜像的依赖。如果有用这个镜像启动的容器存在(即使容器没有运行)，那么同样不可以删除这个镜像。之前讲过，容器是以镜像为基础，再加一层容器存储层，组成这样的多层存储结构去运行的。因此该镜像如果被这个容器所依赖，那么删除必然会导致故障。如果这些是容器不需要的，应该先将它们删除，然后再来删除镜像。
```



#### 用 docker image ls 命令来配合

```shell
docker image rm $(docker image ls -q redis)
# 删除所有仓库名为redis的镜像。$()表示输入结果，同``。
docker image rm $(docker image ls -q -f before=mongo:3.2)
# 删除所有在mongo:3.2之前创建的镜像
```



#### CentOS/RHEL 的用户需要注意的事项

> 在 Ubuntu/Debian 上有UnionFS可以使用，如aufs或者overlay2，而 CentOS 和 RHEL的内核中没有相关驱动。因此对于这类系统，一般使用devicemapper驱动利用 LVM 的一些机制来模拟分层存储。这样的做法除了性能比较差外，稳定性一般也不好，而且配置相对复杂。Docker 安装在 CentOS/RHEL 上后，会默认选择devicemapper，但是为了简化配置，其devicemapper是跑在一个稀疏文件模拟的块设备上，也被称为loop-lvm。这样的选择是因为不需要额外配置就可以运行 Docker，这是自动配置唯一能做到的事情。但是loop-lvm的做法非常不好，其稳定性、性能更差，无论是日志还是docker info中都会看到警告信息。官方文档有明确的文章讲解了如何配置块设备给devicemapper驱动做存储层的做法，这类做法也被称为配置direct-lvm。
> 除了前面说到的问题外，devicemapper+loop-lvm还有一个缺陷，因为它是稀疏文件，所以它会不断增长。用户在使用过程中会注意到/var/lib/docker/devicemapper/devicemapper/data不断增长，而且无法控制。很多人会希望删除镜像或者可以解决这个问题，结果发现效果并不明显。原因就是这个稀疏文件的空间释放后基本不进行垃圾回收的问题。因此往往会出现即使删除了文件内容，空间却无法回收，随着使用这个稀疏文件一直在不断增长。
> 所以对于 CentOS/RHEL 的用户来说，在没有办法使用UnionFS的情况下，一定要配置direct-lvm给devicemapper，无论是为了性能、稳定性还是空间利用率。
> 或许有人注意到了 CentOS 7 中存在被 backports 回来的overlay驱动，不过 CentOS 里的这个驱动达不到生产环境使用的稳定程度，所以不推荐使用。



### 利用 commit 理解镜像构成

> 注意：docker commit命令除了学习之外，还有一些特殊的应用场合，比如被入侵后保存现场等。但是，不要使用docker commit定制镜像，定制镜像应该使用Dockerfile来完成。
>
> 镜像是容器的基础，每次执行docker run的时候都会指定哪个镜像作为容器运行的基础。在之前的例子中，我们所使用的都是来自于 Docker Hub 的镜像。直接使用这些镜像是可以满足一定的需求，而当这些镜像无法直接满足需求时，我们就需要定制这些镜像。
> 回顾一下之前我们学到的知识，镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。
> 现在让我们以定制一个 Web 服务器为例子，来讲解镜像是如何构建的。

```shell
root@ruopu:~# docker pull nginx
# 下载nginx镜像
root@ruopu:~# docker run --name webserver -d -p 80:80 nginx
4fc2b542c0dee4c38cbc1f93c1dd3191f2742f0a58e7724770528e93895b923d
# 用nginx镜像启动一个容器，命名为webserver，并且映射了 80 端口，-d表示让容器在后台运行，这样我们就可以用浏览器去访问这个nginx服务器了。
root@ruopu:~# curl localhost
# 用本机访问或使用浏览器访问IP地址。
root@ruopu:~# docker exec -it webserver bash
# 使用docker exec命令进入容器，-i：交互式操作，-t：终端。进入容器使用bash执行一些命令
root@4fc2b542c0de:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
# 修改默认网页内容
root@4fc2b542c0de:/# exit
# 退出
exit
root@ruopu:~# curl localhost
root@ruopu:~# docker diff webserver 
C /var
C /var/cache
C /var/cache/nginx
A /var/cache/nginx/scgi_temp
A /var/cache/nginx/uwsgi_temp
A /var/cache/nginx/client_temp
A /var/cache/nginx/fastcgi_temp
A /var/cache/nginx/proxy_temp
C /usr
C /usr/share
C /usr/share/nginx
C /usr/share/nginx/html
C /usr/share/nginx/html/index.html
C /run
A /run/nginx.pid
C /root
A /root/.bash_history
# 这里修改了容器的文件，也就是改动了容器的存储层。我们可以通过docker diff命令看到具体的改动。
============================================================================================
* 当我们运行一个容器的时候(如果不使用卷的话)，我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个docker commit命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。
* 语法
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
============================================================================================
root@ruopu:~# docker commit --author "ruopu <ruopu@ccgoldenet.com>" --message "修改了默认网页" webserver nginx:v2
sha256:c3ceec8361f57f2dd49657b42ea808b9062a7a73f6327e5675be3ad8c704055a
# --author是指定修改的作者，--message是记录本次修改的内容。这点和git版本控制相似，不过这里这些信息可以省略留空。
root@ruopu:~# docker image ls nginx
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
nginx               v2                  c3ceec8361f5        About a minute ago   109MB
nginx               latest              e81eb098537d        4 days ago           109MB
# 可以在docker image ls中看到这个新定制的镜像
root@ruopu:~# docker history nginx:v2 
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
c3ceec8361f5        2 minutes ago       nginx -g daemon off;                            97B                 修改了默认网页
e81eb098537d        4 days ago          /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
<missing>           4 days ago          /bin/sh -c #(nop)  STOPSIGNAL [SIGTERM]         0B                  
<missing>           4 days ago          /bin/sh -c #(nop)  EXPOSE 80/tcp                0B                  
<missing>           4 days ago          /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B                 
<missing>           4 days ago          /bin/sh -c set -x  && apt-get update  && apt…   53.8MB              
<missing>           4 days ago          /bin/sh -c #(nop)  ENV NJS_VERSION=1.15.6.0.…   0B                  
<missing>           4 days ago          /bin/sh -c #(nop)  ENV NGINX_VERSION=1.15.6-…   0B                  
<missing>           4 days ago          /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B                  
<missing>           5 days ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           5 days ago          /bin/sh -c #(nop) ADD file:dab9baf938799c515…   55.3MB 
# 用docker history具体查看镜像内的历史记录，如果比较nginx:latest的历史记录，我们会发现新增了我们刚刚提交的这一层。
root@ruopu:~# docker run --name web2 -d -p 81:80 nginx:v2
54b6e4618182e413ea4a8873228f743b535667a2421035c46271514ddea7cad3
# 使用新定制的镜像运行一个新容器，我们命名为新的服务为web2，并且映射到81端口
root@ruopu:~# curl localhost:81
# 访问新容器，也可以访问IP:81
```



#### 慎用docker commit

> 实际环境一般不会使用上面的方法定制新的镜像。如果仔细观察之前的docker diff webserver的结果，你会发现除了真正想要修改的/usr/share/nginx/html/index.html文件外，由于命令的执行，还有很多文件被改动或添加了。如果进行更多的变动，会导致镜像非常臃肿。
>
> 此外，使用docker commit意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为黑箱镜像，换句话说，就是除了制作镜像的人知道执行过什么命令，怎么生成的镜像，别人根本无从得知。虽然docker diff或许可以告诉得到一些线索，但是远远不到可以确保生成一致镜像的地步。
>
> 而且，回顾之前提及的镜像所使用的分层存储的概念，除当前层外，之前的每一层都是不会发生改变的，换句话说，任何修改的结果仅仅是在当前层进行标记、添加、修改，而不会改动上一层。如果使用docker commit制作镜像，以及后期修改的话，每一次修改都会让镜像更加臃肿一次，所删除的上一层的东西并不会丢失，会一直如影随形的跟着这个镜像，即使根本无法访问到。这会让镜像更加臃肿。



### 使用 Dockerfile 定制镜像

> 从刚才的docker commit的学习中，我们可以了解到，镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。
> Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

```shell
root@ruopu:~# mkdir mynginx
root@ruopu:~# cd mynginx/
root@ruopu:~/mynginx# touch Dockerfile
# 在一个空白目录中，建立一个文本文件，并命名为Dockerfile
root@ruopu:~/mynginx# vim Dockerfile 
    FROM nginx
    RUN echo '<h1>Hello,Docker!!!</h1>' > /usr/share/nginx/html/index.html
# 这个 Dockerfile 很简单，一共就两行。涉及到了两条指令，FROM和RUN
```



#### FROM 指定基础镜像

> <font color=red>所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个nginx镜像的容器，再进行修改一样，基础镜像是必须指定的。而FROM就是指定基础镜像，因此一个Dockerfile中FROM是必备的指令，并且必须是第一条指令。</font>
>
> Docker Store中有服务类镜像与基础操作系统镜像可供下载。除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为scratch。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。
>
> ```shell
> FROM scratch
> ...
> ```
>
> 如果你以scratch为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。
>
> 不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如swarm、coreos/etcd。对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接FROM scratch会让镜像体积更加小巧。使用 Go 语言开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go是特别适合容器微服务架构的语言的原因之一。



#### RUN 执行命令

> RUN指令是用来执行命令行命令的。由于命令行的强大能力，RUN指令在定制镜像时是最常用的指令之一。

```shell
============================================================================================
* 格式一
shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的RUN指令就是这种格式。
	RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html

* 格式二
exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。
# Dockerfile 中每一个指令都会建立一层，RUN也不例外。每一个RUN的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，commit这一层的修改，构成新的镜像。也就是说，每一个RUN命令都会创建一层镜像，这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。
# Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过127 层。
============================================================================================
FROM debian:jessie
RUN buildDeps='gcc libc6-dev make' \	
#设置变量这一步很重要，如果不定义，下面的yum安装时将无法进行，yum会将其后面的命令都当作安装的包
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
# 要写成上面的样子，不能每行前面都加一个RUN命令。所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个RUN来一一对应不同的命令，而是仅仅使用一个RUN指令，并使用&&将各个所需命令串联起来。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。
# /var/lib/apt/lists/下是apt的缓存文件。apt-get purge 命令可以将包和配置文件一起删除，--auto-remove可以自动删除所有未使用的包。
# make命令的-C选项：指定读取makefile的目录。如果有多个“-C”参数，make的解释是后面的路径以前面的作为相对路径，并以最后的目录作为被指定目录。如：“make –C ~hchen/test –C prog”等价于“make –C ~hchen/test/prog”。
# 并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加\的命令换行方式，以及行首#进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，,这是一个比较好的习惯。
# 此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了apt缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。
```



#### 构建镜像

```shell
============================================================================================
* 语法
docker build [选项] <上下文路径/URL/->
============================================================================================
root@ruopu:~# cd /root/mynginx/
root@ruopu:~/mynginx# docker build -t nginx:v3 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> e81eb098537d
Step 2/2 : RUN echo '<h1>Hello,Docker!!!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in fdbbe8c89936
Removing intermediate container fdbbe8c89936
 ---> b6eed855d34f
Successfully built b6eed855d34f
Successfully tagged nginx:v3
# 一定不要少了命令中最后的点，下面会有对这个点的说明
#-t nginx:v3表示要构建的镜像的名称。从命令的输出结果中，我们可以清晰的看到镜像的构建过程。在Step 2中，如同我们之前所说的那样，RUN指令启动了一个容器fdbbe8c89936，执行了所要求的命令，随后删除了所用到的这个容器fdbbe8c89936，并最后提交了这一层b6eed855d34f。
```



#### 镜像构建上下文(Context)

> 如果注意，会看到docker build命令最后有一个"."。"."表示当前目录。而Dockerfile就在当前目录，因此不少初学者以为这个路径是在指定Dockerfile所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定上下文路径。那么什么是上下文呢?
> 首先我们要理解docker build的工作原理。Docker 在运行时分为 Docker 引擎(也就是服务端守护进程)和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API，而如docker命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种docker功能，但实际上，一切都是使用的远程调用形式在服务端(Docker 引擎)完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。
> 当我们进行镜像构建的时候，并非所有定制都会通过RUN指令完成，经常会需要将一些本地文件复制进镜像，比如通过COPY指令、ADD指令等。而docker build命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。
>
> 如果在Dockerfile中这么写:
>
> ```COPY ./package.json /app/```
>
> 这并不是要复制执行docker build命令所在的目录下的package.json，也不是复制Dockerfile所在目录下的package.json，而是复制上下文(context) 目录下的package.json。
> 因此，<font color=red>COPY这类指令中的源文件的路径都是相对路径。</font>这也是初学者经常会问的为什么COPY ../package.json /app或者COPY /opt/xxxx /app无法工作的原因，因为这些路径已经超出了上下文的范围，Docker 引擎无法获得这些位置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。
> 现在就可以理解刚才的命令**docker build -t nginx:v3 .** 中的这个 **.** ，实际上是在指定上下文的目录，docker build命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像。
>
> 如果观察docker buile的输出，我们其实已经看到了这个发送上下文的过程，如下：
>
> ```shell
> root@ruopu:~/mynginx# docker build -t nginx:v3 .
> Sending build context to Docker daemon  2.048kB
> ...
> ```
>
> <font color=red>理解构建上下文对于镜像构建是很重要的，避免犯一些不应该的错误。比如有些初学者在发现COPY /opt/xxxx /app不工作后，于是干脆将Dockerfile放到了硬盘根目录去构建，结果发现docker build执行后，在发送一个几十 GB 的东西，极为缓慢而且很容易构建失败。那是因为这种做法是在让docker build打包整个硬盘，这显然是使用错误。</font>
> <font color=red>一般来说，应该会将Dockerfile置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用.gitignore一样的语法写一个.dockerignore，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。</font>
>
> 那么为什么会有人误以为"."是指定Dockerfile所在目录呢？这是因为在默认情况下，如果不额外指定Dockerfile的话，会将上下文目录下的名为Dockerfile的文件作为Dockerfile。
>
> <font color=red>这只是默认行为，实际上Dockerfile的文件名并不要求必须为Dockerfile，而且并不要求必须位于上下文目录中，比如可以用-f ../Dockerfile.php 参数指定某个文件作为Dockerfile</font>



#### 其它 docker build 的用法

##### 直接用 Git repo 进行构建

```shell
root@ruopu:~/mynginx# $ docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14
docker build https://github.com/twang2218/gitlab-ce-zh.git\#:8.14
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM gitlab/gitlab-ce:8.14.0-ce.0
8.14.0-ce.0: Pulling from gitlab/gitlab-ce
aed15891ba52: Already exists
773ae8583d14: Already exists
...
# 这行命令指定了构建所需的 Git repo，并且指定默认的master分支，构建目录为/8.14/，然后 Docker 就会自己去git clone这个项目、切换到指定分支、并进入到指定目录后开始构建。
```



##### 用给定的 tar 压缩包构建

```shell
docker build http://server/context.tar.gz
# 如果所给出的 URL 不是个 Git repo，而是个tar压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。
```



##### 从标准输入中读取 Dockerfile 进行构建

```shell
docker build - < Dockerfile
或
cat Dockerfile | docker build -
# 如果标准输入传入的是文本文件，则将其视为Dockerfile，并开始构建。这种形式由于直接从标准输入中读取 Dockerfile 的内容，它没有上下文，因此不可以像其他方法那样可以将本地文件COPY进镜像之类的事情。
```



##### 从标准输入中读取上下文压缩包进行构建

```shell
docker build - < context.tar.gz
# 如果发现标准输入的文件格式是gzip、bzip2以及xz的话，将会使其为上下文压缩包，直接将其展开，将里面视为上下文，并开始构建。
```



### Dockerfile 指令详解

#### COPY 复制文件

```shell
* 格式:
    COPY <源路径>... <目标路径>
    COPY ["<源路径1>",... "<目标路径>"]
# 和RUN指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。
# COPY指令将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置。比如:
	COPY package.json /usr/src/app/
# <源路径>可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的filepath.Match规则，如:
    COPY hom* /mydir/
    COPY hom?.txt /mydir/
# <目标路径>可以是容器内的绝对路径，也可以是相对于工作目录的相对路径(工作目录可以用WORKDIR指令来指定)。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。
# 此外，还需要注意一点，使用COPY指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用Git进行管理的时候。
```



#### ADD 更高级的复制文件

```shell
# ADD指令和COPY的格式和性质基本一致。但是在COPY基础上增加了一些功能。
# 比如<源路径>可以是一个URL，这种情况下，Docker 引擎会试图去下载这个链接的文件放到<目标路径>去。下载后的文件权限自动设置为600，如果这并不是想要的权限，那么还需要增加额外的一层 RUN 进行权限调整。另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层 RUN 指令进行解压缩。所以不如直接使用RUN指令，然后使用wget或者curl工具下载，处理权限、解压缩，然后清理无用文件更合理。因此，这个功能其实并不实用，而且不推荐使用。
# 如果<源路径>为一个tar压缩文件的话，压缩格式为gzip,bzip2以及xz的情况下，ADD指令将会自动解压缩这个压缩文件到<目标路径>去。
# 在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像ubuntu中:
    FROM scratch
    ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
    ...
# 但在某些情况下，如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用ADD命令了。
# 在 Docker 官方的 Dockerfile 最佳实践文档中要求，尽可能的使用COPY，因为COPY的语义很明确，就是复制文件而已，而ADD则包含了更复杂的功能，其行为也不一定很清晰。最适合使用ADD的场合，就是所提及的需要自动解压缩的场合。
# 另外需要注意的是，ADD指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。
# 因此在COPY和ADD指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用COPY指令，仅在需要自动解压缩的场合使用ADD。
```



#### CMD 容器启动命令

```shell
# CMD 指令的格式和RUN相似，也是两种格式:
	shell 格式：CMD <命令>
	exec 格式：CMD ["可执行文件", "参数1", "参数2"...]
	参数列表格式：CMD ["参数1", "参数2"...]。在指定了ENTRYPOINT指令后，用CMD指定具体的参数。
# 之前介绍容器的时候曾经说过，Docker不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD指令就是用于指定默认的容器主进程的启动命令的。
# 在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如，ubuntu镜像默认的CMD是/bin/bash，如果我们直接运行docker run -it ubuntu的话，会直接进入bash。我们也可以在运行时指定运行别的命令，如docker run -it ubuntu cat /etc/os-release。这就是用cat /etc/os-release命令替换了默认的/bin/bash命令了，输出了系统版本信息。
# 在指令格式上，一般推荐使用exec格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号"，而不要使用单引号。
# 如果使用shell格式的话，实际的命令会被包装为sh -c的参数的形式进行执行。-c选项表示命令从-c后的字符串读取。比如:
	CMD echo $HOME
# 在实际执行中，会将其变更为：
	CMD [ "sh", "-c", "echo $HOME" ]
# 这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。
# 提到CMD就不得不提容器中应用在前台执行和后台执行的问题。这是初学者常出现的一个混淆。
# Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。
# 一些初学者将CMD写为:
	CMD service nginx start
# 然后发现容器执行后就立即退出了。甚至在容器内去使用systemctl命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。
# 对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。
# 而使用service nginx start命令，则是希望 upstart 使用后台守护进程形式启动nginx服务。而刚才说了CMD service nginx start会被理解为CMD [ "sh", "-c", "service nginx start"] ，因此主进程实际上是sh。那么当service nginx start命令结束后，sh也就结束了，sh作为主进程退出了，自然就会令容器退出。
# 正确的做法是直接执行nginx可执行文件，并且要求以前台形式运行。比如:
	CMD ["nginx", "-g", "daemon off;"]
```



#### ENTRYPOINT 入口点

```shell
# ENTRYPOINT 的格式和 RUN 指令格式一样，分为exec格式和shell格式。
# ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT在运行时也可以替代，不过比CMD要略显繁琐，需要通过docker run的参数--entrypoint来指定。
# 当指定了ENTRYPOINT后，CMD的含义就发生了改变，不再是直接的运行其命令，而是将CMD的内容作为参数传给ENTRYPOINT指令，换句话说实际执行时，将变为:
	<ENTRYPOINT> "<CMD>"
# 那么有了CMD后,为什么还要有ENTRYPOINT呢？这种<ENTRYPOINT> "<CMD>"有什么好处么？让我们来看几个场景。

* 场景一:让镜像变成像命令一样使用
# 假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用CMD来实现:
    FROM ubuntu:16.04
    RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
    CMD [ "curl", "-s", "http://ip.cn" ]
# 假如我们使用docker build -t myip . 来构建镜像的话,如果我们需要查询当前公网 IP,只需要执行:
	$ docker run myip
	当前 IP:61.148.226.66 来自:北京市 联通
# 嗯，这么看起来好像可以直接把镜像当做命令使用了，不过命令总有参数，如果我们希望加参数呢？比如从上面的CMD中可以看到实质的命令是curl，那么如果我们希望显示 HTTP头信息，就需要加上-i参数。那么我们可以直接加-i参数给docker run myip么?
    $ docker run myip -i
    docker: Error response from daemon: invalid header field value "oci runtime error: con
    tainer_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable
    file not found in $PATH\"\n".
# 我们可以看到可执行文件找不到的报错，executable file not found。之前我们说过，跟在镜像名后面的是command，运行时会替换CMD的默认值。因此这里的-i替换了原来的CMD，而不是添加在原来的curl -s http://ip.cn后面。而-i根本不是命令，所以自然找不到。
# 那么如果我们希望加入-i这个参数，我们就必须重新完整的输入这个命令：
	$ docker run myip curl -s http://ip.cn -i
# 这显然不是很好的解决方案，而使用ENTRYPOINT就可以解决这个问题。现在我们重新用ENTRYPOINT来实现这个镜像:
    FROM ubuntu:16.04
    RUN apt-get update \
        && apt-get install -y curl \
        && rm -rf /var/lib/apt/lists/*
    ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
# 这次我们再来尝试直接使用docker run myip -i：
    $ docker run myip
    当前 IP:61.148.226.66 来自:北京市 联通
    
    $ docker run myip -i
    HTTP/1.1 200 OK
    Server: nginx/1.8.0
    Date: Tue, 22 Nov 2016 05:12:40 GMT
    Content-Type: text/html; charset=UTF-8
    Vary: Accept-Encoding
    X-Powered-By: PHP/5.6.24-1~dotdeb+7.1
    X-Cache: MISS from cache-2
    X-Cache-Lookup: MISS from cache-2:80
    X-Cache: MISS from proxy-2_6
    Transfer-Encoding: chunked
    Via: 1.1 cache-2:80, 1.1 proxy-2_6:8006
    Connection: keep-alive
    
	当前 IP:61.148.226.66 来自:北京市 联通
# 可以看到，这次成功了。这是因为当存在ENTRYPOINT后，CMD的内容将会作为参数传给ENTRYPOINT，而这里-i就是新的CMD，因此会作为参数传给curl，从而达到了我们预期的效果。

* 场景二:应用运行前的准备工作

# 启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。
# 比如mysql类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要在最终的mysql 服务器运行之前解决。
# 此外，可能希望避免使用root用户去启动服务，从而提高安全性，而在启动服务前还需要以root身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。或者除了服务外，其它命令依旧可以使用 root身份执行，方便调试等。
# 这些准备工作是和容器CMD无关的，无论CMD为什么，都需要事先进行一个预处理的工作。这种情况下，可以写一个脚本，然后放入ENTRYPOINT中去执行，而这个脚本会将接到的参数(也就是<CMD>)作为命令，在脚本最后执行。比如官方镜像redis中就是这么做的：
    FROM alpine:3.4
    ...
    RUN addgroup -S redis && adduser -S -G redis redis
    ...
    ENTRYPOINT ["docker-entrypoint.sh"]
    EXPOSE 6379
    CMD [ "redis-server" ]
# 可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了ENTRYPOINT为docker-entrypoint.sh脚本。
    #!/bin/sh
    ...
    # allow the container to be started with `--user`
    if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
    chown -R redis .
    exec su-exec redis "$0" "$@"
    fi
    exec "$@"
# 该脚本的内容就是根据CMD的内容来判断，如果是redis-server的话，则切换到redis用户身份启动服务器，否则依旧使用root身份执行。比如：
    $ docker run -it redis id
    uid=0(root) gid=0(root) groups=0(root)
```



#### ENV 设置环境变量

```shell
* 格式有两种:
    ENV <key> <value>
    ENV <key1>=<value1> <key2>=<value2>...
# 这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如RUN，还是运行时的应用，都可以直接使用这里定义的环境变量。
	ENV VERSION=1.0 DEBUG=on \
		NAME="Happy Feet"
# 这个例子中演示了如何换行，以及对含有空格的值用双引号括起来的办法，这和 Shell 下的行为是一致的。
# 定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方node镜像Dockerfile中，就有类似这样的代码:
    ENV NODE_VERSION 7.2.0
    RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.ta
    r.xz" \
    && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=
    1 \
    && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs
# 在这里先定义了环境变量NODE_VERSION，其后的RUN这层里，多次使用$NODE_VERSION来进行操作定制。可以看到，将来升级镜像构建版本的时候，只需要更新7.2.0即可，Dockerfile构建维护变得更轻松了。
# 下列指令可以支持环境变量展开：
	ADD、COPY、ENV、EXPOSE、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBU、ILD
# 可以从这个指令列表里感觉到，环境变量可以使用的地方很多，很强大。通过环境变量，我们可以让一份Dockerfile制作更多的镜像，只需使用不同的环境变量即可。
```



#### ARG 构建参数

```shell
* 格式: ARG <参数名>[=<默认值>]
# 构建参数和ENV的效果一样，都是设置环境变量。所不同的是，ARG所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用ARG保存密码之类的信息，因为docker history还是可以看到所有值的。
# Dockerfile中的ARG指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令docker build中用--build-arg <参数名>=<值>来覆盖。
# 在 1.13 之前的版本，要求--build-arg中的参数名，必须在Dockerfile中用ARG定义过了，换句话说，就是--build-arg指定的参数，必须在Dockerfile中使用了。如果对应参数没有被使用，则会报错退出构建。从 1.13 开始，这种严格的限制被放开，不再报错退出，而是显示警告信息，并继续构建。这对于使用 CI 系统，用同样的构建流程构建不同的Dockerfile的时候比较有帮助，避免构建命令必须根据每个 Dockerfile 的内容修改。
```



#### VOLUME 定义匿名卷

```shell
* 格式为:
    VOLUME ["<路径1>", "<路径2>"...]
    VOLUME <路径>
# 之前我们说过，容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在Dockerfile中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。
	VOLUME /data
# 这里的/data目录就会在运行时自动挂载为匿名卷，任何向/data中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行时可以覆盖这个挂载设置。比如:
	docker run -d -v mydata:/data xxxx
# 在这行命令中，就使用了mydata这个命名卷挂载到了/data这个位置，替代了Dockerfile中定义的匿名卷的挂载配置。
```



#### EXPOSE 声明端口

```shell
* 格式为 EXPOSE <端口1> [<端口2>...]
# EXPOSE指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是docker run -P时，会自动随机映射EXPOSE的端口。
# 此外，在早期 Docker 版本中还有一个特殊的用处。以前所有容器都运行于默认桥接网络中，因此所有容器互相之间都可以直接访问，这样存在一定的安全性问题。于是有了一个 Docker引擎参数--icc=false，当指定该参数后，容器间将默认无法互访，除非互相间使用了--links参数的容器才可以互通，并且只有镜像中EXPOSE所声明的端口才可以被访问。这个--icc=false的用法，在引入了docker network后已经基本不用了，通过自定义网络可以很轻松的实现容器间的互联与隔离。
# 要将EXPOSE和在运行时使用-p <宿主端口>:<容器端口>区分开来。-p是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而EXPOSE仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。
```



#### WORKDIR 指定工作目录

```shell
* 格式为 WORKDIR <工作目录路径>
# 使用WORKDIR指令可用来指定工作目录(或者称为当前目录)，以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR会帮你建立目录。
# 之前提到一些初学者常犯的错误是把Dockerfile等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误:
	RUN cd /app
	RUN echo "hello" > world.txt
# 如果将这个Dockerfile进行构建镜像运行后，会发现找不到/app/world.txt文件，或者其内容不是hello。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 Dockerfile 中，这两行RUN命令的执行环境根本不同，是两个完全不同的容器。这就是对 Dockerfile 构建分层存储的概念不了解所导致的错误。
# 之前说过每一个RUN都会启动一个容器、执行命令、然后提交存储层文件变更。第一层RUN cd /app的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器完全没关系，自然不可能继承前一层构建过程中的内存变化。
# 因此如果需要改变以后各层的工作目录的位置，那么应该使用WORKDIR指令。
```



#### USER 指定当前用户

```shell
* 格式: USER <用户名>
# USER指令和WORKDIR相似，都是改变环境状态并影响以后的层。WORKDIR是改变工作目录，USER则是改变之后层的执行RUN，CMD以及ENTRYPOINT这类命令的身份。
# 当然，和WORKDIR一样，USER只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。
    RUN groupadd -r redis && useradd -r -g redis redis
    USER redis
    RUN [ "redis-server" ]
# 如果以root执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程，不要使用su或者sudo，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用gosu。

   RUN groupadd -r redis && useradd -r -g redis redis
 # 建立 redis 用户，并使用 gosu 换另一个用户执行命令
    RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.7/
    gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true  
# 下载 gosu
  CMD [ "exec", "gosu", "redis", "redis-server" ]
# 设置 CMD，并以另外的用户执行
  
```



#### HEALTHCHECK 健康检查

```shell
* 格式:
	HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
	HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令
# HEALTHCHECK指令是告诉 Docker 应该如何进行判断容器的状态是否正常，这是 Docker 1.12引入的新指令。
# 在没有HEALTHCHECK指令前，Docker 引擎只可以通过容器内主进程是否退出来判断容器是否状态异常。很多情况下这没问题，但是如果程序进入死锁状态，或者死循环状态，应用进程并不退出，但是该容器已经无法提供服务了。在 1.12 以前，Docker 不会检测到容器的这种状态，从而不会重新调度，导致可能会有部分容器已经无法提供服务了却还在接受用户请求。
# 而自 1.12 之后，Docker 提供了HEALTHCHECK指令，通过该指令指定一行命令，用这行命令来判断容器主进程的服务状态是否还正常，从而比较真实的反应容器实际状态。
# 当在一个镜像指定了HEALTHCHECK指令后，用其启动容器，初始状态会为starting，在HEALTHCHECK指令检查成功后变为healthy，如果连续一定次数失败，则会变为unhealthy。
# HEALTHCHECK 支持下列选项：
--interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
--timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
--retries=<次数>：当连续失败指定次数后，则将容器状态视为unhealthy，默认 3次。
# 和CMD，ENTRYPOINT一样，HEALTHCHECK只可以出现一次，如果写了多个，只有最后一个生效。
# 在HEALTHCHECK [选项] CMD后面的命令，格式和ENTRYPOINT一样，分为shell格式，和exec格式。命令的返回值决定了该次健康检查的成功与否，0：成功；1：失败；2：保留，不要使用这个值。
# 假设我们有个镜像是个最简单的 Web 服务，我们希望增加健康检查来判断其 Web 服务是否在正常工作，我们可以用curl来帮助判断，其Dockerfile的HEALTHCHECK可以这么写：
    FROM nginx
    RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
    HEALTHCHECK --interval=5s --timeout=3s \
    CMD curl -fs http://localhost/ || exit 1
# 这里我们设置了每 5 秒检查一次(这里为了试验所以间隔非常短，实际应该相对较长)，如果健康检查命令超过 3 秒没响应就视为失败，并且使用curl -fs http://localhost/ || exit 1 作为健康检查命令。
# 使用docker build来构建这个镜像：
	$ docker build -t myweb:v1 .
# 构建好了后，我们启动一个容器:
	$ docker run -d --name web -p 80:80 myweb:v1
# 当运行该镜像后，可以通过docker container ls看到最初的状态为(health: starting):
    $ docker container ls
    CONTAINER ID	IMAGE	COMMAND		CREATED		 STATUS		PORTS		NAMES
	03e28eb00bd0	myweb:v1	"nginx -g 'daemon off"	3 seconds ago Up 2 seconds (health: starting)	80/tcp, 443/tcp		web
# 在等待几秒钟后，再次docker container ls，就会看到健康状态变化为了(healthy):
	$ docker container ls
	CONTAINER ID	IMAGE	COMMAND		CREATED		STATUS		PORTS		NAMES
	03e28eb00bd0	myweb:v1	"nginx -g 'daemon off"	18 seconds ago Up 16 seconds (healthy)	80/tcp, 443/tcp		web
# 如果健康检查连续失败超过了重试次数，状态就会变为(unhealthy)。	
# 为了帮助排障，健康检查命令的输出(包括stdout以及stderr)都会被存储于健康状态里，可以用docker inspect来查看。
$ docker inspect --format '{{json .State.Health}}' web | python -m json.tool
{
    "FailingStreak": 0,
    "Log": [
		{
            "End": "2016-11-25T14:35:37.940957051Z",
            "ExitCode": 0,
            "Output": "<!DOCTYPE html>\n<html>\n<head>\n<title>Welcome to nginx!</titl
e>\n<style>\n	body {\n	width: 35em;\n	margin: 0 auto;\n	font-family: Tahoma, Verdana, Arial, sans-serif;\n	}\n</style>\n</head>\n<body>\n<h1>Welcome to nginx!</h1>\n<p>If you see this page, the nginx web server is successfully inst
alled and\nworking. Further configuration is required.</p>\n\n<p>For online documentat
ion and support please refer to\n<a href=\"http://nginx.org/\">nginx.org</a>.<br/>\nCo
mmercial support is available at\n<a href=\"http://nginx.com/\">nginx.com</a>.</p>\n\n
<p><em>Thank you for using nginx.</em></p>\n</body>\n</html>\n",
			"Start": "2016-11-25T14:35:37.780192565Z"
		}
	],
	"Status": "healthy"
}
```



#### ONBUILD 为他人做嫁衣裳

```shell
* 格式: ONBUILD <其它指令>。
# ONBUILD是一个特殊的指令，它后面跟的是其它指令，比如RUN，COPY等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。
# Dockerfile中的其它指令都是为了定制当前镜像而准备的，唯有ONBUILD是为了帮助别人定制自己而准备的。
# 假设我们要制作 Node.js 所写的应用的镜像。我们都知道 Node.js 使用npm进行包管理，所有依赖、配置、启动信息等会放到package.json文件里。在拿到程序代码后，需要先进行npm install才可以获得所有需要的依赖。然后就可以通过npm start来启动应用。因此，一般来说会这样写Dockerfile：
    FROM node:slim
    RUN mkdir /app
    WORKDIR /app
    COPY ./package.json /app
    RUN [ "npm", "install" ]
    COPY . /app/
    CMD [ "npm", "start" ]
# 把这个Dockerfile放到 Node.js 项目的根目录，构建好镜像后，就可以直接拿来启动容器运行。但是如果我们还有第二个 Node.js 项目也差不多呢？好吧，那就再把这个Dockerfile复制到第二个项目里。那如果有第三个项目呢？再复制么？文件的副本越多，版本控制就越困难，让我们继续看这样的场景维护的问题。
# 如果第一个 Node.js 项目在开发过程中，发现这个 Dockerfile 里存在问题，比如敲错字了、或者需要安装额外的包，然后开发人员修复了这个 Dockerfile ，再次构建，问题解决。第一个项目没问题了，但是第二个项目呢？虽然最初Dockerfile是复制、粘贴自第一个项目的，但是并不会因为第一个项目修复了他们的Dockerfile，而第二个项目的Dockerfile就会被自动修复。
# 那么我们可不可以做一个基础镜像，然后各个项目使用这个基础镜像呢？这样基础镜像更新，各个项目不用同步Dockerfile的变化，重新构建后就继承了基础镜像的更新？好吧，可以，让我们看看这样的结果。那么上面的这个Dockerfile就会变为：
    FROM node:slim
    RUN mkdir /app
    WORKDIR /app
    CMD [ "npm", "start" ]
# 这里我们把项目相关的构建指令拿出来，放到子项目里去。假设这个基础镜像的名字为my-node的话，各个项目内的自己的Dockerfile就变为：
    FROM my-node
    COPY ./package.json /app
    RUN [ "npm", "install" ]
    COPY . /app/
# 基础镜像变化后，各个项目都用这个Dockerfile重新构建镜像，会继承基础镜像的更新。
# 那么，问题解决了么？没有。准确说，只解决了一半。如果这个Dockerfile里面有些东西需要调整呢？比如npm install 都需要加一些参数，那怎么办？这一行RUN是不可能放入基础镜像的，因为涉及到了当前项目的./package.json，难道又要一个个修改么？所以说，这样制作基础镜像，只解决了原来的Dockerfile的前4条指令的变化问题，而后面三条指令的变化则完全没办法处理。
# ONBUILD可以解决这个问题。让我们用ONBUILD重新写一下基础镜像的Dockerfile：
    FROM node:slim
    RUN mkdir /app
    WORKDIR /app
    ONBUILD COPY ./package.json /app
    ONBUILD RUN [ "npm", "install" ]
    ONBUILD COPY . /app/
    CMD [ "npm", "start" ]
# 这次我们回到原始的Dockerfile，但是这次将项目相关的指令加上ONBUILD，这样在构建基础镜像的时候，这三行并不会被执行。然后各个项目的Dockerfile就变成了简单地:
	FROM my-node
# 是的，只有这么一行。当在各个项目目录中，用这个只有一行的Dockerfile构建镜像时，之前基础镜像的那三行ONBUILD就会开始执行，成功的将当前项目的代码复制进镜像、并且针对本项目执行npm install，生成应用镜像。
```

