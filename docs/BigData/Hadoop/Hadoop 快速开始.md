## 单节点安装 Hadoop
准备文件
```
OpenJDK11U-jdk_x64_linux_hotspot_11.0.26_4.tar.gz
hadoop-3.4.1.tar.gz
```

安装 JDK，环境变量配置：
```
$ cat /etc/profile.d/java.sh 
export JAVA_HOME=/usr/local/jdk-11.0.26+4
export PATH=$PATH:$JAVA_HOME/bin
```

创建 hadoop 用户
```
useradd -m -s /bin/bash hadoop
passwd hadoop
```

接下来使用 hadoop 用户进行操作

ssh 配置本机免密登录
```shell
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

验证免密登录效果：
```
ssh localhost
```

安装 Hadoop 配置环境变量（切换为 root 用户配置）：
```
# cat /etc/profile.d/hadoop.sh 
export HADOOP_HOME=/home/hadoop/hadoop-3.4.1
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

创建 Hadoop 数据存储目录：
```
/home/hadoop/hadoop-3.4.1/hadoop-files
/home/hadoop/hadoop-3.4.1/hadoop-files/namenode
/home/hadoop/hadoop-3.4.1/hadoop-files/datanode
```

进入到 Hadoop 目录，编辑配置文件：
etc/hadoop/core-site.xml，配置 HDFS 的默认访问文件地址。
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.199.134:9000</value>  <!-- 使用服务器IP -->
    </property>
</configuration>
```

etc/hadoop/hdfs-site.xml，配置文件快的副本数。
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>  <!-- 单节点副本数为1 -->
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/hadoop/hadoop-3.4.1/hadoop-files/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
		<value>file:///home/hadoop/hadoop-3.4.1/hadoop-files/datanode</value>
    </property>
</configuration>
```

etc/hadoop/hadoop-env.sh，配置 JAVA_HOME 环境变量：
```
export JAVA_HOME=/usr/local/jdk-11.0.26+4
```

格式化文件系统
```shell
hdfs namenode -format
```

启动 NameNode 和 DataNode
```
start-dfs.sh
```

测试 HDFS 功能：
```
$ hdfs dfs -ls /
$ hdfs dfs -mkdir /test
$ hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2025-03-28 18:56 /test
$ hdfs dfs -put test.txt /test
$ hdfs dfs -cat /test/test.txt
hello hadoop
```

访问地址：http://192.168.199.134:9870/

安装配置 YARN
etc/hadoop/mapred-site.xml
```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

etc/hadoop/yarn-site.xml
```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>192.168.199.134</value>
    </property>
</configuration>
```

启动 YARN
```shell
start-yarn.sh
```

访问地址：http://192.168.199.134:8088/cluster