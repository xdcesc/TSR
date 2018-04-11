# TSR
Tensorflow Speech Recognition(毕业设计日志)

### 课题：声控智能车

#### 方案：

1.采用全志A20的CubieBoard2(或树莓派3)作为硬件控制核心

2.在多角度、多位置安放麦克风，组成麦克风阵列，作为语音输入硬件

3.采用混合语音识别方案，在线服务使用灵云，本地服务则由Tensorflow作为框架编写的语音识别模型

4.其中涉及到的辅助驾驶的命令词由车辆仿真提供功能实现

5.采用 udacity/self-driving-car-sim 作为车辆仿真程序, naokishibuya/car-behavioral-cloning 作为辅助驾驶程序

#### 语音识别模型

训练材料：THCHS-30 (由清华大学语音和语言技术中心的Dong Wang, Xuewei Zhang, Zhiyong Zhang开放的语音数据集)

数据集下载地址：http://www.openslr.org/18/

由于本模型用于日常驾驶环境中，故训练时主要采用其中的 train-noise/0db/car 中包含的语音数据集

##### 待续...
