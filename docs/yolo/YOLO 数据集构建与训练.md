## 准备数据集
### 图片准备
如果原始数据是视频，需要先讲视频下载下来，然后对视频进行抽帧。脚本：
```python
from minio import Minio  
import os  
import cv2  
import logging  
import logging.config  
  
# 加载外部日志配置文件  
logging.config.fileConfig('logging.ini')  
logger = logging.getLogger()  
  
minio_client = Minio(  
    endpoint="192.168.31.157",   # MinIO 服务地址  
    access_key="xxxx",        # Access Key  
    secret_key="xxxxxxxxxxx",        # Secret Key  
    secure=False                         # 如果是 https 改为 True)  
  
bucket_name = "file-bucket"  
object_path = "file_path.txt"  
download_dir = "./videos"  
frame_dir = "./frames"  
os.makedirs(download_dir, exist_ok=True)  
os.makedirs(frame_dir, exist_ok=True)  
  
def download_video(file_path):  
    local_path = os.path.join(download_dir, file_path.replace("/", "-"))  
    try:  
        logging.info(f"开始下载: {file_path} -> {local_path}")  
        minio_client.fget_object(bucket_name, file_path, local_path)  
        logging.info(f"下载完成: {local_path}")  
        return local_path  
    except Exception as e:  
        logging.error(f"下载失败: {file_path}，错误: {e}")  
        return None  
  
def extract_frames(video_path, frame_interval=30):  
    if not video_path or not os.path.exists(video_path):  
        logging.error(f"抽帧失败，文件不存在: {video_path}")  
        return  
    try:  
        cap = cv2.VideoCapture(video_path)  
        frame_count = 0  
        saved_count = 0  
        while True:  
            ret, frame = cap.read()  
            if not ret:  
                break  
            if frame_count % frame_interval == 0:  
                img_name = f"{os.path.splitext(os.path.basename(video_path))[0]}_frame{frame_count}.jpg"  
                img_path = os.path.join(frame_dir, img_name)  
                cv2.imwrite(img_path, frame)  
                saved_count += 1  
            frame_count += 1  
        cap.release()  
        logging.info(f"{os.path.basename(video_path)} 抽帧完成，共保存 {saved_count} 张图片")  
    except Exception as e:  
        logging.error(f"抽帧失败: {video_path}，错误: {e}")  
  
if __name__ == "__main__":  
  
    with open(object_path, "r") as f:  
         for file_path in f:    
             file_path = file_path.strip()   
             if not file_path:    
                 continue    
             local_video = download_video(file_path)    
             # extract_frames(local_video)  
             
    for filename in os.listdir(download_dir):  
        file_path = os.path.join(download_dir, filename)  
        extract_frames(file_path)  
  
    logging.info("所有任务完成！")
```

日志配置文件 logging.ini：
```
[loggers]  
keys=root  
  
[handlers]  
keys=consoleHandler,fileHandler  
  
[formatters]  
keys=formatter  
  
[logger_root]  
level=INFO  
handlers=consoleHandler,fileHandler  
  
[handler_consoleHandler]  
class=StreamHandler  
level=INFO  
formatter=formatter  
args=(sys.stdout,)  
  
[handler_fileHandler]  
class=FileHandler  
level=INFO  
formatter=formatter  
args=('process.log', 'a', 'utf-8')  
  
[formatter_formatter]  
format=%(asctime)s %(levelname)s: %(message)s  
datefmt=%Y-%m-%d %H:%M:%S
```

### 打标签
安装 labelimg 工具，然后执行 labelimg 启动即可：
```
pip install labelimg
```

!!! note
    labelimg 对 Python 环境比较挑剔，新版 Python（3.10 及以上）可能运行失败或出现兼容性问题。  
    建议使用 conda 新建独立的 `labelimg` 环境（推荐 Python 3.8 或 3.9），并在该环境中安装和运行 LabelImg 进行数据处理。  
    示例：
    ```bash
    conda create -n labelimg python=3.8
    conda activate labelimg
    pip install pyqt5 lxml labelImg
    labelImg
    ```

打开目录选择图片文件夹

选择图片文件夹之后在选择一个目录，用来保存标注信息。

点击菜单 View 然后选中 Auto save mode。

点击 Change save format 按钮，切换到 yolo。

使用 w 进行框选目标，d 下一张图片，a 上一张图片。

标注完成之后，在标注信息存储目录，会有一个 classes.txt 文件，存储标签列表。第一行的编号为 0，依次递增。

每个标注图片对应一个 txt 文件，文件中的每一行对应一个目标框，格式为：
```
class_id x_center y_center width height
```

classs_id 对应 classes.txt 里面的标签编号。

x_center y_center width height 表示目标框的中心点和宽高，取值范围都是 0～1，都是相对值（除以图片的宽高）。

### 数据准备
数据标注完成之后，要对数据目录进行整理，如下：
```
my_dataset/
    ├── images/     // 存放图片
    │   ├── train/     // 训练集图片
    │   │   ├── 0001.jpg
    │   │   ├── 0002.jpg
    │   │   └── ...
    │   ├── val/     // 验证集图片
    │       ├── 1001.jpg
    │       ├── 1002.jpg
    │       └── ...
    ├── labels/     // 存放标签
    │   ├── train/     // 训练集标签文件
    │   │   ├── 0001.txt
    │   │   ├── 0002.txt
    │   │   └── ...
    │   ├── val/     // 验证集标签文件
    │       ├── 1001.txt
    │       ├── 1002.txt
    │       └── ...
    └── data.yaml
```
!!!note
	1. 需要注意的是 iamges 和 labels 目录结构必须对应，文件名须完全一致（不包括后缀）。
	2. my_dataset 需要放到 datasets 文件夹下。

data.yaml 文件内容如下
```
train: datasets/my_dataset/images/train
val: datasets/my_dataset/images/val
test: datasets/my_dataset/images/test  # 可选

names: ['cat', 'dog', 'person']  # 类别名称，索引位置对应 class_id
```

## 训练
```

```