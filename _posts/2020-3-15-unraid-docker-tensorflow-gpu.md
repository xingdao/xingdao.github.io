---
layout: post
title: 2070 super 8g gaming oc unraid docker tensorflow:latest-py3-gpu-jupyter 
category: Python
keywords: unraid,tensorflow-gpu,2070s
---

### 硬件

    B65 + E3 1230V2 + 2070s

    请确认开启 Vt 和 Vd

### 系统

来源 [https://post.smzdm.com/p/av7mlz4m/](https://post.smzdm.com/p/av7mlz4m/)

    unraid 安装

    2070 多了个 type-c 无法直接分配gpu 后启动
    解决方案 Unraid->SETTINGS->PCIe ACS overide改成Both 

    vm 选择 seabios 首先不要分配 gpu 或者分配gpu后添加 vnc 用于安装系统 并开启ssh

    系统 ubuntu 18.04 server


### 具体步骤

    换源 清华大学 apt & docker
    安装 docker
    
    安装驱动编译环境 gcc make

    安装驱动 NVIDIA-Linux-x86_64-440.64.run
        gcc(7.5.0 无视告警)

    设置 apt 代理 安装 nvidia-docker
        distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
        curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
        curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

        sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
        sudo systemctl restart docker

    设置 docker代理
        https://docs.docker.com/config/daemon/systemd/

    #### Test nvidia-smi with the latest official CUDA image
    docker run --gpus all nvidia/cuda:10.0-base nvidia-smi

    运行 
    docker run -it -p 8888:8888 tensorflow/tensorflow:latest-py3-gpu-jupyter
