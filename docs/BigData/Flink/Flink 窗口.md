窗口是流处理的核心概念之一，用于将无界的数据流划分为有限大小的块，以便进行聚合、统计等有界数据处理。
## 窗口的生命周期
窗口的创建时机是当属于该窗口的第一个数据到达时。
窗口的计算时机是当时间到达或者元素数量达到阈值时处罚窗口计算。
窗口的销毁时机是完成计算之后。
## 窗口的类型
按照时间划分为
滚动窗口，大小固定，不重叠。

滑动窗口，大小固定，可重叠（通过滑动步长控制）。

会话窗口，

按照计数划分为
滚动计数窗口，每 N 条数据触发一次计算。

滑动计数窗口，每 M 条数据滑动一次，统计最新 N 条数据。

## 窗口的 API 操作
指定窗口分配器，设置窗口的类型
如果开窗之前，没有进行 keyBy 操作，那么窗口逻辑只能在一个任务上运行，相当于并行度为 1；如果进行了 keyBy 操作，相当于每个 key 都定义了窗口，各自独立计算。

keyBy 的数据流使用 window 函数，没有 keyBy 的数据流使用 windowAll 函数。

```java
          // 基于时间的窗口  
//        .window(TumblingProcessingTimeWindows.of(Duration.of(200, ChronoUnit.MILLIS)))  
//        .window(TumblingEventTimeWindows.of(Duration.of(200, ChronoUnit.MILLIS)))  
//        .window(SlidingEventTimeWindows.of(Duration.of(200, ChronoUnit.MILLIS), Duration.of(1000, ChronoUnit.MILLIS)))  
//        .window(SlidingProcessingTimeWindows.of(Duration.of(200, ChronoUnit.MILLIS), Duration.of(1000, ChronoUnit.MILLIS)))  
//        .window(ProcessingTimeSessionWindows.withGap(Duration.of(200, ChronoUnit.MILLIS)))  
  
          // 基于数量的窗口  
//        .countWindow(100) // 滚动窗口 5 个元素  
//        .countWindow(5, 2) // 滑动窗口 5 个元素，2 个步长
```

解析一些 TumblingProcessingTimeWidows 的源码
```java
public static TumblingProcessingTimeWindows of(Duration size) {  
    return new TumblingProcessingTimeWindows(size.toMillis(), 0, WindowStagger.ALIGNED);  
}

private TumblingProcessingTimeWindows(long size, long offset, WindowStagger windowStagger) {  
    if (Math.abs(offset) >= size) {  
        throw new IllegalArgumentException(  
                "TumblingProcessingTimeWindows parameters must satisfy abs(offset) < size");  
    }  
  
    this.size = size;  
    this.globalOffset = offset;  
    this.windowStagger = windowStagger;  
}

@Override  
public Collection<TimeWindow> assignWindows(  
        Object element, long timestamp, WindowAssignerContext context) {  
    final long now = context.getCurrentProcessingTime();  
    if (staggerOffset == null) {  
        staggerOffset =  
                windowStagger.getStaggerOffset(context.getCurrentProcessingTime(), size);  
    }  
    long start =  
            TimeWindow.getWindowStartWithOffset(  
                    now, (globalOffset + staggerOffset) % size, size);  
    return Collections.singletonList(new TimeWindow(start, start + size));  
}

// 如果窗口的长度为 10，那么 2025-01-01 01:01:23 时间的数据，就属于 2025-01-01 01:01:20 - 2025-01-01 01:01:30 这个窗口
public static long getWindowStartWithOffset(long timestamp, long offset, long windowSize) {  
    final long remainder = (timestamp - offset) % windowSize;  
    // handle both positive and negative cases  
    if (remainder < 0) {  
        return timestamp - (remainder + windowSize);  
    } else {  
        return timestamp - remainder;  
    }  
}
```
## 简单聚合
sum、min、minBy、max、maxBy。

