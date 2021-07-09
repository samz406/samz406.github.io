---
title: k8s本地安装
date: 2021-07-06 20:59:28
tags: [技术]
categories: K8S
---



本地安装K8S

<!-- more -->



### 本地环境

CentOS Linux release 7.9.2009 

安装etcd和k8s



```
yum install etcd kubernetes -y
```

不出意外会出现下面提示成功

![k8s-setup](k8s-setup.png)



按顺序启动下列相关服务