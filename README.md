
# Efficient and Accurate Arbitrary-Shaped Text Detection with Pixel Aggregation Network
# Paddle-PANet


# 目录
- [结果对比](#结果对比)
- [论文介绍](#论文介绍)
- [快速安装](#快速安装)
## Introduction
```
@inproceedings{wang2019efficient,
  title={Efficient and Accurate Arbitrary-Shaped Text Detection with Pixel Aggregation Network},
  author={Wang, Wenhai and Xie, Enze and Song, Xiaoge and Zang, Yuhang and Wang, Wenjia and Lu, Tong and Yu, Gang and Shen, Chunhua},
  booktitle={Proceedings of the IEEE International Conference on Computer Vision},
  pages={8440--8449},
  year={2019}
}
```

# 结果对比

[CTW1500](dataset/README.md) 

| Method | Backbone | Fine-tuning  | Config | Precision (%) | Recall (%) | F-measure (%) | Model | Log|
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |:-: |
| mmocr_PANet | Resnet18 | N | [ctw_config](https://github.com/open-mmlab/mmocr/blob/main/configs/textdet/panet/README.md) |  77.6 | 83.8 | 80.6 | -- | -- |
| PAN (paper) | ResNet18 | N | [config](https://github.com/whai362/pan_pp.pytorch/blob/master/config/pan/pan_r18_ctw.py)| 84.6 | 77.7 | 81.0 | - | - |
| PaddlePaddle_PANet | ResNet18 | N  | [panet_r18_ctw.py](https://github.com/JennyVanessa/Paddle-PANet/blob/main/config/pan/pan_r18_ctw.py) | 84.51 | 78.62 | **81.46** | [Model](https://github.com/JennyVanessa/Paddle-PANet/tree/main/checkpoints/pan_r18_ctw_train) |  [Log](https://github.com/JennyVanessa/Paddle-PANet/blob/main/trainlog.txt)

# 论文介绍


## 背景简介
场景文本检测是场景文本阅读系统的重要一步，随着卷积神经网络的快速发展，场景文字检测也取得了巨大的进步。尽管如此，仍存在两个主要挑战，它们阻碍文字检测部署到现实世界的应用中。第一个问题是速度和准确性之间的平衡。第二个是对任意形状的文本实例进行建模。最近，已经提出了一些方法来处理任意形状的文本检测，但是它们很少去考虑算法的运行时间和效率，这可能在实际应用环境中受到限制。

之前在CVPR 2019上发的PSENet是效果非常好的文本检测算法，但是整个网络的后处理复杂，导致其运行速度很慢。于是PSENet算法的原班作者提出了PAN网络，使其在不损失精度的情况下，极大加快了网络inference的速度，因此也可以把PAN看做是PSENet V2版本。

## 网络结构
<center><img src="https://ai-studio-static-online.cdn.bcebos.com/ce9767d24b054c3ebf51621a8d14ee922e96e5d9a0c64f84be60f4e7f27695eb"， height=70%, width=70%></center>
上图为PAN的整个网络结构，网络主要由Backbone + Segmentation Head（FPEM + FFM） + Output(Text Region、Kernel、Similarity Vector)组成。

本文使用ResNet-18作为PAN的默认Backbone，并提出了低计算量的Segmentation Head(FPFE + FFM)以解决因为使用ResNet-18而导致的特征提取能力较弱，特征感受野较小且表征能力不足的缺点。

此外，为了精准地重建完整的文字实例(text instance)，提出了一个可学习的后处理方法——像素聚合法（PA），它能够通过预测出的相似向量来引导文字像素聚合到正确的kernel上去。

下面将详细介绍一下上面的各个部分。

## Backbone
Backbone选择的是resnet18, 提取stride为4,8,16,32的conv2,conv3,conv4,conv5的输出作为高低层特征。每层的特征图的通道数都使用1*1卷积降维至128得到轻量级的特征图Fr。

## Segmentation Head
PAN使用resNet-18作为网络的默认backbone，虽减少了计算量，但是backbone层数的减少势必会带来模型学习能力的下降。为了提高效率，作者在 resNet-18基础上提出了一个低计算量但可高效增强特征的分割头Segmentation Head。它由两个关键模块组成：特征金字塔增强模块（Feature Pyramid Enhancement Module，FPEM）、特征融合模块（Feature Fusion Module，FFM）。

### <font face="Times new roman"> FPEM </font>
Feature Pyramid Enhancement Module(FPEM)，即特征金字塔增强模块。FPEM呈级联结构且计算量小，可以连接在backbone后面让不同尺寸的特征更深、更具表征能力，结构如下：
<center><img src="https://ai-studio-static-online.cdn.bcebos.com/723fa8c02d2240e9a31aed857040d7ddc09758974baf47aca11574f5ea7fcb05"， height=50%, width=50%></center>
FPEM是一个U形模组，由两个阶段组成，up-scale增强、down-scale增强。up-scale增强作用于输入的特征金字塔，它以步长32,16,8,4像素在特征图上迭代增强。在down-scale阶段，输入的是由up-scale增强生成的特征金字塔，增强的步长从4到32，同时，down-scale增强输出的的特征金字塔就是最终FPEM的输出。
FPEM模块可以看成是一个轻量级的FPN，只不过这个FPEM计算量不大，可以不停级联以达到不停增强特征的作用。


### <font face="Times new roman"> FFM </font>
Feature Fusion Module(FFM)模块用于融合不同尺度的特征，其结构如下：


## Loss
![image](https://user-images.githubusercontent.com/39580716/139774753-5e47638a-2c66-4a72-aa93-c725a48c91d8.png)


# 快速安装
## Recommended environment
```
Python 3.6+
paddlepaddle-gpu 2.0.2
nccl 2.0+
mmcv 0.2.12
editdistance
Polygon3
pyclipper
opencv-python 3.4.2.17
Cython
```


### Install env
Install paddle following the official [tutorial](https://www.paddlepaddle.org.cn/documentation/docs/zh/install/index_cn.html).
```shell script
pip install -r requirement.txt
./compile.sh
```
## Dataset
> Please refer to [dataset/README.md](dataset/README.md) for dataset preparation.

### Pretrain Backbone 

> download resent18 pre-train model in `pretrain/resnet18.pdparams`

> [pretrain_resnet18](https://pan.baidu.com/s/1zwmcaAfabZ8fT-KoisbR3w)
> password: j5g3

## Training
```shell script
CUDA_VISIBLE_DEVICES=0,1,2,3 python dist_train.py ${CONFIG_FILE}

```
For example:
```shell script
CUDA_VISIBLE_DEVICES=0,1,2,3 python dist_train.py config/pan/pan_r18_ctw.py
#checkpoint continue
python3.7 dist_train.py config/pan/pan_r18_ctw_train.py --nprocs 1 --resume checkpoints/pan_r18_ctw_train
```


## Evaluation 
The evaluation scripts of CTW 1500 dataset. [CTW](https://1drv.ms/u/s!Aplwt7jiPGKilH4XzZPoKrO7Aulk)

Text detection
```shell script
./start_test.sh
```



## License
This project is developed and maintained by [IMAGINE Lab@National Key Laboratory for Novel Software Technology, Nanjing University](https://cs.nju.edu.cn/lutong/ImagineLab.html).


This project is released under the [Apache 2.0 license](https://github.com/whai362/pan_pp.pytorch/blob/master/LICENSE).
