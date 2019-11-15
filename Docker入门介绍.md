# Docker入门介绍

[TOC]

## 1、Docker是什么

Docker 是基于Go语言实现的开源容器项目。利用操作系统本身已有的机制和特性，可以实现远超传统虚拟机的轻量级虚拟化。它是内核级的虚拟化。期望达到使项目运行环境“一次封装，到处运行的目的”。

利用docker创建的运行环境叫做docker容器，容器是通过docker镜像创建的，docker镜像文件可以放在私有仓库中也可以放在共有仓库中。最大的公有仓库是官方Docker Hub。

介绍再详细，不如直接上手使用。

## 2、安装Docker

要想使用Docker，当然要先安装Docker。这里安装过程不是介绍重点，建议直接根据网络上已有的教程安装即可，并无难度。

菜鸟教程各系统下安装方法皆有详细教程，可根据下面链接获取。

菜鸟教程：https://www.runoob.com/docker/docker-tutorial.html

## 3、确保docker已就绪

第一步，查看docker程序是否存在，功能是否正常。

```python
[root@VM_0_12_centos ~]# docker info
Client:
 Debug Mode: false

Server:
 Containers: 3
  Running: 2
  Paused: 0
  Stopped: 1
 Images: 3
 Server Version: 19.03.4
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
 runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
 init version: fec3683
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-957.21.3.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 991.2MiB
 Name: VM_0_12_centos
 ID: N67Z:2GTJ:H536:NCN7:W6R3:54YZ:6G3U:EFEL:7J4M:KDVI:QZEF:CYIN
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled

```

这里使用的是docker info命令，该命令会返回所有容器和镜像的数量、docker使用的执行驱动和存储驱动、以及docker的基本配置。可以理解为查看基本信息。

## 4、运行第一个容器

现在，尝试启动第一个Docker容器。可以使用docker run命令创建容器。

```python
[root@VM_0_12_centos ~]# docker run -i -t ubuntu /bin/bash

```
运行完这条命令后，首先会下载ubuntu这个镜像，然后创建一个容器，最后进入这个容器的bash交互。下面，来分析一下这条命令。

首先，我们告诉docker执行docker run命令，并指定了-i和-t两个命令行参数。-i标志保证容器中STDIN是开启的，尽管我们并没有附着到容器中。持久的标准输入是交互式shell的半边天，-t标志则是另外半边天，它告诉docker为要创建的容器分配一个伪tty终端。这样，新创建的容器才能提供一个交互式shell。若要在命令行下创建一个我们能与之进行交互的容器，而不是一个运行后台服务的容器，则这两个参数已经是最基本的参数了。

接下来，我们告诉docker基于什么镜像来创建容器，示例中使用的是ubuntu镜像。ubuntu镜像是一个常备镜像，也可以称为基础镜像，它由docker公司提供，保存在Docker Hub Registry上。可以以ubuntu基础镜像为基础，在选择的操作系统上构建自己的镜像。到目前为止，我们基于此基础镜像启动了一个容器，并且没有对容器增加任何东西。

这之后，首先docker会检查本地是否存在ubuntu镜像，如果本地还没有该镜像的话，那么docker就会连接官方维护的Docker Hub Registry，查看是否有该镜像。docker一旦找到该镜像，就会下载该镜像并将其保存到本地宿主机中。

随后，docker在文件系统内部用这个镜像创建了一个新容器。该容器拥有自己的网络，IP地址，以及一个用来和宿主机进行通信的桥接网络接口。最后，我们告诉docker在新容器中要运行什么命令，比如/bin/bash启动了一个Bash shell。当容器创建完毕之后，docker就会执行容器中的/bin/bash命令，这时就可以看到容器内的shell了。

```python
root@f7cbdac22a02:/#

```

## 5、使用第一个容器

现在我们已经以root用户登录到了新容器中，容器的ID是f7cbdac22a02。这是一个完整的ubuntu系统，可以用它来做任何事情。

检查容器的主机名：
```python
root@f7cbdac22a02:/# hostname
f7cbdac22a02

```

可以看到，容器的主机名就是该容器的ID。

安装一些软件包：

```python
root@f7cbdac22a02:/# apt-get update && apt-get install vim

```

在容器中安装了Vim软件。

用户可以继续在容器中做任何事情。当所有工作结束时，输入exit，就可以返回ubuntu宿主机的命令行提示符了。

这个容器现在怎样了？容器现在已经停止运行了！只有在指定的/bin/bash命令处于运行状态的时候，我们的容器也才会相应地处于运行状态。一旦退出容器，/bin/bash命令也就结束了，这时容器也随之停止了运行。

但是容器仍然是存在的，可以用docker ps -a命令查看当前系统中容器的列表，并显示它们的状态。

默认情况下，当执行docker ps命令时，只能看到正在运行的容器。如果指定-a标志的话，那么docker ps命令会列出所有容器。

## 6、容器命名

docker会为我们创建的每一个容器自动生成一个随机的名称。docker ps -a命令显示的最后一列即是。但是仍然建议为每一个容器指定一个名称，这不仅便于我们分辨这个容器的作用，也是这个容器的一个唯一标识。

```python
docker run --name my_container -i -t ubuntu /bin/bash

```

容器的命名必须是唯一的。如果试图创建两个名称相同的容器，则命令将会失败。如果要使用的容器名称已经存在，可以先用docker rm命令删除已有的同名容器后，再来创建新的容器。

## 7、重启已停止的容器

当我们退出一个容器之后，容器停止了，我们还可以重新启动。容器的名称或者ID都可以唯一标识一个容器。

