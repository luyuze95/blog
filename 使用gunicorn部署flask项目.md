# 使用gunicorn部署flask项目

[TOC]

## 1、WSGI协议

Web框架致力于如何生成HTML代码，而Web服务器用于处理和响应HTTP请求。Web框架和Web服务器之间的通信，需要一套双方都遵守的接口协议。WSGI协议就是用来统一这两者的接口的。

## 2、WSGI容器

常用的WSGI容器有Gunicorn和uWSGI，但Gunicorn直接用命令启动，不需要编写配置文件，相对uWSGI要容易很多，所以这里我也选择用Gunicorn作为容器。

## 3、gunicorn介绍

gunicorn是一个python Wsgi http server，只支持在Unix系统上运行，来源于Ruby的unicorn项目。Gunicorn使用prefork master-worker模型（在gunicorn中，master被称为arbiter），能够与各种wsgi web框架协作。

## 4、gunicorn安装

gunicorn安装非常简单，使用命令pip install gunicorn即可。一般使用它，主要是为使用其异步的worker模型，还需要安装对应的异步模块。

```python
$ pip install greenlet # 使用异步必须安装
$ pip install eventlet # 使用eventlet workers
$ pip install gevent   # 使用gevent workers
```

## 5、gunicorn使用

这里使用gunicorn来部署一个flask项目举例，此处flask框架的使用不过多阐述，不是本文的重点。

如下例子，保存为app.py

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

```

gunicorn通常使用的参数如下：

```python
-c CONFIG, --config=CONFIG
# 设定配置文件。
-b BIND, --bind=BIND
# 设定服务需要绑定的端口。建议使用HOST:PORT。
-w WORKERS, --workers=WORKERS
# 设置工作进程数。建议服务器每一个核心可以设置2-4个。
-k MODULE
# 选定异步工作方式使用的模块。

```

在shell中输入你的启动配置，比如：

```python
$ gunicorn -w 3 -b 127.0.0.1:8080 app:app
# 此处app:app中，第一个app为flask项目实例所在的包，第二个app为生成的flask项目实例

```

这样运行正常就可以启动服务器了。

## 6、绑定端口

linux通常会禁止绑定使用1024以下的端口，除非在root用户权限。很多人在使用gunicorn时试图将其绑定到80或者443端口，发现无效。如果想绑定到这些端口，常见的有如下的几种方法：

- 使用Nginx代理转发。
- sudo启动gunicorn。
- 安装额外的程序。

## 7、结束gunicorn服务进程

使用ps -ef | grep gunicorn命令找出gunicorn所有进程。

```python
[root@VM_0_12_centos ~]# ps -ef | grep gunicorn
root     16843 23035  0 Oct14 ?        00:00:02 /root/Envs/myflask/bin/python3.6 /root/Envs/myflask/bin/gunicorn -w 3 -b 172.17.0.12:80 app:app
root     22445 23035  0 Oct04 ?        00:00:15 /root/Envs/myflask/bin/python3.6 /root/Envs/myflask/bin/gunicorn -w 3 -b 172.17.0.12:80 app:app
root     22581 23035  0 Oct11 ?        00:00:05 /root/Envs/myflask/bin/python3.6 /root/Envs/myflask/bin/gunicorn -w 3 -b 172.17.0.12:80 app:app
root     23035     1  0 Sep27 ?        00:04:11 /root/Envs/myflask/bin/python3.6 /root/Envs/myflask/bin/gunicorn -w 3 -b 172.17.0.12:80 app:app

```

然后使用 kill -9 进程ID 命令来杀掉进程，注意，我们找到主进程杀掉即可，子进程会随之结束，在上例中，主进程号为23035.

```python
[root@VM_0_12_centos ~]# kill -9 23035
[root@VM_0_12_centos ~]# ps -ef | grep gunicorn

```

杀掉进程后，稍等几秒，再使用ps -ef | grep gunicorn查看，发现gunicorn服务进程已全部杀掉。
