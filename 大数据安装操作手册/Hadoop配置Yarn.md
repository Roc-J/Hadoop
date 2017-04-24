# Hadoop配置-Yarn篇 #

## 作者：秦景坤 ##
## 时间：2017-4-23##

YARN是Hadoop2.x中的内容（伪分布式不启动YARN也可以，一般不会影响程序执行）

有的读者可能会疑惑，怎么启动Hadoop后，见不到书上所说的JobTracker和TaskTracker，这是因为新版的Hadoop使用了新的MapReduce框架（MapReduce V2,也称为YARN,Yet Another Resource Negotiator）。

YARN是从MapReduce中分离出来的，负责资源管理与任务调度。YRAN运行于MapReduce之上，提供了高可用性、高扩展性。

上一篇中通过**./sbin/start-dfs.sh**启动Hadoop，仅仅是启动了MapReduce环境，可以启动YARN,让YARN来负责资源管理和任务调度。

首先修改配置文件mapred-site.xml，这边需要先进行重命名。

	mv ./etc/hadoop/mapred-site.xml.template ./etc/hadoop/mapred-site.xml

然后再进行编辑，同样使用vi进行编辑 vi ./etc/hadoop/mapred-site.xml

![](http://i.imgur.com/SdzkwD4.png)

接着修改配置文件yarn-site.xml:

![](http://i.imgur.com/28g6LHR.png)

然后就可以启动YARN（需要先执行./sbin/start-dfs.sh）

![](http://i.imgur.com/VlaqPoW.png)

启动YARN之后，运行实例的方法还是一样的，仅仅是资源管理方式，任务调度不同。观察日志信息可以发现，不启用YARN时，是"mapred.LocalJobRunner"在跑任务，启用YARN之后，是"mapred.YARNRunner"在跑任务。启动YARN有个好处是可以通过web界面查看任务的运行情况。

但YARN主要是为集群提供更好的资源管理和任务调度，然而这在单机上体现不出价值，反而会使程序跑的稍慢些。因此在单机上是否开启YARN就看实际情况了。

> 如果不想启动YARN，务必把配置文件mapred-site.xml重命名，改成mapred-site.xml.template,需要时改回来就行。否则在该配置文件存在，而未开启YARN的情况下，运行程序会提示"Retrying connect to server:0.0.0.0/0.0.0.0:8032"的错误，这也是为何该配置文件初始文件名为mapred-site.xml.template

同样的，关闭YARN的脚本如下：

	./sbin/stop-yarn.sh
	./sbin/mr-jobhistory-daemon.sh stop historyserver

