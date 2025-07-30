yolo 的语法
```
yolo TASK MODE ARGS
```

- TASK 任务类型，是可选的：
	- **detect** 目标检测，默认值。
	- segment 实力分割。
	- classify 图像分类。
	- pose 关键点检测。
	- obb 定向边界框检测。
- MODE 运行模式，是必须的：
	- train 训练模型。
	- val 验证模型。
	- predict 推理/预测模型。
	- export 导出模型到其它格式。
	- track 目标跟踪。
	- benchmark 基准测试。
- ARGS 参数，是可选的，形如：arg=value。常用的参数：
	- model 指定模型路径或者名称，model=yolo11n.pt。
	- conf 置信度阈值，只保留大于该值的结果，conf=0.5。
	- source 