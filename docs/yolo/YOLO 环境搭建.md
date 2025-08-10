## 环境安装
**安装 Miniconda**  
Mac 下载安装包，一路确认，默认安装目录：/opt/miniconda3  
!!!note
	打开终端，最前面会有 (base) 就说明安装成功。此时终端默认激活 base 环境，执行命令取消默认激活 `conda config --set auto_activate_base false`。

执行命令 `conda --version` 进行验证。

创建 ultralytics 环境
```
conda create -n ultralytics python=3.12
```

查看环境列表
```
conda env list
```

切换到 ultralytics 环境
```
conda activate ultralytics
```

**安装 PyTorch**
```
pip3 install torch torchvision torchaudio
```

验证 PyTorch
```python
import torch
print(torch.rand(5, 3))

// 输出如下内容即可
tensor([[0.0279, 0.0617, 0.8465],
        [0.5971, 0.7788, 0.1976],
        [0.9156, 0.4530, 0.5232],
        [0.8783, 0.4915, 0.4486],
        [0.4522, 0.8738, 0.2487]])
```

**源码安装 Ultralytics**  
直接下载 Release 版本的源码 https://github.com/ultralytics/ultralytics/releases/tag/v8.3.169

解压，进入到目录，执行安装命令 `pip install -e .`

验证安装，直接输入命令 `yolo` 即可。

**安装 VsCode**
安装 Python 和 Jupyter 两个插件。

## 目标预测
使用命令
```
yolo predict model=yolo11n.pt source=ultralytics/assets/bus.jpg
```

使用脚本
```python
from ultralytics import YOLO

model = YOLO('yolo11n.pt')

results = model(source="/Users/jacky/Movies/ASU_1658.MOV", show=True, conf=0.4, save=True)
```