Prometheus 会将所有采集到的样本数据以时间序列的方式存储在内存数据库中，并定时保存在硬盘上。每条时间序列通过指标名称和一组标签集合命名。
样本 sample
时间序列 time-series
指标名称 metrics name
标签集合 labelset
```
  ^
  │   . . . . . . . . . . . . . . . . . . .   node_cpu{cpu="cpu0",mode="idle"}
  │   . . . . . . . . . . . . . . . . . . .   node_cpu{cpu="cpu0",mode="system"}
  │   . . . . . . . . . . . . . . . . . . .   node_load1{}
  │   . . . . . . . . . . . . . . . . . . .  
  v
    <------------------ 时间 ---------------->
```

`__` 开头命名的标签是系统保留标签，智能在系统内部使用。
下面两种方式均可以表示同一条时间序列
```
api_total{method="POST", path="/message"}
{__name__="api_total", method="POST", path="/message"}
```


从存储上来说，所有的监控指标都是相同的，但是在业务 Prometheus 定义了 4 中不同的指标类型：Counter 计数器、Gauge 仪表盘、Histogram 直方图、Summary 摘要。

Counter 只增加不减少的计数器，比如统计接口请求数量的指标  http_request_total。一般在定义 Counter 类型的指标时建议使用 `_total` 作为后缀。

Gauge 可增可减的仪表盘，常见指标如：node_memory_MemFree（主机当前空闲的内容大小）、node_memory_MemAvailable（可用内存大小）都是Gauge类型的监控指标。

除了 Counter 和 Gauge 类型的监控指标以外，Prometheus 还定义了 Histogram 和 Summary 的指标类型。Histogram 和 Summary 主用用于统计和分析样本的分布情况。

例如，指标 prometheus_tsdb_wal_fsync_duration_seconds 的指标类型为Summary。 它记录了 Prometheus Server 中 wal_fsync 处理的处理时间，通过访问 Prometheus Server的 /metrics地址，可以获取到以下监控样本数据：
```
# HELP prometheus_tsdb_wal_fsync_duration_seconds Duration of WAL fsync.
# TYPE prometheus_tsdb_wal_fsync_duration_seconds summary
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.5"} 0.012352463
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.9"} 0.014458005
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.99"} 0.017316173
prometheus_tsdb_wal_fsync_duration_seconds_sum 2.888716127000002
prometheus_tsdb_wal_fsync_duration_seconds_count 216
```

从上面的样本中可以得知当前 Prometheus Server 进行 wal_fsync 操作的总次数为 216 次，耗时 2.888716127000002s。其中中位数（quantile=0.5）的耗时为 0.012352463，9 分位数（quantile=0.9）的耗时为 0.014458005s。

在Prometheus Server自身返回的样本数据中，我们还能找到类型为 Histogram 的监控指标 prometheus_tsdb_compaction_chunk_range_bucket。
```
# HELP prometheus_tsdb_compaction_chunk_range Final time range of chunks on their first compaction
# TYPE prometheus_tsdb_compaction_chunk_range histogram
prometheus_tsdb_compaction_chunk_range_bucket{le="100"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="400"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="1600"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="6400"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="25600"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="102400"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="409600"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="1.6384e+06"} 260
prometheus_tsdb_compaction_chunk_range_bucket{le="6.5536e+06"} 780
prometheus_tsdb_compaction_chunk_range_bucket{le="2.62144e+07"} 780
prometheus_tsdb_compaction_chunk_range_bucket{le="+Inf"} 780
prometheus_tsdb_compaction_chunk_range_sum 1.1540798e+09
prometheus_tsdb_compaction_chunk_range_count 780
```
与Summary类型的指标相似之处在于Histogram类型的样本同样会反应当前指标的记录的总数(以_count作为后缀)以及其值的总量（以_sum作为后缀）。不同在于Histogram指标直接反应了在不同区间内样本的个数，区间通过标签len进行定义。

查询时间序列
```
http_requests_total
```

等同于
```
http_requests_total{}
```

该表达式会返回指标名称为http_requests_total的所有时间序列：
```
http_requests_total{code="200",handler="alerts",instance="localhost:9090",job="prometheus",method="get"}=(20889@1518096812.326)
http_requests_total{code="200",handler="graph",instance="localhost:9090",job="prometheus",method="get"}=(21287@1518096812.326)
```

