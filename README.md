# Neurosurgeon
🥳本项目根据经典论文：Neurosurgeon: Collaborative Intelligence Between the Cloud and Mobile Edge进行实现，为DNN模型选取划分点后分别部署在云端设备和边端设备上进行协同推理(Collabrative Inference)。

论文链接🔗：https://github.com/Tjyy-1223/Neurosurgeon/blob/main/paper/Collaborative_Intelligence%20Between_the_Cloud_and_Mobile_Edge.pdf

![image-20230524094940267.png](https://github.com/Tjyy-1223/Neurosurgeon/blob/main/assets/image-20230524094940267.png?raw=true)

具体工作：

1）初步使用四种经典的DNN模型进行构建。

2）DNN模型层级特征研究：Layer latency 和 Size of output data。

2）Deployment Phase：在本地机器运行DNN层得到构建预测模型，提供模型参数。

3）Runtime Phase：实现DNN模型协同推理，具体脚本命令参考下面的描述。

**项目中提供了模型参数，可以直接clone到本地运行。**

## 项目结构

```python
Neurosurgeon
├── cloud_api.py # 模拟云端设备入口
├── deployment.py # 部署阶段
├── edge_api.py # 模拟边端设备入口
├── models # 采用的DNN模型
│   ├── AlexNet.py
│   ├── LeNet.py
│   ├── MobileNet.py
│   └── VggNet.py
├── net # 网络模块
│   ├── net_utils.py # 网络功能方法
│   ├── monitor_client.py # 带宽监视器客户端
│   └── monitor_server.py # 带宽监视器服务端
│   └── monitor_test.py # 带宽监视器测试服务
├── predictor # 预测器模块
│   ├── config # 模型参数
│   │   ├── cloud
│   │   └── edge
│   ├── dataset # 六种不同DNN层 采集数据集
│   │   ├── cloud
│   │   └── edge
│   ├── get_datasets_func.py # 读取数据集的过程
│   ├── kernel_flops.py 
│   └── predictor_utils.py # 预测器功能
└── utils # 其他工具
    ├── excel_utils.py # excel表格操作功能
    └── inference_utils.py # 协同推理功能

```

## 运行环境

```
python 3.9
speedtest-cli==2.1.3
torch==1.9.0.post2
torchvision==0.10.0a0
xlrd==2.0.1
apscheduler
```

## 项目运行

### 单任务模式

+ **一般用于评估对于DNN推理时延的性能改进：每次需要通过指令向客户端提供任务**
+ **带宽数据为每次进行推理之前 进行单次监测**

云端设备上运行 ： 可以改成服务端开放的ip和端口；-d表示云端使用cpu还是gpu：输入参数"cpu"或"cuda"

```
 python cloud_api.py -i 127.0.0.1 -p 9999 -d cpu
```

边端设备上上运行：-i和-d为服务端开放的ip和端口；-d表示边端使用cpu还是gpu：输入参数"cpu"或"cuda"

```
 # -t表示模型类型 传入参数可以为 "alex_net" "vgg_net" "le_net" "mobilenet"
 python edge_api.py -i 127.0.0.1 -p 9999 -d cpu -t vgg_net
```

**单机运行结果如下：**

**云端设备：** python cloud_api.py -i 127.0.0.1 -p 9999 -d cpu

```
successfully connection :<socket.socket fd=6, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 9999), raddr=('127.0.0.1', 64595)>
get model type successfully.
get partition point successfully.
get edge_output and transfer latency successfully.
short message , transfer latency has been sent successfully
short message , cloud latency has been sent successfully
================= DNN Collaborative Inference Finished. ===================
```

**边端设备：** python edge_api.py -i 127.0.0.1 -p 9999 -d cpu -t alex_net

```
(tjyy) tianjiangyu@tianjiangyudeMacBook-Pro Neurosurgeon % python edge_api.py -i 127.0.0.1 -p 9999 -d cpu -t alex_net
get bandwidth value : 3259.5787388244685 MB/s
best latency : 10.07 ms , best partition point : 0 - None
----------------------------------------------------------------------------------------------------------
short message , model type has been sent successfully
short message , partition strategy has been sent successfully
alex_net 在边缘端设备上推理完成 - 0.072 ms
get yes , edge output has been sent successfully
alex_net 传输完成 - 0.129 ms
alex_net 在云端设备上推理完成 - 34.621 ms
================= DNN Collaborative Inference Finished. ===================
```



### 多任务模式

+ tasks_cloud_api.py开启后：开启后分别启动两个进程（两个端口)，分别用于等待边缘端传来的任务以及定时采集带宽数据
+ tasks_edge_api.py开启后：负责从queue中逐个获取不同的DNN推理任务，并利用云边协同运算进行推理。



```

```



```

```



## 总结

Neurosurgeon是云边协同推理中的优秀框架，首次实现了将DNN模型部署在云边端设备进行协同推理。

但其也有相应的局限性：

+ 只适用于链式拓扑结构
+ 没有考虑模型的多层次结构以及各种DAG拓扑结构 - 可以参考DADS如何解决
+ 只考虑了静态网络环境下的划分状况 - 参考CAS论文如何解决

可以考虑改进的点：

+  线性回归不太准确 - 如何提升预测器性能，可以精确预测DNN层的推理时延 ✅ 因为数据采集较少
+ 已经使用多进程模式，在主任务推理之前 新开启一个进程，用来发送数据获取网络带宽 ✅ 
+ 注意通信过程中的粘包问题 ✅ 基本不会出现bug

## 交流

如果对本项目有更好的想法或者交流，可以在GitHub Issue提出问题
