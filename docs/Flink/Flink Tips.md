## Hadoop
### HDFS
`NameNode` 存储文件的元数据，记录文件属性以及文件块所在的 `DataNode` 信息。  
`DataNode` 具体存储数据及校验和。  
`SecondaryNameNode` `NameNode` 的备份节点。  
### YARN
`ResourceManager` 管理所有集群的资源。处理客户端的请求，监控 `NodeManager`，启动监控 `ApplicationMaster`。  
`NodeManager` 管理单个节点的资源。处理来自 `ResourceManager` 的命令，处理来自 `ApplicationMaster` 的命令。  
`ApplicationMaster` 负责管理单个运行任务。    
`Container` 任务运行的容器。  

## Flink
`JobManager` 对作业进行调度，分发给 `TaskManager`。  
`TaskManager` 指定任务。