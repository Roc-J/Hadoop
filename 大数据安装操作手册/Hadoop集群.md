# Hadoop集群安装配置教程_Hadoop2.7.3 #

本教程参考了厦门大学数据库实验室，网址：[http://dblab.xmu.edu.cn/blog/install-hadoop-cluster/](http://dblab.xmu.edu.cn/blog/install-hadoop-cluster/)

## 作者：秦景坤 ##
## 时间：2017-4-23 ##

## 环境 ##
本教程使用Ubuntu 16.04 64位作为系统环境，基于原生Hadoop2.7.3（stable)版本下验证通过，可适合任何Hadoop2.x.y版本。

本教程使用的主节点是Ubuntu16.04 64位桌面版，两个节点使用的是Ubuntu16.04 server版本  
Master IP :192.168.137.134  
Slave1 IP :192.168.137.137    
Slave2 IP :192.168.137.138

## 准备工作 ##
Hadoop集群的安装配置大致为如下流程:  
1.选定一台机器作为Master  
2.在Master节点上配置hadoop用户、安装SSH server、安装java环境  
3.在Master节点上安装Hadoop，并完成配置  
4.在其他Slave节点上配置hadoop用户，安装SSH server，安装Java环境  
5.将Master节点上的/usr/local/hadoop目录复制到其他Slave节点上  
6.在Master节点上开启Hadoop

## 网络配置 ##
本教程集群所用的节点均在同一个局域网中。

Linux中查看节点IP地址的命令为**ifconfig**，即下图所示的inet地址  

