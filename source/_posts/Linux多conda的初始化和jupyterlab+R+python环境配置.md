---
title: Conda Initiation on Cluster and VSCode+Jupyterlab+R Setup
date: 2024-09-14 15:03:45
tags:
  - Linux
  - 生信
description: 被迫共用同一个集群账号导致的
---

**前一天用的好好的conda环境突然陷入了完全的混乱：R4.4.1没了？？**  
**估计是这个账号使用的本科生太多导致的，开始重建一个**  

0. 试图在此前安装的旧版本conda下重新新建环境和安装R，发现`conda install`和`conda update`都极其缓慢。尝试换源完全没用，估计原因为conda版本太旧；所幸重新装一个conda。
1. 运行安装脚本。注意指定安装路径：`~/hechang/miniconda`。
2. 初始化Conda，修改`~/.bashrc`:
```shell
  __conda_setup="$('/home/huangkeyun/hechang/miniconda/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/huangkeyun/hechang/miniconda/etc/profile.d/conda.sh" ]; then
        . "/home/huangkeyun/hechang/miniconda/etc/profile.d/conda.sh"
    else
        export PATH="/home/huangkeyun/hechang/miniconda/bin:$PATH"
    fi
fi
unset __conda_setup
```
3. 换源，新建环境，安装必需包：
```shell
### 清华源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

### 环境创建与配置
conda create -n r.4.4.1 r-base=4.4.1 jupyterlab

### 启动一下试试
conda activate r.4.4.1
jupyter #报错
```
4. 无法启动jupyterlab。
```shell
whereis python #发现有很多python，默认的版本无法启动

export PATH=/home/huangkeyun/hechang/miniconda/bin:$PATH  #同时加入~/.bashrc中
```

5. 配置jupyterlab+R环境：
  @R on cluster
  ```R
  install.packages("IRkernel")
  IRkernel::installspec()
  ```
  @jupyter on VScode 选中指定环境下的R内核。

现存问题：不知道是vscode插件原因还是jupyter对R的支持问题，jupyter无法显示环境变量之类的信息。
