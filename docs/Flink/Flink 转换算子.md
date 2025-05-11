
keyBy 将数据流划分为多个逻辑分区，相同的 key 的数据会分配到同一个分区（同一个子任务处理）。

keyBy 之后返回 KeyedStream

支持 sum、max、min、maxBy、minBy 等简单聚合算子
max：只会取比较字段的最大值，分比较字段保留第一次的值。
maxBy：取比较字段的最大值，同时非比较字段取最大值的这条数据的值。

reduce 两两聚合
对数据流中的元素进行两两合并，属于增量聚合，每来一条数据就计算一次。输入和输出类型必须相同。注意，逻辑分区内的第一条数据不会走聚合逻辑，因为还没有第二条数据。如果第三条数据进来，那么 reduce 的 tempData 参数就是第一条和第二条 reduce 返回的结果，t1 是第三条数据。
```java
ds  
        .keyBy(TempData::getId)  
        .reduce((ReduceFunction<TempData>) (tempData, t1) -> {  
            TempData result = new TempData();  
            result.setId(tempData.getId());  
            result.setTime(null);  
            result.setTemp(tempData.getTemp() + t1.getTemp());  
            return result;  
        })  
        .map(JSON::toJSONString)  
        .print();
```
