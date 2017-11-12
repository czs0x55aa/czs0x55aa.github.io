---
title: 使用docker配置深度学习环境
date: 2017-11-12 20:45:53
tags: ["深度学习", "学习笔记"]
---
之前一直用着原生系统搭建起来的环境，用着还算稳定，不过偶尔也会遇到一些困扰。比如tf,pytorch这类框架的API不是很稳定，版本更新后老的代码用不了。有时候跑别人的程序需要切换对应的版本。  
在师弟的安利下尝试了一下用docker来运行深度学习环境。  
之前用过几次，但是对docker的了解较少，稍微做了一点功课，[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)。  
这里略过le docker的安装，详见docker的官方文档。
## 安装nvidia-docker
对于深度学习开发环境来说，CUDA环境是至关重要的，要在docker中训练模型，首先得要能够在docker中运行CUDA，nvidia官方给出了解决方案————[nvidia-docker](https://github.com/NVIDIA/nvidia-docker)。  
<!-- more -->
![nvidia-docker官方的配图](https://cloud.githubusercontent.com/assets/3028125/12213714/5b208976-b632-11e5-8406-38d379ec46aa.png)
在docker中运行CUDA驱动，原生的系统中则省去了CUDA的安装。  
上层的容器（具体的框架运行环境）则依赖于CUDA驱动，通过nvidia-docker来运行容器，可以获得CUDA加持。  
更多细节建议参考[nvidia-docker wiki](https://github.com/NVIDIA/nvidia-docker/wiki)
Ubuntu环境安装nvidia-docker:
```sh
# Install nvidia-docker and nvidia-docker-plugin
wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb
sudo dpkg -i /tmp/nvidia-docker*.deb && rm /tmp/nvidia-docker*.deb
```
测试nvidia-docker:
```sh
# Test nvidia-smi
nvidia-docker run --rm nvidia/cuda nvidia-smi
```
第一次运行该命令需要较长的时间拉取nvidia/cuda镜像。  
如果一切顺利的话，最终可以看到熟悉的`nvidia-smi`的输出。  
## 安装DL框架的运行环境
基本上主流的框架都会提供docker安装方式，这里以[Pytorch](https://github.com/pytorch/pytorch)为例。  
Pytorch官方提供了Dockerfile，使用编译安装的，需要clone官方的repo，然后执行下面的命令即可。  
```sh
docker build -t pytorch .
```
由于是编译安装的方式，构建镜像大概需要一两个小时的时间。  
测试运行一下：
```sh
nvidia-docker run --rm -ti --ipc=host pytorch
```
可以尝试启动python然后import torch，此时CUDA状态是可用的。

## 运行自己的模型代码
通过上面的方式运行容器，虽然此时CUDA和框架框架都具备了，但是和本机是隔离的，容器中并没有自己的模型代码。  
一种解决的方式是通过磁盘的映射。  
```sh
nvidia-docker run --rm -ti --ipc=host \
-e CUDA_VISIBLE_DEVICES=0 \
--volume=$PWD:/workspace \
pytorch python main.py
```
其中`$PWD`是当前运行命令的目录，也可以手动替换成其他的，workspace则是pytorch容器中的默认目录，这个是官方Dockefile中默认定义的。  
用这种方式可以将本地的代码目录映射到容器中，但是需要留心代码中的绝对路径，因为容器中的路径和本地路径是不同的（目前还没有找到比较好的解决方案）。  

## 扩展镜像环境
上面运行的容器是由pytorch的官方镜像创建的，但是我们的代码可能还用到了别的，而官方镜像中不包含的内容。比如用了某个第三方的python库，这个时候就需要对镜像进行扩展。  
方法是继承官方的pytorch镜像，然后自定义自己的Dockerfile，并创建出新的容器，运行`nvidia-docker`时指定使用我们自己自定义的镜像即可。

## 结束.
至此，基本可以像平时那样跑模型了，而且安装不同的框架和版本都非常方便。  
nvidia-docker的命令有点长，可以考虑写成shell脚本的形式。  
目前还没有解决的是代码中的路径问题，自己通过fabfile来自动化的替换掉配置文件中的路径，但是有些情况仍然不太方便。
