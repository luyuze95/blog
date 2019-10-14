# centos下搭建python双版本环境

[TOC]

centos7自带有python，版本是python2.7。但是在生产环境中，往往可能需要更高的python版本，甚至是有需要双版本python同时使用的情况。接下来我们手动安装python3，并且配置后可以并存使用。

## 一、安装python3

### 1、理清自带python位置

```python
[root@centos ~]# whereis python
python: /usr/bin/python2.7 /usr/bin/python /usr/lib/python2.7 /usr/lib64/python2.7 /etc/python /usr/include/python2.7 /usr/share/man/man1/python.1.gz
```

由此得知我们的python在 /usr/bin目录中

```python
[root@centos ~]# cd /usr/bin/
[root@centos bin]# ll python*
lrwxrwxrwx. 1 root root    7 2月   7 09:30 python -> python2
lrwxrwxrwx. 1 root root    9 2月   7 09:30 python2 -> python2.7
-rwxr-xr-x. 1 root root 7136 8月   4 2017 python2.7
```

可以看到，python指向的是python2，python2指向的是python2.7，因此我们可以装个python3，然后将python指向python3，然后python2指向python2.7，那么两个版本的python就能共存了。

### 2、更新用于下载编译python3的相关包

```python
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
```

运行此命令即可更新

### 3、安装pip

```python
yum -y install epel-release
yum install python-pip
```

运行以上两个命令安装pip

### 4、用pip安装wget

```python
pip install wget
```

运行上述命令安装wget

### 5、用wget下载python3的源码包

```python
wget https://www.python.org/ftp/python/3.6.4/Python-3.6.4.tar.xz
```

执行上述命令，即可下载python3的tar.xz的源码包

### 6、编译python3源码包

```python
# 先对tar.xz包解压
xz -d Python-3.6.4.tar.xz
tar -xf Python-3.6.4.tar

# 然后先进入到解压后的目录中，依次执行下面命令进行手动编译
cd Python-3.6.4
./configure prefix=/usr/local/python3
make && make install

# 如果出现can't decompress data; zlib not available这个错误，则需要安装相关库，如果正常，则不必执行下面的命令。
#安装依赖zlib、zlib-devel
yum install zlib zlib
yum install zlib zlib-devel
```

如果最后没提示出错，就代表正确安装了，在/usr/local/目录下就会有python3目录

### 7、添加软链接

```python
#将原来的链接备份
mv /usr/bin/python /usr/bin/python.bak
 
#添加python3的软链接
ln -s /usr/local/python3/bin/python3.6 /usr/bin/python
 
#测试是否安装成功了
python -V
```

### 8、更改yum配置

更改yum配置，因为其要用到python2才能执行，否则会导致yum不能正常使用

```python
vi /usr/bin/yum
把#! /usr/bin/python修改为#! /usr/bin/python2
 
vi /usr/libexec/urlgrabber-ext-down
把#! /usr/bin/python 修改为#! /usr/bin/python2
```

### 9、使用python双版本

```python
[root@centos ~]# python2
Python 2.7.5 (default, Jun 20 2019, 20:27:34) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
[root@centos ~]# python
Python 3.6.4 (default, Sep 27 2019, 16:54:08) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()

```

至此，python双版本已经都可以正常使用了。

## 二、创建虚拟环境

一般在生产环境中，我们不会直接使用系统的python环境，因为这样各种依赖包存在很难管理的情况，因此，使用虚拟环境就会好很多，每一个具体的项目，我们可以使用一个单独的虚拟环境，这样各环境之间独立使用，依赖互不影响。

### 1、使用virtualenv

#### 安装

```python
pip install virtualenv
```

#### 为一个工程创建一个虚拟环境

```python
$ cd my_project_dir
$ virtualenv venv　　# venv为虚拟环境目录名，目录名自定义
```

