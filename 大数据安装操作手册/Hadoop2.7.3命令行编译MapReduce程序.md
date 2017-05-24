# Hadoop2.7.3 -使用命令行编译打包运行自己的MapReduce程序 #

作者：秦景坤
GitHub：https://github.com/Roc-J
CSDN博客：http://blog.csdn.net/qjk19940101

本文以Hadoop2.7.3单机模式环境下的WordCount实例来介绍2.7.3版本中如何编辑自己的MapReduce程序

实例中至少有3个jar包：

* $HADOOP_HOME/share/hadoop/common/hadoop-common-2.7.3.jar  
* $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.3.jar  
* $HADOOP_HOME/share/hadoop/common/lib/commons-cli-1.2.jar  

> 通过命令**hadoop classpath**可以得到运行Hadoop程序所需的全部classpath信息

![](http://i.imgur.com/drUL3WZ.png)

## 编译，打包Hadoop MapReduce程序 ##
可以将Hadoop的classpath信息添加到CLASSPATH变量中，在/etc/profile中增加如下几行：

	export HADOOP_HOME=/usr/local/hadoop
	export CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath):$CLASSPATH

执行source /etc/profile使变量生效，接着就可以通过javac命令编辑WordCount.java  

1.首先建立一个工作目录：  
	
	mkdir WordCount
	cd WordCount
	
将WordCount源码放置该目录下，进行编译

	javac WordCount.java

![](http://i.imgur.com/Ii9FU5K.png)

接着把.class文件打包成jar，才能在Hadoop中运行


	jar -cvf WordCount.jar ./WordCount*.class

打包完成后，创建几个输入文件来测试一下：


	mkdir input
	echo "today is a good day" > ./input/test1
	echo "today i am happy,so how about the other day" > ./input/test2

![](http://i.imgur.com/qdGKfqW.png)



> 如果读者的Hadoop是单机模式，请跳过此步骤。如果读者的Hadoop环境已经配置成伪分布，那么需要执行以下命令，小编的是伪分布式
> 保证已经开启Hadoop服务

	start-all.sh，相当于执行start-dfs.sh and start-yarn.sh

可以到hadoop安装目录下开启hadoop服务

	# 把本地文件上传到伪分布式HDFS上
	/usr/local/hadoop/bin/hadoop fs -put ./input input

开始执行，命令行输入

	/usr/local/hadoop/bin/hadoop fs -put ./input input

伪分布正确的执行结果如下

![](http://i.imgur.com/80HazE9.png)

> 如果读者已经将Hadoop的bin目录添加到环境变量，那么每一句命令的/usr/local/hadoop/bin/都可以不写

## WordCount.java源码 ##

	import java.io.IOException;
	import java.util.Iterator;
	import java.util.StringTokenizer;
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.io.IntWritable;
	import org.apache.hadoop.io.Text;
	import org.apache.hadoop.mapreduce.Job;
	import org.apache.hadoop.mapreduce.Mapper;
	import org.apache.hadoop.mapreduce.Reducer;
	import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
	import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
	import org.apache.hadoop.util.GenericOptionsParser;
	 
	public class WordCount {
	    public WordCount() {
	    }
	 
	    public static void main(String[] args) throws Exception {
	        Configuration conf = new Configuration();
	        String[] otherArgs = (new GenericOptionsParser(conf, args)).getRemainingArgs();
	        if(otherArgs.length < 2) {
	            System.err.println("Usage: wordcount <in> [<in>...] <out>");
	            System.exit(2);
	        }
	 
	        Job job = Job.getInstance(conf, "word count");
	        job.setJarByClass(WordCount.class);
	        job.setMapperClass(WordCount.TokenizerMapper.class);
	        job.setCombinerClass(WordCount.IntSumReducer.class);
	        job.setReducerClass(WordCount.IntSumReducer.class);
	        job.setOutputKeyClass(Text.class);
	        job.setOutputValueClass(IntWritable.class);
	 
	        for(int i = 0; i < otherArgs.length - 1; ++i) {
	            FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
	        }
	 
	        FileOutputFormat.setOutputPath(job, new Path(otherArgs[otherArgs.length - 1]));
	        System.exit(job.waitForCompletion(true)?0:1);
	    }
	 
	    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
	        private IntWritable result = new IntWritable();
	 
	        public IntSumReducer() {
	        }
	 
	        public void reduce(Text key, Iterable<IntWritable> values, Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
	            int sum = 0;
	 
	            IntWritable val;
	            for(Iterator i$ = values.iterator(); i$.hasNext(); sum += val.get()) {
	                val = (IntWritable)i$.next();
	            }
	 
	            this.result.set(sum);
	            context.write(key, this.result);
	        }
	    }
	 
	    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
	        private static final IntWritable one = new IntWritable(1);
	        private Text word = new Text();
	 
	        public TokenizerMapper() {
	        }
	 
	        public void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context) throws IOException, InterruptedException {
	            StringTokenizer itr = new StringTokenizer(value.toString());
	 
	            while(itr.hasMoreTokens()) {
	                this.word.set(itr.nextToken());
	                context.write(this.word, one);
	            }
	 
	        }
	    }
	}
	 