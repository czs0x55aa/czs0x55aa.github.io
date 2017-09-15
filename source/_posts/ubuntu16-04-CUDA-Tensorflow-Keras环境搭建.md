---
title: ubuntu16.04+CUDA+Tensorflow+Keras环境搭建
date: 2016-12-20 15:52:20
tags: "零碎记录"
categories:
---
ubuntu安装环节略过，另外系统安装后记得更新下Nvidia的驱动
## 实验环境
系统：ubuntu 16.04
显卡：GTX 1060 6G
python环境：python2.7 (已安装Anaconda4.2)
<!-- more -->
## 1. 安装CUDA与cuDNN
由于16.04自带的gcc和g++版本是5.4的，需要将其替换为5.0之前的版本
### 1.1 更换gcc和g++的版本
安装地板版的gcc与g++
```bash
sudo apt-get install gcc-4.9 g++-4.9
```
删除/usr/bin目录下原始的gcc,g++的链接文件
```bash
cd /usr/bin
sudo rm gcc
sudo rm g++
```
创建软链接
```bash
sudo ln -s gcc-4.9 gcc
sudo lin -s g++-4.9 g++
```
### 1.2 安装CUDA8.0
CUDA下载地址：
https://developer.nvidia.com/cuda-downloads
使用deb包安装
```bash
sudo dpkg -i cuda-repo-ubuntu1604-8-0-local_8.0.44-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda
```
写环境变：
```bash
vim ~/.bashrc
export PATH=/usr/local/cuda/bin
export LD_LIBRARY_PATH=/usr/local/cuda/lib64
```
### 1.3 安装cuDNN
cuDNN下载：https://developer.nvidia.com/rdp/cudnn-download
解压缩后将文件拷贝到CUDA目录下即可
```bash
sudo cp lib64/* /usr/local/cuda/lib64/
sudo cp include/cudnn.h /usr/local/cuda/include/
```
## 2. 安装TensorFlow
**PS:目前tensorflow已经出了1.0版本，建议参考官方最新的[安装指导](https://www.tensorflow.org/install/install_linux)，旧版本升级可以使用`pip install --upgrade tenforlow-gpu`**
官网提供了多种安装方式，当前的版本是0.12
由于自己以及安装了Anaconda，所以采用该方式安装：
```bash
conda create -n tensorflow python=2.7
source activate tensorflow
# Ubuntu/Linux 64-bit, GPU enabled, Python 2.7
# Requires CUDA toolkit 8.0 and CuDNN v5. For other versions, see "Installing from sources" below.
(tensorflow)$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-0.12.0-cp27-none-linux_x86_64.whl
pip install --ignore-installed --upgrade $TF_BINARY_URL
```
我这里安装的是GPU的版本，如果没有GPU则安装对应的CPU版本即可。
### 3. 安装Keras
在tensorflow的环境下直接安装即可：
```bash
source activate tensorflow
pip install keras
```