PromQL 还支持指标的标签过滤，完全匹配：= 和 !=，正则匹配：=~ 和 !~。
```
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```

上述查询默认返回指标的最新一个样本，这样的返回结构称为瞬时向量，相应的表达式称为瞬时向量表达式。

如果想要获取一段时间范围内的样本数据，则需要使用区间向量表达式，区间向量表达式需要定义事件选择范围。
```
http_requests_total{}[5m]
```
返回结果
```
http_requests_total{code="200",handler="alerts",instance="localhost:9090",job="prometheus",method="get"}=[
    1@1518096812.326
    1@1518096817.326
    1@1518096822.326
    1@1518096827.326
    1@1518096832.326
    1@1518096837.325
]
http_requests_total{code="200",handler="graph",instance="localhost:9090",job="prometheus",method="get"}=[
    4 @1518096812.326
    4@1518096817.326
    4@1518096822.326
    4@1518096827.326
    4@1518096832.326
    4@1518096837.325
]
```

还支持其它时间单位：y、w、d、h、m、s。


在瞬时向量表达式或者区间向量表达式中，都是以当前时间为基准：
```
http_request_total{} # 瞬时向量表达式，选择当前最新的数据
http_request_total{}[5m] # 区间向量表达式，选择以当前时间为基准，5分钟内的数据
```

而如果我们想查询，5分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢? 这个时候我们就可以使用位移操作，位移操作的关键字为**offset**。
可以使用offset时间位移操作：
```
http_request_total{} offset 5m
http_request_total{}[1d] offset 1d
```

标量 scalar

标量只是一个数字，没有时序。

可以通过内置函数 scalar 将单个瞬时向量转换为标量。


数学运算

```
node_memory_free_bytes_total / (1024 * 1024)
```
当瞬时向量与标量之间进行数学运算时，数学运算会依次作用与瞬时向量中的每一个样本值，从而的到一组新的时间序列。

如果是瞬时向量与瞬时向量之间进行数学运算时，过程会相对复杂一点。 例如，如果我们想根据node_disk_bytes_written和node_disk_bytes_read获取主机磁盘IO的总量，可以使用如下表达式：
```
node_disk_bytes_written + node_disk_bytes_read
```

那这个表达式是如何工作的呢？依次找到与左边向量元素匹配（标签完全一致）的右边向量元素进行运算，如果没找到匹配元素，则直接丢弃。同时新的时间序列将不会包含指标名称。 该表达式返回结果的示例如下所示：
```
{device="sda",instance="localhost:9100",job="node_exporter"}=>1634967552@1518146427.807 + 864551424@1518146427.807
{device="sdb",instance="localhost:9100",job="node_exporter"}=>0@1518146427.807 + 1744384@1518146427.807
```

PromQL支持的所有数学运算符如下所示：
- `+` (加法)
- `-` (减法)
- `*` (乘法)
- `/` (除法)
- `%` (求余)
- `^` (幂运算)

Prometheus还提供了下列内置的聚合操作符，这些操作符作用域瞬时向量。可以将瞬时表达式返回的样本数据进行聚合，形成一个新的时间序列。
- `sum` (求和)
- `min` (最小值)    
- `max` (最大值)
- `avg` (平均值)
- `stddev` (标准差)
- `stdvar` (标准方差)
- `count` (计数)
- `count_values` (对value进行计数)
- `bottomk` (后n条时序)
- `topk` (前n条时序)
- `quantile` (分位数)

使用聚合操作的语法如下：
```
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
```
其中只有`count_values`, `quantile`, `topk`, `bottomk`支持参数(parameter)。

by 是通过指定的标签进行分组，without 是剔除指定的标签，其余标签作为分组依据。
假设有以下指标：
```
http_requests_total{instance="server1", job="web", method="GET"}  100 
http_requests_total{instance="server2", job="web", method="GET"} 200 
http_requests_total{instance="server1", job="web", method="POST"} 50 
http_requests_total{instance="server2", job="web", method="POST"} 150
```


使用 by 聚合
```
sum(http_requests_total) by (method)
```

结果
```
{method="GET"}  300 
{method="POST"} 200
```


使用 without 聚合
```
sum(http_requests_total) without (instance)
```

结果
```
{job="web", method="GET"}  300 
{job="web", method="POST"} 200
```

内置函数
