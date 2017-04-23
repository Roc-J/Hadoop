# Ubuntu16.04安装Hadoop2.7.3 教程 #
参考厦门大学数据库实验室 http://dblab.xmu.edu.cn/blog/install-hadoop/，遇到相关的地方有改动。

## 作者：秦景坤 ##
## 日期：2017-4-20 ##
## 主要内部包括 环境配置和本地和伪分布 ##

**本文档适合于原生Hadoop2，参考相关文档，亲自动手实践来一步一步搭建环境。转载请指明出处。**

## 环境 ##

本教程使用Ubuntu16.04 64位作为系统环境，包括桌面版和server版，其他版本系统，若有差异请自行百度安装教程系统。  

本教程基于原生Hadoop2，安装的版本是Hadoop 2.7.3版本。

> 使用本教程请确保系统处于联网状态，如果自己的电脑配置不够高，安装虚拟机会导致系统变慢变卡的话，建议可以将电脑安装成双系统来进行安装，然后再使用本教程。

> 本教程是在虚拟机下安装的Ubuntu16.04版本，但是时使用的远程连接xshell进行的操作。

装好了Ubuntu系统之后，在安装Hadoop前还需要做一些必备工作

> 创建hadoop用户

如果你安装Ubuntu的时候不是用的“hadoop”用户，那么需要增加一个名为hadoop的用户。（其实对用户名没有那么大严格的要求，但是感觉这样挺方便的）

首先按ctrl+alt+t打开终端窗口，或者用xshell进行连接操作，输入如下命令创建新用户。

    sudo useradd -m hadoop -s /bin/bash

这条命令创建了可以登录的hadoop用户，并使用/bin/bash作为shell。

>sudo命令  
> 本文中会大量使用到sudo命令。sudo是linux系统管理指令，是允许系统管理员让普通用户执行一些或者全部的root命令的一个工具，如halt，reboot，su等等。这样不仅减少了root用户的登录 和管理时间，同样也提高了安全性。sudo不是对shell的一个代替，它是面向每个命令的。当使用sudo命令时，就需要输入您当前使用用户的密码

> 密码
> 在Linux的终端中输入密码，终端是不会显示你当前输入的密码，也不会提示你已经输入了多少字符密码。

接着使用如下命令设置密码，可简单的设置为hadoop，按提示输入两次密码：

    sudo passwd hadoop

可为hadoop用户增加管理员权限，方便部署，避免一些对新手来说比较棘手的权限问题

    sudo adduser hadoop sudo

