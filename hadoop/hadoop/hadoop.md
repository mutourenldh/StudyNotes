##### 一、hadoop伪分布式运行过程

###### 1. 停止所有进程

进入/opt/module/hadoop-2.7.2目录

~~~shell
sbin/yarn-daemon.sh stop resourcemanager
sbin/yarn-daemon.sh stop nodemanager
sbin/mr-jobhistory-daemon.sh stop historyserver
sbin/hadoop-daemon.sh stop datanode
sbin/hadoop-daemon.sh stop namenode
~~~

###### 2. 清除之前的数据

在/opt/module/hadoop-2.7.2目录下删除logs和data目录中的数据

~~~shell
rm -rf logs
rm -rf data
~~~

在格式化之前，删除 datanode里面的信息（默认在/tmp目录下）我们配置在 /opt/module/hadoop-2.7.2/data/tmp目录下，删除该目录下的所有数据

清除之前的输出文件夹

~~~shell
bin/hdfs dfs -rm -R /user/ldh/output
~~~

###### 3. 格式化NameNode    

~~~shell
hadoop namenode -format
~~~

###### 4. 启动所有进程

~~~shell
sbin/hadoop-daemon.sh start namenode
sbin/hadoop-daemon.sh start datanode
sbin/yarn-daemon.sh start resourcemanager
sbin/yarn-daemon.sh start nodemanager
 sbin/mr-jobhistory-daemon.sh start historyserver
~~~

###### 5. 查看HDFS文件系统

http://192.168.23.129:50070/explorer.html#

![1533264754983](E:\GitHub\StudyNotes\hadoop\hadoop\1533264754983.png)

创建输入文件夹

~~~shell
#创建输入文件夹
[ldh@hadoop101 hadoop-2.7.2]$bin/hdfs dfs -mkdir -p /user/ldh/input
#将测试文件上传文件系统
[ldh@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -put wcinput/wc.input /user/ldh/input/
#查看上传文件是否正确
[ldh@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -cat /user/ldh/input/wc.input
#运行mapreduce程序
[ldh@hadoop101 hadoop-2.7.2]$ bin/hadoop jar
share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/ldh/input/ /user/ldh/output
#查看输出结果
[ldh@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -cat /user/ldh/output/*
#删除输出结果
[ldh@hadoop101 hadoop-2.7.2]$ hdfs dfs -rmr /user/ldh/output
~~~

浏览器查看输出结果

http://192.168.23.129:50070/explorer.html#/user/ldh/output

查看日志：http://192.168.23.129:19888/jobhistory



查看上传的文件命令

hadoop fs -lsr /



