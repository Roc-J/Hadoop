# 分布式文件系统HDFS学习指南 #

本文参考厦门大学数据库实验室，表示感谢 [http://dblab.xmu.edu.cn/blog/290-2/](http://dblab.xmu.edu.cn/blog/290-2/)

## 作者：秦景坤 ##
## 时间：2017-4-26 ##
Hadoop分布式文件系统（Hadoop Distributed File System ,HDFS）是Hadoop核心组件之一，如果已经安装Hadoop，那么其中就已经包含了HDFS组件，无需另外安装。

下面介绍Linux操作系统中关于HDFS文件操作的常用shell命令，利用web界面查看和管理Hadoop文件系统，以及利用Hadoop提供的Java API进行基本的文件操作。

## 一、启动Hadoop ##

在学习HDFS编程实现，需要启动Hadoop，执行如下命令：

	cd /usr/local/hadoop
	./sbin/start-dfs.sh

## 二、HDFS编程 ##

### 1.利用shell命令操作 ###

Hadoop支持很多shell命令，其中fs是HDFS最常用的命令，利用fs可以查看HDFS文件系统的目录结构、上传和下载数据、创建文件。

> 实际上有三种shell命名方式。  
> 1.hadoop fs  
> 2.hadoop dfs  
> 3.hdfs dfs  

hadoop fs 适用于任何不同的文件系统，比如本地文件系统和HDFS文件系统   
hadoop dfs 只能适用于HDFS文件系统  
hdfs dfs 跟hadoop dfs的命名作用一样，也只能适用于HDFS文件系统

可以在终端输入如下命令，查看fs总共支持了哪些命令

	hadoop fs

![](http://i.imgur.com/aYarGwv.png)

在终端输入如下命令，可以查看具体哪个命令怎么用  

例如，查看put命令如何使用，可以输入如下的命令

	hadoop fs -help put
	
![](http://i.imgur.com/hKfQkAC.png)

### (2)利用HDFS的web界面管理 ###

打开Linux浏览器，输入 http://localhost:50070，即可看到HDFS的Web管理界面

![](http://i.imgur.com/2uvavaF.png)

### (3)利用Java API与HDFS进行交互 ###

Hadoop不同的文件系统通过调用java API进行交互，上面介绍的shell命令，本质上是Java API的应用。下面提供了Hadoop官方的Hadoop API文档，想要深入学习Hadoop，可以访问如下网址：

[http://hadoop.apache.org/docs/stable/api/](http://hadoop.apache.org/docs/stable/api/ "Hadoop API文档")

利用Java API进行交互，需要安装软件Eclipse(安装教程不综述)

**在Eclipse中创建项目**
第一次打开Eclipse，需要填写workspace（工作空间），用来保存程序所在的位置，这里按照默认，不需要改动，如下如：

![](http://i.imgur.com/byMM0M3.png)

点击OK按钮，进入Eclipse软件。开始创建项目，选择顶部菜单File->New->Java Project

![](http://i.imgur.com/fHMjzfE.png)

输入项目名称，本教程输入的项目名称"Dblab"，其他不用改写，点击"Finish"按钮即可。


#### 项目加载所需要的jar包 ####

#### 如何获取Java API ####
API所在的jar包都在已经安装好的hadoop文件夹里，路径:/usr/local/hadoop/share/hadoop 如果你安装的hadoop不在此目录下，请找到jar包

* 加载上面路径common文件夹下:  
* lib所有的jar包和hadoop-common-2.7.3.jar  
* 加载上面路径hdfs文件夹下：  
* lib所有的jar包和hadoop-hdfs-2.7.3.jar   
* 在所有项目中加载jar包，具体操作如下:在所选的Eclipse项目(Dblab)上右键点击-->弹出菜单中选择Properties-->java Build Path-->Libraries-->Add External JARS

![](http://i.imgur.com/5j36kHC.png)

* 实例：利用hadoop的java api检测伪分布式文件系统HDFS上是否存在某个文件，写入文件，读取文件。

第一个编写程序，判断文件是否存在：

	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.Path;
	
	
	public class File_exists {
		public static void main(String[] args) {
			try {
				String filename = "test";
				Configuration conf = new Configuration();
				conf.set("fs.defaultFS", "hdfs://localhost:9000");
				conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
				FileSystem fs = FileSystem.get(conf);
				if(fs.exists(new Path(filename))){
					System.out.println("file exists.");
				}else {
					System.out.println("file not exists.");
				}
				fs.close();
			} catch(Exception e) {
				e.printStackTrace();
			}
		}
	}

程序运行之后

![](http://i.imgur.com/PUYD1tJ.png)

可以看到文件不存在。

第二个编程序写入文件

	import org.apache.hadoop.conf.Configuration;  
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.FSDataOutputStream;
	import org.apache.hadoop.fs.Path;
	 
	public class Chapter3 {    
	        public static void main(String[] args) { 
	                try {
	                        Configuration conf = new Configuration();  
	                        conf.set("fs.defaultFS","hdfs://localhost:9000");
	                        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
	                        FileSystem fs = FileSystem.get(conf);
	                        byte[] buff = "Hello world".getBytes(); // 要写入的内容
	                        String filename = "test"; //要写入的文件名
	                        FSDataOutputStream os = fs.create(new Path(filename));
	                        os.write(buff,0,buff.length);
	                        System.out.println("Create:"+ filename);
	                        os.close();
	                        fs.close();
	                } catch (Exception e) {  
	                        e.printStackTrace();  
	                }  
	        }  
	}
	
![](http://i.imgur.com/9Yhhp11.png)

第三个程序编写读取文件

	import java.io.BufferedReader;
	import java.io.InputStreamReader;
	 
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.fs.FSDataInputStream;
	 
	public class Chapter3 {
	        public static void main(String[] args) {
	                try {
	                        Configuration conf = new Configuration();
	                        conf.set("fs.defaultFS","hdfs://localhost:9000");
	                        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
	                        FileSystem fs = FileSystem.get(conf);
	                        Path file = new Path("test"); 
	                        FSDataInputStream getIt = fs.open(file);
	                        BufferedReader d = new BufferedReader(new InputStreamReader(getIt));
	                        String content = d.readLine(); //读取文件一行
	                        System.out.println(content);
	                        d.close(); //关闭文件
	                        fs.close(); //关闭hdfs
	                } catch (Exception e) {
	                        e.printStackTrace();
	                }
	        }
	}


![](http://i.imgur.com/qT6Vu4Q.png)


### 导出jar包运行 ###

1.点击Eclipse顶部菜单选项"File"，选择"export"

![](http://i.imgur.com/q3039cc.png)

2.选择"Runnable JAR File"，然后下一步next

![](http://i.imgur.com/05MgiP3.png)

3.在终端输入命令：

	java jar File_Read.jar

![](http://i.imgur.com/sb5UbeB.png)