## 增量聚合函数
来一条数据计算一次，窗口触发的时候输出计算结果
reduce，两两聚合，来一条数据就计算一次，只有一条数据不会触发计算。注意，该函数的输入、中间状态和输出类型需要一致。
```java
ds  
        .keyBy(VehicleGps::getPlate)  
                .window(TumblingProcessingTimeWindows.of(Duration.of(200, ChronoUnit.MILLIS)))  
                        .reduce(new ReduceFunction<VehicleGps>() {  
                            @Override  
                            public VehicleGps reduce(VehicleGps vehicleGps, VehicleGps t1) throws Exception {  
                                return null;  
                            }  
                        })
```
aggregate，基本等同于 reduce，但是输入、累加状态、输出类型可以都不一致。
```java
ds  
        .keyBy(VehicleGps::getPlate)  
                .window(TumblingProcessingTimeWindows.of(Duration.of(200, ChronoUnit.MILLIS)))  
                        .aggregate(new AggregateFunction<VehicleGps, Integer, Integer>() {  
                            @Override  
                            public Integer createAccumulator() {  
                                return null;  
                            }  
  
                            @Override  
                            public Integer add(VehicleGps vehicleGps, Integer o) {  
                                return null;  
                            }  
  
                            @Override  
                            public Integer getResult(Integer o) {  
                                return null;  
                            }  
  
                            @Override  
                            public Integer merge(Integer o, Integer acc1) {  
                                return null;  
                            }  
                        })
```

增量聚合也可以拿到窗口上下文，示例：
```java
ds  
        .keyBy(VehicleGps::getPlate)  
                .window(TumblingProcessingTimeWindows.of(Duration.of(200, ChronoUnit.MILLIS)))  
                        .aggregate(new AggregateFunction<VehicleGps, Integer, Integer>() {  
                            @Override  
                            public Integer createAccumulator() {  
                                return null;  
                            }  
  
                            @Override  
                            public Integer add(VehicleGps vehicleGps, Integer o) {  
                                return null;  
                            }  
  
                            @Override  
                            public Integer getResult(Integer o) {  
                                return null;  
                            }  
  
                            @Override  
                            public Integer merge(Integer o, Integer acc1) {  
                                return null;  
                            }  
                        }, 
                        // public abstract class ProcessWindowFunction<IN, OUT, KEY, W extends Window>  
                        new ProcessWindowFunction<Integer, String, String, TimeWindow>() {  
                            @Override  
                            public void process(String s, ProcessWindowFunction<Integer, String, String, TimeWindow>.Context context, Iterable<Integer> elements, Collector<String> out) throws Exception {  
                                  
                            }  
                        });
```
这里的 ProcessWindowFunction 窗口函数只会触发一次，即 elements 只有一条数据，就是 AggregateFunction 窗口计算完成后的结果。这样，就能在增量聚合函数中拿到窗口上下文信息了。 
## 全窗口函数
数据来了不进行计算，窗口触发的时候计算并输出结果 process 
```java
ds  
    .keyBy(VehicleGps::getPlate)  
    .window(TumblingProcessingTimeWindows.of(Duration.of(200, ChronoUnit.MILLIS)))  
    .process(new ProcessWindowFunction<VehicleGps, String, String, TimeWindow>() {  
		/**
		* s keyBy 的 key
		* context 窗口上下文
		* iterable 窗口内的数据
		* collector 采集器
		*/
        @Override  
        public void process(String s, Context context, Iterable<VehicleGps> iterable, Collector<String> collector) throws Exception {  
  
        }  
    })
```

窗口上下文支持的功能
```java
public abstract W window();  
  
public abstract long currentProcessingTime();  
  
public abstract long currentWatermark();  
  
public abstract KeyedStateStore windowState();  
  
public abstract KeyedStateStore globalState();  
  
public abstract <X> void output(OutputTag<X> var1, X var2);
```

## 窗口的时间语义

事件时间

处理时间