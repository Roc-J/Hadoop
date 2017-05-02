# 如何在Ubuntu16.04上安装MongoDB #

## 作者：秦景坤 ##
## 时间：2017-5-2 ##

## 什么是MongoDB ##
* MongDB是有C++编写，是一个基于分布式文件系统存储的开源数据库系统。  
* 在高负载的情况下，添加更多的节点，可以保证服务器性能。
* MongoDB旨在为WEB应用提供可扩展的高性能数据性能解决方案  
* MongoDB将数据存储为一个文档，数据结构由键值（key=>value）对组成。MongoDB文档类似于JSON对象。字段值可以包含其他文档，数据及文档数据。

![](http://i.imgur.com/4XKQZDx.png)

## 主要特点 ##
* MongoDB提供了一个面向文档存储，操作起来比较简单和容易。  
* 可以在MongoDB记录中设置任何属性的索引来实现更快的排序
* 可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。
* 如果负载的增加（需要更多的存储空间和更强的处理能力），它可以分布在计算机网络中的其他节点上这就是所谓的分片。
* Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。  
* MongoDB使用update()命令可以实现替换完成的文档（数据）或者指定一些指定的数据字段。
* MongoDB中的Map/Reduce主要是用来对数据进行批量处理和聚合操作。  
* Map和Reduce。
	* Map函数调用emit(key,value)遍历集合中所有的记录
	* 将key和value传给Reduce函数进行处理
* Map函数和Reduce函数是使用javascript函数进行编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作。  
* GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件。
* MongoDB允许在服务端执行脚本，可以用javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可。

## 安装步骤 ##
### 1. 导入公钥 ###

为Ubuntu系统增加MongoDB软件源的公共密钥，命令如下：

	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6

### 2.创建源列表文件MongoDB ###
使用此命令在/etc/apt/source.list.d/中创建MongoDB列表文件：

	echo "deb [ arch=amd64 ] http://repo.mongodb.org/apt/ubuntu precise/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

### 3.更新本地系统软件包数据库 ###

	sudo apt-get update

### 4.安装最新稳定版的MongoDB软件包 ###

	sudo apt-get install -y mongodb-org

必须要在/lib/systemd/system目录中创建一个新的mongodb systemd服务文件。切换到该目录，创建新的mongodb服务文件'mongodb.service'

	cd /lib/systemd/system/
	vi mongod.service

粘贴脚本:

	[Unit]
	Description=High-performance, schema-free document-oriented database
	After=network.target
	Documentation=https://docs.mongodb.org/manual
	
	[Service]
	User=mongodb
	Group=mongodb
	ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf
	
	[Install]
	WantedBy=multi-user.target

保存文件并退出。

更新systemd服务

	systemctl daemon-reload

![](http://i.imgur.com/uezEiOy.png)

启动mongoDB并将其添加为启动时启动的服务  

	systemctl start mongod

![](http://i.imgur.com/rqyEI8f.png)

现在检查mongodb是否已经通过netstat命令在端口27017上启动

	netstat -plntu

![](http://i.imgur.com/TDYc9Iz.png)


### 打开mongoDB shell ###
命令行输入mongo，可以打开MongoDB shell环境

	mongo

![](http://i.imgur.com/zhBjKOq.png)