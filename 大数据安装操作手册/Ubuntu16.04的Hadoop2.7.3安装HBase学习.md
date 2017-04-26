# 分布式数据库HBase #

## HBase介绍 ##

HBase是一个分布式的、面向列的开源数据库，源于Google的一篇论文《BigTable：一个结构化数据的分布式存储系统》。HBase以表的形式存储数据，表有行和列组成，列划分为若干个列族/列族(column family)。  
欲了解HBase的官方咨询，请访问HBase官方文档[http://hbase.apache.org/](http://hbase.apache.org/ "HBase官方")

HBase的运行有三种模式：单机模式、伪分布模式、分布式模式。

**单机模式:** 在一台计算机上安装和使用HBase，不涉及数据的分布式存储  
**伪分布式模式：** 在一台计算机上模拟一个小的集群  
**分布式模式：** 使用多台计算机实现物理意义上的分布式存储。

## 环境 ##
* 操作系统： Ubuntu16.04   
* hadoop: 2.7.3  
* HBase: 1.2.5 

HBase下载地址 : [http://archive.apache.org/dist/hbase/stable/](http://archive.apache.org/dist/hbase/stable/)

## 安装并配置HBase ##
### HBase安装 ###

1.解压安装包hbase-1.2.5-bin.tar.gz到路径/usr/local，命令如下:

	sudo tar -zxf ~/Downloads/hbase-1.2.5-bin.tar.gz -C /usr/local

2.将解压的文件名hbase-1.2.5改为hbase，以方便使用，命令如下:  

	cd /usr/local
	sudo mv hbase-1.2.5 hbase

3.配置环境变量  
将hbase下的bin目录添加到path中，这样启动hbase就无需到/usr/local/hbase目录下，大大的方便了hbase的使用。

![](http://i.imgur.com/IULVwUA.png)

编辑完成后，再执行source命令使上述配置在当前终端立即生效，命令如下

	source /etc/profile

4.添加HBase权限  

    cd /usr/local
	sudo chown -R hadoop ./hbase #将hbase下的所有文件的所有者改为hadoop，hadoop是当前用户的用户名

5.查看HBase版本，确定hbase安装成功，命令如下:   

	hbase version 
	或者
	/usr/local/hbase/bin/hbase version

执行命令后，输出信息如下： 

![](http://i.imgur.com/bUTCYvM.png)

看到以上输出信息表示HBase已经安装成功。

### HBase配置 ###

HBase有三种运行模式，单机模式，伪分布式模式，分布式模式。

确保 jdk,hadoop,SSH已经安装。

#### 单机模式配置 ####

1.配置/usr/local/hbase/conf/hbase-env.sh  
* 配置JAVA环境变量  
* 添加配置HBASE_MANAGES_ZK为true  
用vi命令打开并编辑hbase-env.sh

![](http://i.imgur.com/Z3O9Xds.png)

![](http://i.imgur.com/J09qrO3.png)

配置JAVA环境变量，jdk的安装目录默认是 /usr/local/java/jdk1.8.0_131(请自行修改个人的安装目录）  
配置HBASE_MANAGES_ZK为true,表示hbase自己管理zookeeper，不需要单独的zookeeper。  

hbase-env.sh中本来就存在这些变量的配置，大家只要删除前面的#并修改配置内容即可。

2.配置/usr/local/hbase/conf/hbase-site.xml  

	vi /usr/local/hbase/conf/hbase-site.xml

在启动HBase前需要设置属性hbase.rootdir，用于指定HBase数据的存储位置，因为如果不设置的话，hbase.rootdir默认为/tmp/hbase-${user.name}；这意味着每次重启系统都会丢失数据。此处设置为HBase安装目录下的hbase-tmp文件夹即（/usr/local/hbase/hbase-tmp),添加配置如下:   

![](http://i.imgur.com/cZQ2l0d.png)

	<configuration>
        <property>
                <name>hbase.rootdir</name>
                <value>/usr/local/hbase/hbase-tmp</value>
        </property>
	</configuration>

3.接下来测试运行。首先切换目录至HBase安装目录/usr/local/hbase；再启动HBase。

	cd /usr/local/hbase
	bin/start-hbase.sh
	bin/hbase shell

上述三条命名中，bin/start-hbase.sh用于启动HBase，bin/hbase shell用于打开shell命令行模式，用户可以通过输入shell命令操作HBase数据库

成功启动HBase，截图如下:  

![](http://i.imgur.com/swQWIvc.png)

停止HBase运行，命令如下

	bin/stop-hbase.sh


![](http://i.imgur.com/iYzQuZI.png)


