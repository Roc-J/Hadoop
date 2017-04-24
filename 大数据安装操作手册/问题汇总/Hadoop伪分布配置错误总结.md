# Hadoop伪分布配置错误总结  #

## 作者：秦景坤 ##

小编在进行hadoop学习过程中，在对hadoop的伪分布进行配置时，对两个配置文件进行了修改。一个是core-site.xml和hdfs-site.xml。

配置后运行出现了以下错误：
    
    17/03/13 04:20:36 ERROR namenode.NameNode: Failed to start namenode.
	java.lang.IllegalArgumentException: URI has an authority component
		at java.io.File.<init>(File.java:423)
		at org.apache.hadoop.hdfs.server.namenode.NNStorage.getStorageDirectory(NNStorage.java:329)
		at org.apache.hadoop.hdfs.server.namenode.FSEditLog.initJournals(FSEditLog.java:276)
		at org.apache.hadoop.hdfs.server.namenode.FSEditLog.initJournalsForWrite(FSEditLog.java:247)
		at org.apache.hadoop.hdfs.server.namenode.NameNode.format(NameNode.java:986)
		at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1434)
		at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1559)
	17/03/13 04:20:36 INFO util.ExitUtil: Exiting with status 1
	17/03/13 04:20:36 INFO namenode.NameNode: SHUTDOWN_MSG: 
	/************************************************************
	SHUTDOWN_MSG: Shutting down NameNode at ubuntu/127.0.1.1
	************************************************************/

#### 错误发生的原因是配置文件写错了，配置文件中不要加file: ####

core-site.xml内容如下：

![](http://i.imgur.com/RNVGmWO.png)

hdfs-site.xml内容如下：
![](http://i.imgur.com/uC4phT2.png)

重新运行启动即可。

    17/03/13 04:33:37 INFO common.Storage: Storage directory /usr/local/hadoop/tmp/dfs/name has been successfully formatted.
	17/03/13 04:33:37 INFO namenode.FSImageFormatProtobuf: Saving image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
	17/03/13 04:33:37 INFO namenode.FSImageFormatProtobuf: Image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 353 bytes saved in 0 seconds.
	17/03/13 04:33:37 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
	17/03/13 04:33:37 INFO util.ExitUtil: Exiting with status 0
	17/03/13 04:33:37 INFO namenode.NameNode: SHUTDOWN_MSG: 
	/************************************************************
	SHUTDOWN_MSG: Shutting down NameNode at ubuntu/127.0.1.1
	************************************************************/





