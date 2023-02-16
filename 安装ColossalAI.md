# 安装ColossalAI
ColossalAI是开源的通用型深度学习加速工具，可用来开源实现chatGPT（基于GPT3模型）。

下面简述Colossal AI的安装步骤。

## 安装Nvidia CUDA开发包
1. 查询PyTorch支持的CUDA版本（2023-02-16，最高支持CUDA v11.7），参考[PyTorch](https://pytorch.org/get-started/locally/)
2. 下载对应的CUDA版本，参考[Nvidia CUDA](https://developer.nvidia.com/cuda-11-7-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exe_local)
  * [CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive)
  * [NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)
4. 执行所下载的EXE文件，安装CUDA开发包
5. 设置CUDA_HOME环境变量，指向CUDA的安装目录，例如：`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.7`
6. 确认CUDA版本
  ```
  >nvcc -V
  nvcc: NVIDIA (R) Cuda compiler driver
  Copyright (c) 2005-2022 NVIDIA Corporation
  Built on Tue_May__3_19:00:59_Pacific_Daylight_Time_2022
  Cuda compilation tools, release 11.7, V11.7.64
  Build cuda_11.7.r11.7/compiler.31294372_0
  ```

## 安装PyTorch
1. 安装Python 3.x或通过Miniconda安装Python 3.x
2. 参考[PyTorch](https://pytorch.org/get-started/locally/)，安装PyTorch，注意要与CUDA的版本对应。
  * 利用pip：`pip3 install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu117`
  * 利用conda：`conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia`

## 确认PyTorch版本
```
(base) C:\Users\frank>python
Python 3.10.9 | packaged by conda-forge | (main, Jan 11 2023, 15:15:40) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> print(torch.__version__)
1.13.1
>>> print(torch.version.cuda)
11.7
>>>
```

## 修改PyTorch后端分布引擎
由于Windows不支持NCCL后端，需要修改源代码将其后端修改为`GLOO`，否则会报错：`RuntimeError: Distributed package doesn't have NCCL built in`。
1. 打开`C:\Tools\miniconda3\lib\site-packages\torch\distributed\distributed_c10d.py`文件
2. 找到下述代码部分：
```python
        # Use the group name as prefix in the default store, such that
        # a single store can be reused by multiple groups.

        backend = Backend.GLOO
        if backend == Backend.GLOO:
            ...
        elif backend == Backend.NCCL:
            ...
        elif backend == Backend.UCC and is_ucc_available():
            ...
        else:
            ...

    return pg
```
3. 将后端强制设为`GLOO`:
```python
        # Use the group name as prefix in the default store, such that
        # a single store can be reused by multiple groups.
        prefix_store = PrefixStore(group_name, store)
        # Windows不支持NCCL，将backend强制设为GLOO
        backend = Backend.GLOO
        if backend == Backend.GLOO:
            ...
        elif backend == Backend.NCCL:
            ...
        elif backend == Backend.UCC and is_ucc_available():
            ...
        else:
            ...

    return pg
```

## 测试Colossal AI
### 利用命令测试GPU
这个基准测试跑一个并行的MLP模型， 输入数据的维度为`（批大小，序列长度，隐藏层维度）`。通过指定GPU的数量，Colossal-AI会搜索所有可行的并行配置。用户可以通过查看`colossalai benchmark --help`来自定义相关的测试参数。

`colossalai benchmark --gpus 1`
