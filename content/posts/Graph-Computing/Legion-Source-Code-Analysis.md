---
title: Legion Source Code Analysis
subtitle:
description:
keywords:
summary:
license:
date: 2024-03-05T20:17:06+08:00
lastmod: 2024-03-09T21:55:18+08:00
tags:
categories:
  - Graph-Computing
weight: 0

password:
message:

draft: false
repost:
  enable: false
  url:

hiddenFromHomePage: false
hiddenFromSearch: false

comment: false
lightgallery: force
---

## 项目文件分类

### Storage
2. GPU_Graph_Storage.[cuh]
6. GPUGraphStore.[cu/cuh]

> 这两个的区别可能在于下面的是图的逻辑存储结构，上面的是物理存储结构(因为涉及到了CSR)

3. GPU_Memory_Graph_Storage.[cu]
4. GPU_Node_Storage.[cuh]


### Cache
5. GPUCache.[cu/cuh]

1. GPU_IPC_Service.[cu/h]
7. GPUMemoryPool.[cu/cuh]
8. hashmap.[h]
9. helper_multiprocess.[cpp/h]
10. Kernels.[cu/cuh]
11. Operator.[cu/h]
12. Server.[cu/h]

## 初始化
GPUServer->Initialize
    获取了性能计数器以便获取 $N_{TSUM}$ , 供 Cost Model 使用
    ```cpp
    monitor_ = new PCM_Monitor(); // PCM:Performance Counter Monitor,性能计数器监视器
    monitor_->Init();
    ```
  - GPUGraphStore->Initialize
    1. 数据集的初始化
    1.1 把数据集分为训练、验证 和 测试 集
    1.2 把数据加载到 GPU 的显存中

    2. Cache 部分的初始化
  - GPURunner->Initialize
    1. 执行器初始化，没个gpu都对应一个执行器
    创建了2个gpu stream






## 系统执行参数
在`main.cpp`的`server->Initialize(atoi(argv[1]));`环节，会执行`GPUGraphStore.cu`中定义的`void GPUGraphStore::Initialze()`，其中包含一个步骤`ReadMetaFIle()`，这个步骤读取了`./meta_config`文件中提前写入的以下参数：

```text
Dataset path
Raw Batchsize

Graph nodes num
Graph edges num

Feature dim

Training set num
Validation set num
Testing set num

Cache memory

Train epoch
Partition?
```

## 训练、验证、测试集大小和批次确定
在`main.cpp`的`server->Initialize(atoi(argv[1]));`环节，会执行`GPUGraphStore.cu`中定义的`void GPUGraphStore::Initialze()`，其中包含两个步骤

```c
env_ = NewIPCEnv(shard_count);
env_ -> Coordinate(info);
```

关于IPC的内容可以追溯到`CUDA_IPC_Service.cu`，在`Coordinate()`中实现了对于`train_step`、`train_batch_size`、`valid_step`、`valid_batch_size`、`test_step` 和 `test_batch_size`的更新

## 函数调用流程

1. main.cpp调用Server->Run()，目标位于Server.cu
2. Run()调用RunnerLoop，目标仍位于Server.cu
3. RunnerLoop调用Runner->RunOnce()，目标仍位于Server.cu
4. RunOnce()中的关键在于一个类型为IPCEnv*的变量env调用的方法IPCPost，定义env的是RunOnce()中的参数params，往回捯到Server->Run，发现定义params的是params_容器，发现它是Server.cu->class GPUServer中的一个私有容器，其中元素的类型为RunnerParams*，此类型声明位于Server.h。想要定位IPCPost，需要找到params_初始化的内容。params_在Server.cu->GPUServer->Initialize()中被初始化为gpu_ipc_env_，这同样是一个Server.cu->class GPUServer的私有变量，类型为IPCEnv*，而gpu_ipc_env_与params_在同样的位置被初始化为gpu_graph_store->GetIPCEnv()，其中有GPUGraphStore* gpu_graph_store = new GPUGraphStore();，于是定位到GPUGraphStore.cuh文件
5. 在GPUGraphStore.cuh中发现了IPCEnv的结构存在于CUDA_IPC_Service.h(Inter-Process Communication,IPC,进程间通信）。截止到目前不确定作用的函数调用包括IPCPost和GetIPCEnv，前者是IPCEnv类的方法，后者是GPUGraphStore类的方法
6. 考虑到关键仍然在于第3点中的RunOnce()，所以优先去看IPCPost()到底做了什么。在IPCPost中，根据设备id和现在的管道拿到一个信号量，然后将此信号量加1表示代表资源被释放，即指定设备id和指定管道资源被释放（这是什么资源？）

7. 根据上述分析，再往下走是和信号量相关的内容，分析这么多完全没看到和图划分相关的内容，于是将视线回归到main.cpp中server->Run()的前一步server->PreSc()（主要是考虑到Initialize无非做的是一些数据初始化的工作，应该不太会涉及到具体的任务代码）
8. 于是转到Server.cu，PreSc是GPUServer类的一个方法，根据ChatGPT的说法，PreSc很可能是preprocess scheduler的缩写，即预处理调度器。在其中不难发现包含调用CandidateSelection和CostModel的内容，根据论文，这部分已经是图划分的下一个cache构建阶段的工作了，所以代码中如果包括图划分就应该在调用CandidateSelection之前就出现了，之前是一个构建线程的工作，线程所需要执行的操作是PreSCLoop
9. 在PreSCLoop中，一个似乎是关键的函数调用是runner->RunPreSc(params);
10. 考虑到runner是一个Runner*类型的指针（实际上，runner是由GPUServer的一个私有数据runner_初始化得到的，而runner_虽然是Runner类型，但是却被初始化了GPURunner类型（这里存在一个类的多态机制），所以可以定位到GPURunner的RunPreSc方法
11. 在RunPreSc中，关键操作是op_factory_[i]->run(op_params_[i]);，其中op_factory_是GPURunner中的一个私有包含Operator*类型变量的容器，因此定位到Operator.h/cu
12. 对于Operator的设计思想是，首先我们仅阐述一下它是怎么做的，至于为何采用这种方式暂时还不太清楚。它创建了一个操作类，对于操作这个对象来说，run就是它的一个方法。然后去创建了不同的操作，都继承这个原始操作类。关于这一点，可以从Server.cu的Initialize()对op_factory_进行初始化的代码中可以看出
13. 开始探究涉及到的几个操作的作用，首先是Batch_Generator。从Operator.cu中看到Batch_Generator重写的run方法中关键操作是batch_generator_kernel，由此定位到Kernel.cu
14. 进行到这一步，很难再继续往下走了，因为都是具体函数的实现。我们从函数追踪这个思路中跳出来，回到Server.cu->PreSc->PreSCLoop->RunPreSc->op_factory_[i]->run(op_params_[i]);,然后回到op_factory_的初始化上，发现它初始化时的操作顺序是BatchGenerator->FeatureExtractor->(RandomSampler->FeatureExtractor->RandomSampler->FeatureExtractor-> … ->RandomSampler->FeatureExtractor)->CachePlanner->CacheUpdater
15. 在RunPreSc中，从第0个操作开始执行，然后每次递增2，所以对应到的操作依次是BatchGenerator->RandomSampler->RandomSampler-> … -> CachePlanner->CacheUpdater。在看论文时，理解的是在每次采样之前都会进行一次shuffle从而生成一个batch，如果真的是按照上面这个执行流，那有可能是在BatchGenerate阶段把后续所需要使用的batch都一同生成了

> 项目源码中的pcm文件夹就是intel开源的pcm项目，代码中所需的性能监控由pcm提供