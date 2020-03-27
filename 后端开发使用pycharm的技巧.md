# 后端开发使用pycharm的技巧

[TOC]

## 1、使用说明

首先说明，本文所使用的功能为pycharm专业版所支持，其他版本如社区版，教育版，则不一定支持。

作为一名后端开发，我猜你的桌面上一定打开着一系列的软件，用作开发调试工具，比如navicat数据库连接工具，postman接口调试工具，pycharm代码编写IDE，以及其他一些工具。今天，我就介绍一下pycharm中你可能还不知道的一些功能，让你的IDE、postman、navicat融为一体，从此不再需要频繁切屏。

## 2、database

这个功能本人觉得可以完全替代navicat，那么这个database功能在哪里呢。

<img src="/Users/luyuze/projects/blog/image/database.jpg" style="zoom:50%;" />

就在pycharm的右上角竖排的两个图标的其中一个，在这里可以添加数据库的连接。

<img src="/Users/luyuze/projects/blog/image/新建数据库连接.png" style="zoom:50%;" />

可以看到，支持非常多的数据库种类，基本上主流的数据库都可以连接，这里以MySQL为例。

<img src="/Users/luyuze/projects/blog/image/新建mysql连接.jpg" style="zoom:50%;" />

基本就和navicat一样，输入你要连接的数据库的连接名，host，port，user，password，数据库名，然后点击test connection就可以测试连接，第一次测试可能需要下载数据库连接驱动，下载就可以，测试成功就可以成功连接到你需要连接的数据库。

这里我新建一个数据库作为演示，可以看到，连接成功后可以显示数据库中所有的表，表字段，字段类型，字段注释，很齐全，打开表后，数据展示也很清晰，也可以直接像navicat那样直接对表数据进行可视化的增删改查操作，很方便我们开发的时候进行数据的测试调试。

<img src="/Users/luyuze/projects/blog/image/test_database.png" style="zoom:50%;" />

有了这个工具，从此可以抛弃navicat，直接在pycharm这样的IDE开发工具中进行数据库可视化操作了，免去切屏切来切去的麻烦。



## 3、HTTP Client

这一个工具可能知道的人更少，平时我们后端开发在调试restful api时，最常用的工具是postman，这个工具确实很方便，但是在pycharm中，也可以完成接口调试，那就是HTTP Client。

那么这个HTTP Client在哪里呢。

<img src="/Users/luyuze/projects/blog/image/http_client.png" style="zoom:50%;" />

打开之后

<img src="/Users/luyuze/projects/blog/image/httpclient界面.png" style="zoom:50%;" />

这些功能相信大家都应该再熟悉不过了，与postman是一样的，填写一个http请求的一些必须请求就可以发送请求，获取响应信息。但是这种方式不适合反复测试与保存，我更推荐的是接下来要介绍的，也是上图中蓝色提示部分的信息，即Convert request to the new format，转换请求为新的格式，那么是什么格式呢。

我们在项目中新建一个test目录，然后new新文件时，在最下方，有一个HTTP Request，默认后缀是http，我们就新建这种文件来做接口测试。

<img src="/Users/luyuze/projects/blog/image/newfile.png" style="zoom:50%;" />

这种文件是用来以一种固定的格式来定义请求的信息的，比如

<img src="/Users/luyuze/projects/blog/image/userhttp.png" style="zoom:50%;" />

先写请求方式、url，再写请求头信息，再写请求体(如果有)，也可以点击Add Request快速生成请求的模版，点击请求方法左边的小箭头就可以运行，查看结果，我们这里写了两个restful api来测试一下，连接的是上一节的数据库，测试增和查。

<img src="/Users/luyuze/projects/blog/image/result1.png" style="zoom:50%;" />

可以看到get请求到的json数据就展示出来了，展示效果和postman一样都很清晰。

再试试post一条新数据进数据库。点post的小箭头。

<img src="/Users/luyuze/projects/blog/image/result2.png" style="zoom:50%;" />

一样可以请求，去数据库看看结果

<img src="/Users/luyuze/projects/blog/image/result3.png" style="zoom:50%;" />

小赵已经添加进去了。

对于这个功能，我觉得完全可以替代postman，我们可以为我们的每一个数据模型在test下新建一个http请求文件，定义好GET、POST、PUT、DELETE请求信息，我们要测试接口时，直接点击就能运行，再配合上database功能直接修改数据库的数据，从此开发再也不用三个四个软件切来切去。

有收获的小伙伴留言点个赞！谢谢！