```python
docker start my_container # 这里也可以用容器ID来启动

```

或者容器仍在运行时，我们也可以重启。

```python
docker restart my_container

```

docker也提供docker create命令来创建一个容器，但是并不运行它。

## 8、附着到容器上

docker容器重新启动的时候，会沿用docker run命令时指定的参数来运行，因此我们的容器重新启动后会运行一个交互式会话shell。此外，也可以用docker attach命令，重新附着到该容器的会话上。

```python
docker attach my_container

```

可能需要按下回车键才能进入该会话，如果退出容器的shell，容器会再次停止运行。

## 9、创建守护式容器

除了这些交互式运行的容器，也可以创建长期运行的容器。守护式容器没有交互式会话，非常适合运行应用程序和服务。下面就来启动一个守护式容器。

```python
docker run --name daemon_dave -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"

```

我们在上面的docker run命令使用了-d参数，因此Docker会将容器放到后台运行。

我们还在容器要运行的命令里使用了一个while循环，该循环会一直打印hello world，直到容器或其进程停止运行。

通过组合使用上面的这些参数，你会发现docker run命令并没有像上一个容器一样将主机的控制台附着到新的shell会话上，而是仅仅返回一个容器ID而已，我们还是在宿主机的命令行中。如果执行docker ps命令，可以看到一个正在运行的容器。

为了探究该容器内部都在干些什么，可以用docker logs命令来获取容器的日志。

```python
$ docker logs daemon_dave
hello world
hello world
hello world
hello world
hello world
hello world
……

```

这里我们看到while循环正在向日志里打印hello world。docker会输出最后几条日志并返回，我们也可以在命令后使用-f参数来监控docker日志，这与tail -f命令非常相似。

```python
$ docker logs -f daemon_dave

```

可以通过Ctrl+C退出日志跟踪。

也可以通过--tail获取最后几条日志

```python
$ docker logs --tail 10 daemon_dave

```

另外，也可以用docker logs --tail 0 -f daemon_dave命令来跟踪最新日志而不必读取整个日志文件。

还可以使用-t标识为每条加上时间戳。

## 10、Docker日志驱动

可以在启动docker守护进程或者执行docker run命令时指定--log-driver选项来实现容器所用的日志驱动。

有好几个选项，包括默认的json-file，json-file也为我们前面看到的docker logs命令提供了基础。

其他可用的选项还包括syslog，该选项禁用docker logs命令，并且将所有容器的日志输出重定向到Syslog。可以在启动Docker守护进程时指定该选项，将所有容器的日志都输出到Syslog，或者通过docker run对个别容器进行日志重定向输出。

```python
dokcer run --log-driver="syslog" --name daemon_dwayne -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"

```

最后还有一个可用选项是none，这个选项将会禁用所有容器中的日志，导致docker logs命令也被禁用。

## 11、查看容器内的进程

查看容器内部的进程，可以使用docker top命令。

```python
$ docker top daemon_dave

```

该命令执行后，可以看到容器内的所有进程、运行进程的用户以及进程ID。

## 12、Docker统计信息

使用docker stats命令，可以显示一个或者多个容器的统计信息。

```python
$ docker stats demo_1 demo_2 demo_3

```

可以显示CPU、内存、网络I/O以及存储I/O的性能和指标。

## 13、在容器内部运行进程

可以通过docker exec命令在容器内部额外启动新进程。可以在容器内运行的进程有两种类型：后台任务和交互式任务。后台任务在容器内运行且没有交互需求，而交互式任务则保持在前台运行。

下面先看一个后台任务例子：

```python
$ docker exec -d daemon_dave touch /etc/new_config_file

```

这里的-d表示需要运行一个后台进程。

也可以启动一个诸如打开shell的交互式任务：

```python
$ docker exec -t -i daemon_dave /bin/bash

```

## 14、停止守护式容器

直接使用docker stop就可以停止守护式容器

```python
$ docker stop daemon_dave
            # 容器名或者容器ID
```

## 15、自动重启容器

如果由于某种错误而导致容器停止运行，还可以通过--restart标志，让Docker重新启动该容器。--restart标志会检查容器的退出代码，并据此来决定是否要重启容器。默认的行为是docker不会重启容器。

```python
$ docker run --restart=always --name daemon_dave -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"  

```

在本例中，--restart标志被设置成always。无论容器的退出代码是什么，docker都会重启该容器。除了always，还可以将这个标志设置成on-failure，这样，只有当容器的退出代码为非0值的时候，才会自动重启。另外，on-failure还接受一个可选的重启次数参数。

```python
--restart=on-failure:5

```

## 16、深入容器

除了通过docker ps命令获取容器的信息，还可以使用docker inspect获取更多。

```python
$ docker inspect daemon_dave

```

docker inspect命令会对容器进行详细的检查，然后返回其配置信息，包括名称、命令、网络配置以及很多有用的数据。

## 17、删除容器

如果容器已经不再使用，可以使用docker rm命令来删除它们。

```python
$ docker rm daemon_dave

```

也可以通过下面小技巧来删除全部容器

```python
$ docker rm `docker ps -a -q`

```

上面的docker ps命令会列出现有的全部容器，-a标志代表列出所有容器，-q表示只需要返回容器的ID而不会返回其他信息，这样我们就得到了容器ID的列表，并传给了docker rm命令，从而达到删除所有容器的目的。

## 18、小结

本篇介绍了Docker容器的基本工作原理，让docker初学者对如何使用docker以及docker的优势有一个初步的认识。