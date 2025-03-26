## 节点介绍
### HDFS
`NameNode` 存储文件的元数据，记录文件属性以及文件块所在的 `DataNode` 信息。

`DataNode` 具体存储数据及校验和。  
`SecondaryNameNode` `NameNode` 的备份节点。  
### YARN
`ResourceManager` 管理所有集群的资源。处理客户端的请求，监控 `NodeManager`，启动监控 `ApplicationMaster`。  
`NodeManager` 管理单个节点的资源。处理来自 `ResourceManager` 的命令，处理来自 `ApplicationMaster` 的命令。  
`ApplicationMaster` 负责管理单个运行任务。    
`Container` 任务运行的容器。  



## 单节点安装 Hadoop
JDK OpenJDK8U-jdk_x64_linux_hotspot_8u442b06.tar.gz，安装并配置 JAVA_HOME 环境变量。
同时 JAVA_HOME 的环境变量要在 etc/hadoop/hadoop-env.sh 配置文件中配置。
Hadoop [hadoop-3.4.1.tar.gz](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/stable/hadoop-3.4.1.tar.gz)
ssh 需要保证服务器的 ssh 服务是启动的。

环境变量配置
```
$ cat /etc/profile.d/hadoop.sh 
export HADOOP_HOME=/usr/local/hadoop-3.4.1
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

$ cat /etc/profile.d/java.sh 
export JAVA_HOME=/usr/local/jdk-11.0.26+4
export PATH=$PATH:$JAVA_HOME/bin
```

创建 hadoop 用户
```
useradd -m -s /bin/bash hadoop

# 授权 Hadoop 目录
chown -R hadoop:hadoop /usr/local/hadoop-3.4.1
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


进入到 Hadoop 目录，编辑配置文件：
etc/hadoop/core-site.xml，配置 HDFS 的默认访问文件地址。
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.199.134:9000</value>  <!-- 使用服务器IP -->
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
	<value>/usr/local/hadoop-3.4.1/hadoop-files/tmp</value>
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
        <value>file:///usr/local/hadoop-3.4.1/hadoop-files/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
		<value>file:///usr/local/hadoop-3.4.1/hadoop-files/datanode</value>
    </property>
</configuration>
```


格式化文件系统
```shell
bin/hdfs namenode -format
```

启动 NameNode 和 DataNode
```
sbin/start-dfs.sh
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
sbin/start-yarn.sh
```

访问地址：http://192.168.199.134:8088/cluster