[参考资料](https://zhuanlan.zhihu.com/p/655162922)

# 环境配置

1. 配置虚拟环境

   [miniconda 安装](https://www.anaconda.com/docs/getting-started/miniconda/install#quickstart-install-instructions)

   ```shell
   conda create -n pytorch python=3.12 -y

   conda env list

   conda activate pytorch
   ```

   > 在 fish 环境中安装 conda 时，可以先进入 bash，再执行 `source ~/miniconda3/bin/activate` 和 `conda init --all`。
   >
   > 在 windows 环境安装 conda 时，由于限制策略，可能无法执行 `conda activate`，参考以下博客解决：[powershell 下 conda activate 无法执行](https://blog.csdn.net/u010393510/article/details/130715238)。

2. 安装 pytorch 依赖

   [pytorch 官网](https://pytorch.org/get-started/locally/)

3. 安装 yolo

   ```shell
   git clone https://github.com/ultralytics/ultralytics.git

   cd ultralytics

   pip install -e .
   ```

# 使用 yolo

通常有两种方式使用 yolo：

- 使用 yolo 提供的 CLI 工具。

- 使用 python 脚本。

## yolo CLI

`yolo help` 获取帮助。

### 训练（train）

在自定义或预加载的数据集上微调模型。

使用 coco128 数据集以 yolov8n 预训练模型，训练 100 轮。

```shell
yolo detect train data=coco128.yaml model=yolov8n.pt epochs=100 imgsz=640
```

> 如果使用官方提供的模型和数据集，会自动从 github 上下载相关资源。
>
> 训练完成后，会提示训练后的模型保存的路径，其中 best.pt 是多轮训练中得到的最好的模型。

### 验证（val）

对训练后的模型进行校验。

```shell
yolo detect val model=./best.pt
```

```shell
val: New cache created: /home/errlst/download/datasets/coco8/labels/val.cache
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95): 100%|██████████| 1/1 [00:00<00:00,  5.08it/s]
                   all          4         17      0.761        0.9      0.908      0.678
                person          3         10       0.73        0.4      0.476      0.268
                   dog          1          1      0.891          1      0.995      0.697
                 horse          1          2      0.712          1      0.995       0.68
              elephant          1          2      0.888          1      0.995      0.531
              umbrella          1          1       0.68          1      0.995      0.995
          potted plant          1          1      0.668          1      0.995      0.895
```

- Class，类别名称，包括 all（所有类别平均）和具体类别。

- Images：验证集中包含该类别的图像数量。

- Instances：验证集中包含该类别的实例数量。

- P（Precision）：精确率，检测到的目标框中，属于真实目标的比例。精确率高 = 宁可少检，也不乱检。

- R（Recall）：召回率，真实目标中，被模型正确检测到比例。召回率高 = 宁可错检，也不漏检。

- mAP50，平均精度均值，以 Iou=0.5 为阈值计算的 AP 平均值，反映模型在宽松要求下的综合性能（更关注分类准确度）。

- mAP50-95，IoU 阈值从 0.5 到 0.95 的平均 AP 值，反映模型在严格定位要求下的综合性能（更关注定位精度）。

### 预测（predict）

```shell
yolo detect predict model=./best.pt source='https://ultralytics.com/images/bus.jpg'
```

### 导出（export）

将模型导出不同格式（如 onnx、coreml），部署到生产环境。

```shell
yolo export model=./best.pt format=onnx
```

## python