![](http://i.imgur.com/EauiUDV.png)

首先在Master节点上完成准备工作，再进行后续集群配置。

为了便于区分，可以修改各个节点的主机名（在终端标题，命令行中可以看到主机名，以便区分）。在Ubuntu16.04中，在Master节点上执行如下命令修改主机名（即该为Master，注意是区分大小写的）。

    sudo vim /etc/hostname

直接写一个Master即可

然后执行如下命令修改自己所用节点的IP映射：

	sudo vim /etc/hosts

![](http://i.imgur.com/jPui7XE.png)

在/etc/hosts中将该映射关系填写上去即可，如下图所示（一般该文件中只有一个127.0.0.1，其对应名为localhost，如果有多余的删除，特别是不能有"127.0.0.1 Master"这样的记录）：

![](http://i.imgur.com/eeuX45t.png)

修改完成后需要重启一下，重启后在终端中才会看到机器名的变化。接下来的教程中请注意区别Master节点和Slave节点的操作


> 需要在所有的节点上完成网络配置  
> 如上面讲的是Master节点的配置，而在其他的Slave节点上，也要对/etc/hostname(修改为Slave1,Slave2等)和/etc/hosts（跟Master的配置一样）这两个文件进行修改。  

配置好后需要在各个节点上执行如下命令，测试是否相互ping得通，如果ping不通，后面就无法顺利配置成功：

Master ping

![](http://i.imgur.com/uWUCBUP.png)

Slave1 ping 

![](http://i.imgur.com/HFrZ6eH.png)

Slave2 ping 

![](http://i.imgur.com/LjSvUaf.png)

## SSH无密码登录节点 ##

这个操作是要让Master节点可以无密码SSH登录到各个Slave节点上。  

首先生成Master节点的公钥，在Master节点的终端中执行（因为改过主机名，所以还需要删除原有的再重新生成一次）：

	cd ~/.ssh            #如果没有该目录，先执行一次ssh localhost
	rm ./id_rsa*         #删除之前生成公钥（如果有)
	ssh-keygen -t rsa    #一直按回车即可

![](http://i.imgur.com/qRf0eYl.png)

让Master节点需能无密码SSH本机，在Master节点上执行:

	cat ./id_rsa.pub >> ./authorized_keys 

完成后可执行**ssh Master**验证一下（可能需要输入yes，成功后执行exit返回原来的终端）。接着在Master节点将公钥传输到Slave1节点：

	scp /home/hadoop/.ssh/id_rsa.pub hadoop@Slave1:/home/hadoop 

> scp是secure copy的简写，用于在Linux下进行远程拷贝文件，类似于cp命令，不过cp只能在本机中拷贝。执行scp时会要求输入Slave1上hadoop用户的密码（hadoop)，输入完成后会提示传输完毕，如下图所示。

![](http://i.imgur.com/raZaDFp.png)

接着在Slave1节点上，将ssh公钥加入授权：

    mkdir ~/.ssh       #如果不存在该文件夹需先创建，若已存在则忽略
	cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
	rm ~/id_rsa.pub    #用完就可以删掉了

如果有其他Slave节点，也要执行将Master公钥传输到Slave节点，在Slave节点上加入授权这两步。

![](http://i.imgur.com/PiUwcq1.png)

![](http://i.imgur.com/teaeUXj.png)

这样，在Master节点上就可以无密码SSH到各个Slave节点了，可在Master节点上执行如下命令 

	ssh Slave1

![](http://i.imgur.com/E6h4Yfb.png)

> 注意，操作是在Master上执行的  
> ssh登录后，终端标题以及命令符变成Slave1，此时执行的命令等同于在Slave1节点上执行，可执行exit退回到原来的Master终端

![](http://i.imgur.com/K5UCidY.png)

## 配置集群/分布式环境 ##

集群/分布式模式需要修改/usr/local/hadoop/etc/hadoop 中的5个配置文件，更多设置可点击查看官方说明，这里仅设置了正常启动所必需的设置项:  
slaves、core-site.xml、hdfs-site.xml、mapred-site.xml、yarn-site.xml。

1.文件slaves,将作为DataNode的主机名写入该文件，每行一个，默认为localhost，所以在伪分布配置时，节点即作为NameNode也作为DataNode。分布式配置可以保留localhost，也可以删掉，让Master节点仅作为NameNode使用。  

本教程让Master节点仅作为NameNode使用，因此将文件中原来的localhost删除，添加两行内容：

	Slave1
	Slave2

2.文件core-site.xml改为下面的配置

![](http://i.imgur.com/cji3EPh.png)

3.文件hdfs-site.xml，dfs.replication一般设为3，但只有2个Slave节点，所以dfs.replication的值设置为2

![](http://i.imgur.com/tiyxQrD.png)

4.文件mapred-site.xml(可能需要先重命名，默认文件夹为mapred-site.xml.template),然后配置修改如下:

![](http://i.imgur.com/8KcyUsm.png)

5.文件yarn-site.xml:

![](http://i.imgur.com/qH6FIpN.png)

配置好后，将Master上的/usr/local/Hadoop文件夹复制到各个节点上。因为之前有跑过伪分布模式，建议在切换到集群模式前先删除之前的临时文件。在Master节点上执行:   

    cd /usr/local      
	sudo rm -r ./hadoop/tmp  #删除Hadoop临时文件
	sudo rm -r ./hadoop/logs/* #删除日志文件
	tar -zcf ~/hadoop.master.tar.gz ./hadoop
	cd ~
	scp ./hadoop.master.tar.gz Slave1:/home/hadoop
	scp ./hadoop.master.tar.gz Slave2:/home/hadoop

![](http://i.imgur.com/3swH6Pw.png)

在Slave1,Slave2节点上执行：

	sudo rm -r ./usr/local/hadoop       #删掉旧的（如果存在）
	tar -zxf ~/hadoop.master.tar.gz -C /usr/local
	sudo chown -R hadoop /usr/local/hadoop

![](http://i.imgur.com/x4JpyDW.png)

![](http://i.imgur.com/w2KgYSb.png)

同样，如果有其他Slave节点，也要执行将hadoop.master.tar.gz传输到Slave节点，在Slave节点解压文件的操作。

首次启动需要在Master节点执行NameNode的格式化：

	hdfs namenode -format

![](http://i.imgur.com/X8AQkEu.png)

接着可以启动hadoop了，启动需要在Master节点上进行：

	start-dfs.sh
	start-yarn.sh
	mr-jobhistory-daemon.sh start historyserver

![](http://i.imgur.com/oisq0cW.png)

通过命令jps可以查看各个节点所启动的进程。正确的话，在Master节点上可以看到NameNode，ResourceManager、SecondaryNameNode、JobHistoryServer进程，如下图所示。

![](http://i.imgur.com/wdCffys.png)

在Slave节点上可以看到DataNode和NodeManager进程，如下图所示：

![](http://i.imgur.com/RT7cOto.png)

> 问题解决  
> 如果发生下面的问题：

	The program 'jps' can be found in the following packages:
	 * openjdk-8-jdk-headless
	 * openjdk-9-jdk-headless
	Try: sudo apt install <selected package>

![](http://i.imgur.com/WQNV0n4.png)

从Orcle（可能）安装JDK后，会出现此问题。您可以使用update-alternatives程序链接jps到标准路径目录来解决此问题。使用此命令将其固定在终端中。

	udo update-alternatives --install /usr/bin/jps jps /usr/local/java/jdk1.8.0_131/bin/jps 1

后面路径是自己java安装的路径


缺少任一进程都表示出错。另外还需要在Master节点上通过命令**hdfs dfsadmin -report**查看DataNode是否正常启动，如果Live datanodes不为0，则说明集群启动成功。

![](http://i.imgur.com/zPPop3J.png)

也可以通过Web页面看到查看DataNode和NamaNode的状态: http://master:50070/

![](http://i.imgur.com/Exh7i4e.png)

### 伪分布式，分布式配置切换时的注意事项 ###
1.从分布式切换到伪分布式时，不要忘记修改slaves配置文件
2.在两者之间切换时，若遇到无法正常启动的情况，可以删除所涉及节点的临时文件夹，这样虽然之前的数据会被删掉，但能保证集群正确启动。所以如果集群以前能启动，但后来启动不了，特别是DataNode无法启动，不妨试着删除所有节点（包括Slave节点）上的/usr/local/hadoop/tmp文件夹，在重新执行一次hdfs namenode -format ，再次启动试试。

## 执行分布式实例 ##
执行分布式实例过程与伪分布式模式一样，首先创建HDFS上的用户目录:

	hdfs dfs -mkdir -p /user/hadoop

将/usr/local/hadoop/etc/hadoop中的配置文件作为输入文件复制到分布式文件系统中： 

	hdfs dfs -mkdir input
	hdfs dfs -put /usr/local/hadoop/etc/hadoop/*.xml input

通过查看DataNode的状态（占用大小有改变），输入文件确实复制到了DataNode中，如下所示

![](http://i.imgur.com/ADr5PBw.png)

接着就可以运行MapReduce作业了:

	hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep input output 'dfs[a-z.]+'

运行时的输出信息与伪分布类似，会显示Job的进度。

可能会有点慢，但如果迟迟没有进度，比如5分钟都没看到进度，那不妨重启Hadoop再试试。若重启还不行，则很有可能是内存不足引起，建议增大虚拟机的内存，或者通过更改YARN的内存配置解决。

![](http://i.imgur.com/5jEkbdO.png)

同样可以通过web界面查看任务进度http://master:8088/cluster，在web界面点击"Tracking UI"这一列的History连接，可以看到任务的运行信息。

![](http://i.imgur.com/6WirigQ.png)

执行完毕后的输出结果：

![](http://i.imgur.com/a6MOmrU.png)

关闭Hadoop集群也是在Master节点上执行的：

	stop-yarn.sh
	stop-dfs.sh
	mr-jobhistory-daemon.sh stop historyserver

![](http://i.imgur.com/vIntTZW.png)

此外，同伪分布一样，也可以不启动YARN，但是要记得改掉mapred-site.xml的文件名


