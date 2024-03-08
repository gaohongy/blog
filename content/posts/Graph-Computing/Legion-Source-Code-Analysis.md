---
title: Legion Source Code Analysis
subtitle:
description:
keywords:
summary:
license:
date: 2024-03-05T20:17:06+08:00
lastmod: 2024-03-08T16:35:12+08:00
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
