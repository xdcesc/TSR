# TSR
Tensorflow Speech Recognition(毕业设计日志)

### 课题：声控智能车

#### 方案：

1.采用全志A20的CubieBoard2(或树莓派3)作为硬件控制核心

2.在多角度、多位置安放麦克风，组成麦克风阵列，作为语音输入硬件

3.采用混合语音识别方案，在线服务使用灵云，本地服务则由Tensorflow作为框架编写的语音识别模型

4.其中涉及到的辅助驾驶的命令词由车辆仿真提供功能实现

5.采用 udacity/self-driving-car-sim 作为车辆仿真程序, naokishibuya/car-behavioral-cloning 作为辅助驾驶程序

服务器系统：centos7

开发板系统：Linaro

Tensorflow版本：1.5.0

#### 语音识别模型

训练材料：THCHS-30 (由清华大学语音和语言技术中心的Dong Wang, Xuewei Zhang, Zhiyong Zhang开放的语音数据集)

数据集下载地址：http://www.openslr.org/18/

由于本模型用于日常驾驶环境中，故训练时主要采用其中的 train-noise/0db/car 中包含的语音数据集

#### 使用Google Cloud训练模型

已预先将语音数据集上传至Google Cloud云存储中(约2G)

将编写好的模型文件上传至云服务器中，并用ml-engine平台进行训练

训练文件所在目录格式

<pre><code>
  trainer/
    |-setup.py
    |-__init__.py
    |-requirements.txt
    |-trainer/
      |-index.py
      |-__init__.py
</code></pre>

上传后，在控制台输入

<pre><code>
  python setup.py install
</code></pre>

将自动安装依赖库

##### 如何连接Google

由于国内访问Google受到限制，本次使用在国外搭建的服务器做反向代理连接

最普遍的方案：shadowsocks

安装方式：pip install shadowsocks

服务端配置：

<pre><code>
  vi shadowsocks.json
  输入以下信息：
    {
      "server":"该服务器ip地址",
      "local_port":1080,
      "port_password":{
        "10015":"password" //可以配置多端口不同密码
      },
      "timeout":300,
      "method":"aes-256-cfb",
      "fast_open": false
    }
</code></pre>

运行：ssserver -c ./shadowsocks.json -d start

客户端配置：

<pre><code>
  vi shadowsocks.json
  输入以下信息：
    {
      "server":"服务端ip地址",
      "server_port":10015,  //端口
      "local_port":1080,
      "password":"password",  //该端口密码
      "timeout":600,
      "method":"aes-256-cfb"
    }
</code></pre>

运行：sslocal -c ./shadowsocks.json -d start

客户端还需安装privoxy，用来监听代理

安装方式：yum install privoxy

配置方式：

<pre><code>
  vi /etc/privoxy/config
  在最后一行插入：
    forward-socks5 / 127.0.0.1:1080 .
  注意！一定不要漏掉最后一个点(.)
  保存后在控制台执行：
    service privoxy start
  以后每次需要使用代理需先在控制台输入
    export http_proxy=http://127.0.0.1:8118
    export https_proxy=http://127.0.0.1:8118
  而后用
    wget www.google.com
  测试连接是否正常
</code></pre>

##### 安装Google Cloud SDK

在控制台输入以下信息用于添加源：

<pre><code>
  sudo tee -a /etc/yum.repos.d/google-cloud-sdk.repo << EOM
  [google-cloud-sdk]
  name=Google Cloud SDK
  baseurl=https://packages.cloud.google.com/yum/repos/cloud-sdk-el7-x86_64
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
  https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
  EOM
    
  yum install google-cloud-sdk
</code></pre>

而后安装：

<pre><code>
  yum install google-cloud-sdk
</code></pre>


##### 初始化控制台

在控制台输入

<pre><code>
  gcloud init
</code></pre>

按照步骤登陆Google账号，选择对应项目，并选择区域为us-east1(训练价格相对便宜)

再在控制台配置相关环境变量(每次进入控制台或重启后均需再次配置)以及安装相应python库

<pre><code>
  BUCKET_NAME=name
  REGION=us-east1
  DATA_DIR=gs://$BUCKET_NAME/data/wav/test/car/
  LABEL_FILE=gs://$BUCKET_NAME/data/doc/trans/test.noise.word.txt
  JOB_NAME=ASR1
  OUTPUT_PATH=gs://$BUCKET_NAME/$JOB_NAME
  gsutil cp gs://$BUCKET_NAME/trainer/index.py ./trainer/index.py
  sudo pip install librosa
</code></pre>

##### 本地测试

在控制台输入

<pre><code>
  gcloud ml-engine local train \
    --job-dir gs://$BUCKET_NAME/$JOB_NAME \
    --module-name trainer.index \
    --package-path trainer/ \
    -- \
    --data_dir $DATA_DIR \
    --label_file $LABEL_FILE \
    --how_many_training_steps 80,20 \
    --save_step_interval 10
</code></pre>

#### 正式训练

在控制台输入

<pre><code>
  gcloud ml-engine jobs submit training $JOB_NAME \
    --job-dir gs://$BUCKET_NAME/$JOB_NAME \
    --runtime-version 1.5 \
    --module-name trainer.index \
    --package-path trainer/ \
    --region $REGION \
    --scale-tier STANDARD_1 \
    -- \
    --data_dir $DATA_DIR \
    --label_file $LABEL_FILE \
    --how_many_training_steps 80,20 \
    --save_step_interval 10
</code></pre>

##### 待续...
