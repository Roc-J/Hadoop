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

> 注意：在单机和伪分布的切换的时候需要把hadoop的单机和伪分布的配置文件一并修改。

#### 伪分布式模式配置 ####
1.配置/usr/local/hbase/conf/hbase-env.sh

	vi /usr/local/hbase/conf/hbase-env.sh

配置：  
* JAVA_HOME  
* HBASE_CLASSPATH  
* HBASE_MANAGES_ZK  

HBASE_CLASSPATH设置为本机Hadoop安装目录下的conf目录（即/usr/local/hadoop/conf）

![](http://i.imgur.com/zekthb8.png)	

![](http://i.imgur.com/aXSY8Ud.png)

2.配置/usr/local/hbase/conf/hbase-site.xml  

	vi /usr/local/hbase/conf/hbase-site.xml 

修改hbase.rootdir，指定HBase数据在HDFS上存储路径；将属性hbase.cluster.distributed设置为true。假设当前Hadoop集群运行在伪分布模式下，在本机上运行，且NameNode运行在9000端口。  

	<configuration>
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://localhost:9000/hbase</value>
        </property>
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</name>
        </property>
	</configuration>

![](http://i.imgur.com/q4UNv81.png)

3.接下来测试运行HBase  
（1）首先登陆ssh，之前设置了无密码登录，因此这里不需要密码，在切换目录至/usr/local/hadoop，再启动hadoop。
（如果已经启动hadoop请跳过）

	ssh localhost
	cd /usr/local/hadoop
	./sbin/start-dfs.sh
	
输入命令jps，能看到NameNode，DataNode和SecondaryNode都已经成功启动，表示hadoop启动成功。  

![](http://i.imgur.com/v2GCVHX.png)

(2)切换到/usr/local/hbase，再启动HBase命令如下  

	cd /usr/local/hbase
	bin/start-hbase.sh

启动成功，输入命令jps,可以看到hbase启动成功。  

![](http://i.imgur.com/I7Ms21Z.png)

进入shell界面：

	bin/hbase shell 

![](http://i.imgur.com/PFjvcVO.png)

4.停止HBase运行，命令如下

	bin/stop-hbase.sh

> 注意  
> 这里启动关闭Hadoop和HBase的顺序一定是: 

启动Hadoop -> 启动HBase -> 关闭HBase -> 关闭Hadoop

## 编程实践 ##
### 利用Shell命令 ###
#### HBase中创建表 ####

HBase中用create命令创建表，具体如下:

	create 'student','Sname','Ssex','Sage','Sdept','course'

命令截图：

![](http://i.imgur.com/LVqkiUT.png)

此时创建了一个"student"表，属性有:Sname,Ssex,Sage,Sdept,course。因为HBase的表中会有一个系统默认的属性作为行健，无需自行创建，默认为put命令操作中表名后第一个数据。创建完"student"表后，可通过describe命令查看"student"表的基本信息


![](http://i.imgur.com/4Rv1jW5.png)

#### HBase数据库基本操作 ####
下面主要介绍HBase的增，删，改，查操作。在添加数据时，HBase会自动为添加的数据添加一个时间戳，故在需要修改数据时，只需直接添加数据，HBase即会生成一个新的版本，从而完成"改"操作，旧的版本依旧保留，系统会定时回收垃圾数据，只留下最新的几个版本，保存的版本数可以在创建表的时候指定。

* 添加数据  
HBase中用put命令添加数据，注意:一次只能为一个表的一行数据的一个列，也就是一个单元格添加一个数据，所以直接用shell命令插入数据效率很低，在实际应用中，一般都是利用编程操作数据。

运行命令添加一行数据：

	put 'student','95001','Sname','Roc-J'

![](http://i.imgur.com/Jg9CYwg.png)

命令执行表示 在student表添加了学号为95001，名字为LiYing的一行数据，其行健为95001

	put 'student','95001','course:math','80'

命令执行：

![](http://i.imgur.com/kUSBzt5.png)

即在95001行下的course列族的math列添加了一个数据。

* 删除数据

在HBase中用delete以及deleteall 命令进行删除数据操作，区别是  
* delete用于删除一个数据，是put的反向操作   
* deleteall操作用于删除一行数据   

1.delete命令  

	delete 'student','95001','Ssex'

![](http://i.imgur.com/JjSI08Z.png)

2.deleteall命令

	deleteall 'student','95001'

![](http://i.imgur.com/h6kbOtm.png)

* 查看数据  
HBase中有两个用于查看数据的命令:   
* get 命令，用于查看表的某一个单元格数据  
* scan命令用于查看某个表的全部数据  

1、get命令 

	get 'student','9999'

![](http://i.imgur.com/PxiHvhO.png)

2.scan命令

	scan 'student'

命令执行得到的是'student'表的全部数据

![](http://i.imgur.com/FtA56hI.png)

* 删除表

删除表有两步，第一步先让该表不可用，第二步删除表

	disable 'student'
	drop 'student'

![](http://i.imgur.com/Qioig6O.png)

#### 查询表历史数据   ####
查询表的历史版本，需要两步：  
1.在创建表的时候，指定保存的版本数（假设指定为5）

	create 'employee',{NAME=>'username',VERSION=>5}

2.插入数据然后更新数据，使其产生历史版本数据
> 注意，这里插入数据和更新数据都是用put命令  

	put 'employee','88888','username','Roc-J'
	put 'employee','88888','username','Roc-J1'
	put 'employee','88888','username','Roc-J2'
	put 'employee','88888','username','Roc-J3'
	put 'employee','88888','username','Roc-J4'
	put 'employee','88888','username','Roc-J5'
	
3.查询时，指定查询的历史版本数。默认会查询出最新的数据。（有效值为1到5）  

	get 'employee'.'88888',{COLUMN=>'username',VERSIONS=>5}

![](http://i.imgur.com/GguneUN.png)

#### 退出HBase数据库表操作 ####

最后退出数据库操作，输入exit命令即可退出。
> 注意，这里退出HBase数据库是退出对数据库表的操作，而不是停止启动HBase数据库后台运行。

	exit

## Java API编程实例 ##

本实例使用Eclipse编写java程序，来对HBase数据库进行增删改查等操作。

1.启动hadoop,启动HBase

	cd /usr/local/hadoop
	./sbin/start-dfs.sh
	cd /usr/local/hbase
	./bin/start-hbase.sh

2.新建java project-HBase_Demo  
新建class-HBase_Example  

![](http://i.imgur.com/2ATUTCO.png)

3.在工程中导入外部jar包。

![](http://i.imgur.com/9QFmZOX.png)


程序代码如下：

	import java.io.IOException;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.hbase.*;
	import org.apache.hadoop.hbase.client.*;
	
	
	public class HBase_Example {
		public static Configuration configuration;
		public static Connection connection;
		public static Admin admin;
		
		//主函数中的语句请逐句执行
		public static void main(String[] args) throws IOException {
			//
			createTable("Score",new String[]{"sname","course"});
	
		}
		
		//建立链接
		public static void init(){
			configuration = HBaseConfiguration.create();
			configuration.set("hbase.rootdir", "hdfs://localhost:9000/hbase");
			try{
				connection = ConnectionFactory.createConnection(configuration);
				admin = connection.getAdmin();
			} catch(IOException e) {
				e.printStackTrace();
			}
		}
		
		//关闭链接
		public static void close(){
			try {
				if(admin!= null) {
					admin.close();
				}
				if(null != connection) {
					connection.close();
				}
			}catch (IOException e) {
				e.printStackTrace();
			}
		}
		
		/**
		 * 建表，HBase的表中有一个系统默认的属性作为主键，主键无需自行创建，默认为put命令操作中表名后的第一个数据，因此此处无需创建id列
		 * @param myTableName
		 * @param colFamily
		 * @throws IOException
		 */
		public static void createTable(String myTableName,String[] colFamily) throws IOException {
			init();
			TableName tableName = TableName.valueOf(myTableName);
			if(admin.tableExists(tableName)){
				System.out.println("table is exists!");
			}else{
				HTableDescriptor hTableDescriptor = new HTableDescriptor(tableName);
				for(String str:colFamily){
					HColumnDescriptor hColumnDescriptor = new HColumnDescriptor(str);
					hTableDescriptor.addFamily(hColumnDescriptor);
				}
				admin.createTable(hTableDescriptor);
				System.out.println("create table success");
			}
			close();
		}
	
		public static void deleteTable(String tableName) throws IOException {
			init();
			TableName tn = TableName.valueOf(tableName);
			if(admin.tableExists(tn)){
				admin.disableTable(tn);
				admin.deleteTable(tn);
			}
			close();
		}
		
		/**
		 * 查看已有表
		 * @throws IOException
		 */
		public static void listTables() throws IOException {
			init();
			HTableDescriptor hTableDescriptors[] = admin.listTables();
			for(HTableDescriptor hTableDescriptor: hTableDescriptors){
				System.out.println(hTableDescriptor.getNameAsString());
			}
			close();
		}
		
		/**
		 * 向某一行的某一列插入数据
		 * @param tableName 表名
		 * @param rowkey 行健
		 * @param colFamily 列族名
		 * @param col 列名
		 * @param val 值
		 * @throws IOException
		 */
		public static void insertRow(String tableName,String rowkey,String colFamily,String col,String val) throws IOException {
			init();
			Table table = connection.getTable(TableName.valueOf(tableName));
			Put put = new Put(rowkey.getBytes());
			put.addColumn(colFamily.getBytes(), col.getBytes(), val.getBytes());
			table.put(put);
			table.close();
			close();
		}
		
		/**
		 * 删除数据
		 * @param tableName 表名
		 * @param rowkey 行键
		 * @param colFamily 列族名
		 * @param col
		 * @throws IOException
		 */
		public static void deleteRow(String tableName,String rowkey,String colFamily,String col) throws IOException {
			init();
			Table table = connection.getTable(TableName.valueOf(tableName));
			Delete delete = new Delete(rowkey.getBytes());
			//删除制定列族的所有数据
			//delete.addFamily(colFamily.getBytes());
			//删除指定列的数据
			//delete.addColumn(colFamily.getBytes(), col.getBytes());
			
			table.delete(delete);
			table.close();
			close();
		}
		
		/**
		 * 根据行健rowkey查找数据
		 * @param tableName
		 * @param rowkey
		 * @param colFamily
		 * @param col
		 * @throws IOException
		 */
		public static void getData(String tableName,String rowkey,String colFamily,String col) throws IOException {
			init();
			Table table = connection.getTable(TableName.valueOf(tableName));
			Get get = new Get(rowkey.getBytes());
			get.addColumn(colFamily.getBytes(), col.getBytes());
			Result result = table.get(get);
			showCell(result);
			table.close();
			close();
		}
		
		/**
		 * 格式化输出
		 * @param result
		 */
		public static void showCell(Result result) {
			Cell[] cells = result.rawCells();
			for(Cell cell:cells){
				System.out.println("RowName: "+ new String(CellUtil.cloneRow(cell))+"");
				System.out.println("Timetamp: " + cell.getTimestamp()+" ");
				System.out.println("column Family: "+new String(CellUtil.cloneFamily(cell))+"  ");
				System.out.println("row Name:" + new String(CellUtil.cloneQualifier(cell))+" ");
				System.out.println("value:" + new String(CellUtil.cloneValue(cell))+" ");
				
			}
		}
		
		
	}


主函数中逐句执行，首先执行了一句创建表：

控制台输出：

	log4j:WARN No appenders could be found for logger (org.apache.hadoop.security.Groups).
	log4j:WARN Please initialize the log4j system properly.
	log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
	create table success

可以看到成功创建表

