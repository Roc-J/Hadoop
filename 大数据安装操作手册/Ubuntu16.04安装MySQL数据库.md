# Ubuntu16.04 安装MySQL及常用操作 #

作者：秦景坤  
GitHub：https://github.com/Roc-J  
CSDN博客：http://blog.csdn.net/qjk19940101  

MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下产品。MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件。  

MySQL是一种关系数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。 
 
MySQL所使用的 SQL 语言是用于访问数据库的最常用标准化语言。MySQL 软件采用了双授权政策，分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。

## 安装MySQL ##
使用一下命令即可进行mysql安装，注意安装前先更新一下软件源以获得最新版本

	sudo apt-get update  #更新软件源
	sudo apt-get install mysql-server  #安装mysql

![](http://i.imgur.com/yJI2bBm.png)

![](http://i.imgur.com/cmiCRKI.png)

apparmor
mysql-client-5.7
mysql-common
mysql-server
mysql-server-5.7
mysql-server-core-5.7
因此无需再安装mysql-client等。安装过程会提示设置mysql root用户的密码，设置完成后等待自动安装即可。默认安装完成就启动了mysql。

下面是mysql数据的启动和关闭命令：

	service mysql start
	service mysql stop

确认是否启动成功，mysql节点处于LISTEN状态表示启动成功

	sudo netstat -tap | grep mysql

![](http://i.imgur.com/XjEW4Ga.png)

进入mysql shell界面：

	mysql -u root -p

![](http://i.imgur.com/Jc9StHu.png)

## MySQL常用操作 ##
> 注意，MySQL中每个命令后都要以英文分号；结尾。


### 1.显示数据库 ###

	mysql>show databases;

MySQL安装完有四个数据库：information_schema,mysql,performance_schema,sys。mysql数据库非常重要，它里面有MySQL的系统信息，改密码和新增用户需要用这个表的相关表进行操作。

![](http://i.imgur.com/KEgvNAV.png)

### 2.显示数据库中的表 ###

	mysql>use mysql; #打开数据库，对每个数据库进行操作
	mysql>show tables;

![](http://i.imgur.com/JGku7px.png)

### 显示数据表的结构 ###

	describe user[表名]

![](http://i.imgur.com/fcg1lUm.png)

####  ####