![](http://i.imgur.com/K0xvwsS.png)

最后使用su hadoop 切换到用户hadoop，或者注销当前用户，选择hadoop登录。

> 更新apt  

切换到hadoop用户后，先更新一些apt，后续会使用apt安装软件，如果没更新可能有一些软件安装不了。按ctrl+alt+t打开终端窗口或者使用xshell建立远程连接，执行如下命令：

    sudo apt-get update

![](http://i.imgur.com/bFjusSP.png)

后续需要更改一些配置文件，我比较喜欢用的是vim（vi增强版，基本用法相同），建议安装一下（如果你实在还不会用vi，请使用桌面版的ubuntu的gedit,这样可以使用文本编辑器进行修改，并且每次文件更改完成后请关闭整个gedit程序，否则会占用终端。)

	sudo apt-get install vim

安装软件时需要确认，在提示处输入y即可。

![](http://i.imgur.com/XVwv8q2.png)

上面显示没有待安装的项目。


> 安装SSH、配置SSH无密码登录

集群，单节点模式都需要用到SSH登录（类似于远程登录，你可以登录某台Linux主机，并且在上面运行命令），Ubuntu默认已安装了SSH client，此外还需要安装SSH server:

    sudo apt-get install openssh-server

安装后，可以使用命令登录本机：

    ssh localhost

此时会有如下提示（SSH首次登陆提示），输入yes。然后按提示输入密码hadoop，这样就可以登录到本机。

![](http://i.imgur.com/US1aVLl.png)

但这样登录是需要每次输入密码的，我们需要配置成SSH无密码登录比较方便。

首先推出刚才的ssh，就回到了原先的终端窗口，然后利用ssh-keygen生成密钥，并将密钥加入到授权中：

![](http://i.imgur.com/rnyjxRM.png)

* exit 退出刚才的ssh localhost   
* cd ~/.ssh/ 若没有该目录，请执行一次ssh localhost  
* ssh-keygen -t rsa 会有提示，都按回车就可以  
* cat ./id_rsa.pub >> ./authorized_keys 加入授权

> ~的含义
> 在Linux中，~代表的是用户的主文件夹，即"/home/用户名"这个目录，如你的用户名为hadoop，则~就代表"/home/hadoop"。此外，命令中的#后面的文字是注释，只需要输入前面命令即可。

此时再用ssh localhost命令，无需输入密码就可以直接登录了，如下图所示。

![](http://i.imgur.com/BtbBhkJ.png)


## 安装Java环境   ##

Java环境可选择Oracle的JDK，或是OpenJDK，本教程安装的是Oracle的JDK。

1.检查自己的系统是32-bit还是64-bit

在命令行输入：

    getconf LONG_BIT

返回32或者是64，可以来查看操作系统的位数是32位还是64位

![](http://i.imgur.com/PwWzPid.png)

2.检查你的系统是否已经安装了java，输入命令：  

    java -version

![](http://i.imgur.com/sSnxc6Y.png)

> 如果有安装OpenJDK，需要先卸载OpenJDK

3.建立Java目录，输入命令:

    sudo mkdir -p /usr/local/java

![](http://i.imgur.com/Q3MTT3p.png)

4.下载Linux版本的Orcle Java JDK，网址如下：[http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

![](http://i.imgur.com/dxX5dhl.png)

下载对应的版本jdk，本教程下载的是jdk-8u131-linux-x64.tar.gz

5.复制文件到刚刚创建的Java目录，路径为/usr/local/java;由于大多数的下载文件都会在路径/home/"你的用户名"/Downloads这个文件夹中，这里以此为例：

	cd /home/hadoop/Downloads 
	sudo cp -r jdk-8u131-linux-x64.tar.gz /usr/local/java/
	cd /usr/local/java

6.解压下载文件到此目录：  

	sudo tar xvzf jdk-8u131-Linux-x64.tar.gz

7.下面开始设置环境变量

	sudo vim /etc/profile
	
复制下面代码到文件的末尾处，然后保存并关闭文件：

	JAVA_HOME=/usr/local/java/jdk1.8.0_131
	PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
	export JAVA_HOME
	export PATH

8.通知你的Ubuntu Linux系统Oracle Java JDK的位置:

![](http://i.imgur.com/iOrky2r.png)

9.将Orcle java JDK设置为默认:

![](http://i.imgur.com/fitpgHd.png)
	
10.重新加载下/etc/profile中的路径:

	source /etc/profile

11.输入如下代码，测试是否安装配置成功：

![](http://i.imgur.com/jxhknns.png)

这样，Hadoop安装所需要的Java环境就配置好了。

## 安装Hadoop2 ##

Hadoop2 可以通过[http://mirrors.cnnic.cn/apache/hadoop/common/](http://mirrors.cnnic.cn/apache/hadoop/common/) 下载，一般选择下载最新的稳定版本，即下载"stable"下的hadoop-2.x.y.tar.gz这个格式的文件，这是编译好的，另一个包含src的则是Hadoop源代码，需要进行编译才可使用。


> 截止到2017年4月21日，Hadoop官方网站已经更新到2.8.0版本。对于2.7.3以上版本的Hadoop，仍可以参照此教程学习，可放心下载官网最新版本的Hadoop。  
> 1.如果读者是使用虚拟机方式安装Ubuntu系统的用户，请用虚拟机中的Ubuntu自带firefox浏览器访问本指南，再点击下面的地址，才能把hadoop文件下载虚拟机Ubuntu中。  
> 2.如果用windows下载，则文件会被下载到Windows系统中，虚拟机中的Ubuntu无法访问外部Windows系统的文件，可以通过xftp软件进行传输。  
> 3.如果读者是使用双系统方式安装Ubuntu系统的用户，请进去Ubuntu系统，在Ubuntu系统打开Firefox浏览器访问本教程。

下载完hadoop文件后一般可以直接使用。

我们选择将Hadoop安装至/usr/local/中：  

	sudo tar -zxf ~/Downloads/hadoop-2.7.3.tar.gz -C /usr/local  #解压到/usr/local中
	cd /usr/local/
	sudo mv ./hadoop-2.7.3/ ./hadoop  #将文件夹名改为hadoop
	sudo chown -R hadoop ./hadoop     #修改文件权限  

Hadoop解压后即可使用。输入如下命令来检查Hadoop是否可用，成功则会显示Hadoop版本信息：

	cd /usr/local/hadoop
	./bin/hadoop version

![](http://i.imgur.com/q15EiHU.png)

> 相对路径与绝对路径  
> 请务必注意命令中的相对路径与绝对路径，本文后续出现的./bin/...，./etc/...等包含./的路径，均为相对路径，以/usr/local/hadoop为当前目录。例如在/usr/local/hadoop目录中执行./bin/hadoop version等同于执行 /usr/local/hadoop/bin/hadoop version。可以将相对路径改成绝对路径来执行，但如果你是在主文件夹~中执行 ./bin/hadoop version,执行的会是/home/hadoop/bin/hadoop version,就不是我们所想要的。

## Hadoop单击配置（非分布式） ##

Hadoop默认模式为非分布式模式（本地模式），无需进行其他配置即可运行。非分布式即单java进程，方便进行调试。

现在可以执行例子来感受下Hadoop的运行。Hadoop附带了丰富的例子（运行./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar 可以看到所有的例子），包括wordcount、terasort、join、grep等。

![](http://i.imgur.com/o7FYPLN.png)

在此我们选择运行grep例子，将input文件夹中的所有文件作为输入，筛选当中符合正则表达式dfs[a-z.]+的单词并统计出现的次数，最后输出结果到output文件夹中。

    cd /usr/local/hadoop
	mkdir ./input
	cp ./etc/hadoop/*.xml ./input  #将配置文件作为输入文件
	./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep ./input/ ./output 'dfs[a-z.]+'
	cat ./output/*

执行成功后如下所示，输出了作业的相关信息，输出的结果是符合正则的单词dfsadmin出现了1次

![](http://i.imgur.com/M9PheDn.png)

![](http://i.imgur.com/UsxlqZy.png)

![](http://i.imgur.com/W58LNyn.png)

> 注意，Hadoop默认不会覆盖结果文件，因此再次运行上面实例会提示出错，需要先将./output删除。

	rm -r ./output

## Hadoop伪分布式配置 ##

Hadoop可以在单节点上以伪分布式的方式运行，Hadoop进程以分离的java进行来运行，节点既作为NameNode也作为DataNode，同时，读取的是HDFS中的文件。

Hadoop的配置文件位于/usr/local/etc/hadoop中，伪分布式需要修改2个配置文件**core-site.xml**和**hdfs-site.xml**。Hadoop的配置文件是xml格式，每个配置以声明property的name和value的方式来实现。

修改配置文件core-site.xml（使用 vi ./etc/hadoop/core-site.xml）,修改下面的  
	
	<configuration>  
	</configuration>  

![](http://i.imgur.com/bOWTiP7.png)

![](http://i.imgur.com/SBFvRBf.png)

同样的，修改配置文件hdfs-site.xml

![](http://i.imgur.com/iDA1KDR.png)


> Hadoop配置文件说明
> Hadoop的运行方式是由配置文件决定的（运行Hadoop时会读取配置文件），因此如果需要从伪分布模式切换到非分布式模式，需要删除core-site.xml中的配置项。  
> 此外，伪分布式虽然只需要配置fs.defaultFS和dfs.replication就可以运行（官方教程如此)，不过若没有配置hadoop.tmp.dir参数，则默认使用的临时目录为/tmp/hadoop-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行format才行。所以进行了设置，同时也指定dfs.namenode.name.dir和dfs.datanode.data.dir，否则在接下来的步骤中可能会处错。

配置完成后，执行NameNode的格式化  

	./bin/hdfs namenode -format

成功的话，会看到"successfully formatted"和"Exitting with status 0"的提示，若为"Exitting with status 1"则是处错。

第一次运行的时候出现了一个处错：

![](http://i.imgur.com/UalC5LK.png)

上面的错误说是XML的版本不支持，在笔者将2.0改为1.0后，重新运行结果如下:

![](http://i.imgur.com/5FZppcC.png)

接着开启NameNode和DataNode守护进程。

	./sbin/start-dfs.sh          #start-dfs.sh是个完整的执行文件，中间没有空格键  

![](http://i.imgur.com/VpflUPD.png)

若出现SSH提示，输入yes即可。

> 启动时若提示没有找到JAVA_HOME，则需要在hadoop文件下的/etc/hadoop/hadoop-env.sh下将JAVA_HOME重新写一下路径，笔者遇到了这个问题。  
> 启动时可能会出现如下WARN提示：WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable  
> WARN提示可以忽略，并不会影响正常使用。


启动完成后，可以通过命令jps来判断是否成功启动，若成功启动则会列出如下进程:  
"NameNode","DataNode","SecondaryNameNode"（如果SecondaryNameNode没有启动，请运行sbin/stop-dfs.sh关闭进行，然后再次尝试启动尝试）。如果没有NameNode或DataNode,那就是配置不成功，请仔细检查之前步骤，或通过查看启动日志排查原因。

![](http://i.imgur.com/8Mt0NS7.png)   


## 运行Hadoop伪分布式实例 ##
上面的单击模式，grep例子读取的是本地数据，伪分布式读取则是HDFS上的数据。要使用HDFS,首先需要在HDFS中创建用户目录: 

	./bin/hdfs dfs -mkdir -p /usr/hadoop

接着将./etc/hadoop中的xml文件作为输入文件复制到分布式系统中，即将/usr/local/hadoop/etc/hadoop复制到分布式文件系统中的/user/hadoop/input中。使用的是hadoop用户，并且已创建相应的用户目录/user/hadoop，因此在命名中可以使用相对路径如input，其对应的绝对路径是/user/hadoop/input

    ./bin/hdfs dfs -mkdir input
	./bin/hdfs dfs -put ./etc/hadoop/*.xml input

复制完成后，可以使用如下命令查看文件列表：
    
    ./bin/hdfs dfs -ls input

![](http://i.imgur.com/91vnjWs.png)

伪分布式运行MapReduce作业的方式和单击模式相同，区别在于伪分布式读取的是HDFS中的文件（可以将单机步骤中创建的本地input文件夹，输出结果output文件夹都删掉来验证这一点）。

	./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'

![](http://i.imgur.com/3OwY3kb.png)

查看运行结果的命令（查看的是位于HDFS中的输出结果）：

	./bin/hdfs dfs -cat output/*

![](http://i.imgur.com/y1rXLWy.png)

若要关闭Hadoop，则运行

	./sbin/stop-dfs.sh

> 注意  
> 下次启动hadoop时，无需进行NameNode的初始化，只需要运行./sbin/start-dfs.sh就可以。