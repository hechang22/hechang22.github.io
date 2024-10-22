---
title: Github Copilot+ChatGPT-Next-Web搭建记录
date: 2024-03-18
tags:
  - GPT
categories: 计算机
description:
---
### 服务器：阿里云

轻量应用服务器2+2，30M带宽（香港）两年，  
“云工开物”提供300元优惠+三折（部分高校）。折算下来近似于一千元以内免费。  
让我们一起感谢阿里云。
![end](https://picbed-8tn.pages.dev/img/24_3_18_aliyun.png)
### Github Copilot 学生申请

[Github Education](https://education.github.com/discount_requests/application)
三次申请后通过。  
![end](https://picbed-8tn.pages.dev/img/24_3_18_copilot.png)
- ***学校邮箱***；
- 手持校园卡拍照；
- *approved*状态可能持续几天，直到收到邮件（不一定）或*approved*由绿变**紫**；

### copilot-gpt4-service配置

#### 安装docker-ce
服务器为Ubuntu22.04，直接apt安装并不可行。  

首先，更新现有包列表：

```shell
sudo apt update 
```

安装几个依赖包，让apt可以支持HTTPS：

```shell
sudo apt install apt-transport-https ca-certificates curl software-properties-common 
```

将官方Docker库的GPG公钥添加到系统中：

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
```

将Docker库添加到APT里：

```shell
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" 
```

再次更新现有包列表

```shell
sudo apt update 
```

为了确保修改生效，让新的安装从Docker库里获取，而不是从Ubuntu自己的库里获取，执行：

```shell
apt-cache policy docker-ce
```

可能会看到如下图的输出，各个系统的情况可能略有不同。

```shell
docker-ce:
  Installed: (none)
  Candidate: 5:19.03.9~3-0~ubuntu-focal
  Version table:
     5:19.03.9~3-0~ubuntu-focal 500
        500 Index of linux/ubuntu/ focal/stable amd64 Packages
```

开始安装吧：

```shell
sudo apt install docker-ce 
```

现在Docker应该已经安装好了，守护进程也开启了，开机启动也开启了。我们看看Docker的运行状态吧。

```shell
sudo systemctl status docker 
```

你看到的输出应该是这样的:

```shell
output:
 docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor prese>
     Active: active (running) since Mon 2024-03-18 10:21:31 CST; 20min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 59919 (dockerd)
      Tasks: 9
     Memory: 30.0M
        CPU: 614ms
     CGroup: /system.slice/docker.service
             └─59919 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/co>
```

#### 安装并运行 copilot-gpt4-service 服务

```shell
docker run -d \
  --name copilot-gpt4-service \
  --restart always \
  -p 8080:8080 \
  -e HOST=0.0.0.0 \
  aaamoon/copilot-gpt4-service:latest
```
启动在端口8080
***Attention!别忘了在阿里云开放8080端口！！！！！***
#### 获取token

```shell
bash -c "$(curl -fsSL https://raw.githubusercontent.com/aaamoon/copilot-gpt4-service/master/shells/get_copilot_token.sh)"
```

获得了ghu_开头的token

### 配置客户端 ChatGPT-Next-Web

[ChatGPT-Next-Web](https%3A//github.com/ChatGPTNextWeb/ChatGPT-Next-Web)

  