virtualenv venv 将会在当前的目录中创建一个文件夹，包含了Python可执行文件，以及 pip 库的一份拷贝，这样就能安装其他包了。虚拟环境的名字（此例中是 venv ）可以是任意的；若省略名字将会把文件均放在当前目录。在任何你运行命令的目录中，这会创建Python的拷贝，并将之放在叫做 venv 的文件中。

你可以选择使用一个Python解释器：

```python
$ virtualenv -p /usr/bin/python2.7 venv　　　　# -p参数指定Python解释器程序路径
```

这将会使用 /usr/bin/python2.7 中的Python解释器。

#### 开始使用虚拟环境

```python
$ source venv/bin/activate　# 激活
```

从现在起，任何你使用pip安装的包将会放在 venv 文件夹中，与全局安装的Python隔绝开。

像平常一样安装包，比如：

```python
pip install requests
```

#### 退出使用虚拟环境

如果你在虚拟环境中暂时完成了工作，则可以停用它：

```python
$ . venv/bin/deactivate
```

这将会回到系统默认的Python解释器，包括已安装的库也会回到默认的。

要删除一个虚拟环境，只需删除它的文件夹。（执行 rm -rf venv ）。

这里virtualenv 有些不便，因为virtual的启动、停止脚本都在特定文件夹，可能一段时间后，你可能会有很多个虚拟环境散落在系统各处，你可能忘记它们的名字或者位置。

### 2、使用virtualenvwrapper

鉴于virtualenv不便于对虚拟环境集中管理，所以推荐直接使用virtualenvwrapper。 virtualenvwrapper提供了一系列命令使得和虚拟环境工作变得便利。它把你所有的虚拟环境都放在一个地方。

#### 安装

linux安装virtualenvwrapper(确保virtualenv已安装)

```python
pip install virtualenvwrapper
```

#### 配置

安装完成后，在~/.bashrc写入以下内容
```python
export WORKON_HOME=~/Envs
source /usr/local/bin/virtualenvwrapper.sh　
```

- 第一行：virtualenvwrapper存放虚拟环境的目录
- 第二行：virtrualenvwrapper会安装到python的bin目录下，所以该路径是python安装目录下bin/virtualenvwrapper.sh

```python
source ~/.bashrc　# 读入配置文件，立即生效
```

#### 可能存在的问题

- 问题1

```python
/bin/python: No module named virtualenvwrapper
virtualenvwrapper.sh: There was a problem running the initialization hooks.

If Python could not import the module virtualenvwrapper.hook_loader,
check that virtualenvwrapper has been installed for
VIRTUALENVWRAPPER_PYTHON=/bin/python and that PATH is
set properly.
```

- 问题2

```python
[root@centos ~]# source .bashrc
-bash: /usr/local/bin/virtualenvwrapper.sh: No such file or directory
```

- 解决方法
    - 问题1

    在~/.bashrc写入以下内容

    ```python
    export VIRTUALENVWRAPPER_PYTHON=/usr/local/python36/bin/python3 # 指定虚拟使用的python解释器路径
    ```

    - 问题2

    virtualenvwrapper.sh找不到报错，找到后拷贝到/usr/local/bin/下

    然后执行source ~/.bashrc　

#### 开始使用

- 创建虚拟环境

```python
mkvirtualenv venv　# venv为虚拟环境的名字，自定义
```

这样会在WORKON_HOME变量指定的目录下新建名为venv的虚拟环境。若想指定python版本，可通过"--python"指定python解释器

```python
mkvirtualenv --python=/usr/local/python3.5.3/bin/python venv
```

- 查看当前的虚拟环境目录

```python
[root@centos ~]# workon
```

- 切换到虚拟环境

```python
[root@centos ~]# workon py3
(py3) [root@centos ~]#
```

- 退出虚拟环境

```python
(py3) [root@centos ~]# deactivate
[root@centos ~]# 
```

- 删除虚拟环境

```python
rmvirtualenv venv
```