## 目标

训练论文"A Lightened CNN for Deep Face Representation"中的网络。

## 数据

CASIA-WebFace 可在[这里](http://www.cbsr.ia.ac.cn/english/CASIA-WebFace-Database.html)申请。

### 预处理

获得CASIA-Webface数据集之后，使用[tools](https://github.com/Tonyfy/LCNN_TRAIN/tree/master/tools)中的：

* 脚本`addLabeltopic.py`将图片的label置为0-10574，并将label加入图片的名字中，如`0_001.jpg`。
* 脚本`getallfilesInOnedir.py`将所有图片复制到某一级目录下，如image/
* 使用`code_point`中的工具对人脸图片进行标点，其中bbox.txt中指定了固定的人脸位置，固定face bounding box 是(65 178 78 191)。
* 可使用`show_result.m`脚本对标点结果进行可视化。
* 根据每个图片样本的标点信息将人脸进行旋转和裁剪，使得样本标准化。
经过上述操作，可以得到10575人的494414张标准化人脸图块。

### 使用caffe，训练模型

* 生成lmdb数据
* 编写train_test.prototxt和solver.prototxt
* 开始训练！

## 网络配置
 
网络结构可以在此[可视化](http://ethereon.github.io/netscope/#/gist/c64671dc9359cc82c8baa02b1e1e5ec5)。

详细配置查看查看我的github项目中[prototxt/](https://github.com/Tonyfy/LCNN_TRAIN/tree/master/prototxt)中的[LCNN_solver.prototxt](https://github.com/Tonyfy/LCNN_TRAIN/blob/master/prototxt/LCNN_solver.prototxt)和[LCNN_train_test.prototxt](https://github.com/Tonyfy/LCNN_TRAIN/blob/master/prototxt/LCNN_train_test.prototxt)。

对学习率的设置
初始学习率设置可以依次尝试｛0.1，0.03，0.01，0.003，0.001，..｝，选择训练不会出现发散的学习率进行初始化。一般来讲，batchsize越大，初始的学习率可以大一些。

## 训练记录

训练过程中，根据log信息解析到train_loss,  test accurate 等变化。绘制如下。
（batch size:64 ，mirror，初始lr:0.01）

![train loss-iter][train-loss-iter]
![test accurate-iter][test-accu-iter]

[train-loss-iter]:accurate/LCNN_trainloss_iter.png
[test-accu-iter]:accurate/LCNN_testaccu_iter.png

根据检测曲线判定模型收敛到稳定的性能。

## 模型部署

### python

参考```demo/demo.py```，输入标准化之后的人脸，进行特征提取。并进行人脸验证。

通过LCNN模型提取出```demo/*.png```人脸的特征是256维的高维向量。程序对输入的3张人脸进行特征，分别计算出两两之间的余弦相似度。据此可进行人脸验证。如下图

![f][f]  
---------------
![f1][f1]
[f]:demo/f.jpg 
[f1]:demo/f1.jpg

## 测试记录

测试数据集：[lfw](http://vis-www.cs.umass.edu/lfw/)。取官方给出的[pairs.txt](http://vis-www.cs.umass.edu/lfw/pairs.txt)进行测试。保证pairs.txt中的图片不会出现在训练集中。

|数据组织|数量|描述|
|---|---|---|
|Intra-pair|3000|每一个pair是同一个人物的两张不同场景下的人脸图片对|
|Extra-pair|3000|每一个pair是不同人物的人脸图片对|

使用pairs.txt中给出的6000对人脸对测试本模型的准确度。按照标准的ROC评测方案，并在True Positive Rate与False Reject Rate相当时，取出准确率用于表征模型的准确率。将不同迭代次数时的测试结果记录在下表中。

|model|EER(%)|th|err_num(pairs)|
|----|----|----|----|
|iter_3560000|97.6|0.2905|146|

在lfw上测试的roc曲线为：

![lfw-roc][roc]
[roc]:accurate/lfw-roc.jpg

部分判断错误的人脸pair 如下图所示
![showerr][err]
[err]:accurate/showerr.png



## 计算环境

NVIDIA GTX1070

## 常见应用

在人脸验证环节取得较高准确率之后，可基于此开发刷脸签到和视频监控等应用程序，如[《手把手教你开发基于深度学习的人脸识别【考勤/签到】系统》](http://blog.csdn.net/shixiangyun2/article/details/51585004).
