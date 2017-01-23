---
description: Getting Started tutorial for Docker Engine swarm mode
keywords: tutorial, cluster management, swarm mode
title: Getting started with swarm mode
---

本指南为你介绍了Docker引擎Swarm mode的特性。在开始之前，你可以阅读[关键概念](../key-concepts.md)熟悉相关的内容。

本指南将指导你完成以下的活动：

* 初始化一个swarm mode模式下的Docker集群
* 添加节点进入swarm
* 部署应用服务到swarm
* 当一切就绪，管理swarm

本指南通过在终端上输入Docker Engine CLI命令来进行演示。你应该在联网的机器上安装Docker，并且熟悉你在终端上运行的命令。

如果你对Docker一无所知，可以参考文章[关于Docker引擎](../../index.md)。

## 准备工作

为了运行指南，你需要准备：

* [三个联网的主机](index.md#three-networked-host-machines)
* [安装Docker 引擎1.12或者更高的版本](index.md#docker-engine-1-12-or-newer)
* [manager机器的IP地址](index.md#the-ip-address-of-the-manager-machine)
* [主机之间的开放端口](index.md#open-ports-between-the-hosts)

### 三个联网的主机

本指南使用了三个联网主机作为swarm中的节点。这些主机可以是你PC上的虚拟机，或者数据中心，或者云服务提供商。本指南使用了以下的机器名：

* manager1
* worker1
* worker2

>**注意:** 你也可以根据一些指南测试一个单节点的swarm，在这种情况下，你只需要一个主机。多节点的命令将不会生效，但是你依然可以初始化swarm，创建服务，并且扩容服务。

###  Docker引擎1.12或者更高的版本

本指南需要主机上安装1.12或者更高版本的Docker引擎。安装Docker引擎并且确认在每台机器上Docker引擎的守护进程都处于运行状态。你可以从以下的内容中获取最新的Docker引擎版本：

* [在Linux主机上安装Docker引擎](index.md#install-docker-engine-on-linux-machines)

* [使用Mac下的Docker或者Windows下的Docker](index.md#use-docker-for-mac-or-docker-for-windows)

#### 在Linux主机上安装Docker引擎

如果你在物理主机或者云主机上使用Linux，你只需要查看[Linxu安装指南](../../installation/index.md)的文档。部署好三个machine，你就已经做好准备了。你可以在Linux机器上测试单节点或者多节点swarm的场景。

#### 使用Mac下的Docker或者Windows下的Docker

或者，在单台机器上安装最新的[Docker for Mac](/docker-for-mac/index.md)应用或者[Docker for Windows](/docker-for-windows/index.md)应用。你可以在这台电脑上测试单节点或者多节点的场景，但是你将需要使用Docker Machine去测试多节点的场景。


* 现在，你不能单独使用Mac下的Docker或者Windows下的Docker测试多节点swarm。但是，你可以使用[Docker Machine](/machine/overview.md) 包含的版本去创建swarm节点（参考[开始使用Docker Machine和本地虚拟机](/machine/get-started.md)），继续阅读指南，你将发现所有多节点的特性。在这种情况下，你可以在Mac或者Windows下的Docker上运行命令，但是Docker主机本身并不会加入swarm。（例如，它将不会是我们例子中的`manager1`， `worker1`，或者`worker2`）。当你创建好了节点，你可以像在Mac或者Windows终端中运行Docker一样，运行所有swarm的命令。

### manager机器的IP地址

IP地址需要是一个主机操作系统可达的网络接口。在swarm中的所有节点都需要能访问到manager的那个IP地址。

因为所有的其他节点都通过那个IP地址访问manager节点，你应该使用一个固定的IP地址。

你可以在Linux或者macOS上运行 `ifconfig`，查看到一组可使用的网络接口列表。

如果你在使用Docker Machine，你可以通过`docker-machine ls`或者 `docker-machine ip <MACHINE-NAME>` &#8212;例如，`docker-machine ip manager1`来获取managerde IP地址。

本指南使用了`manager1`，IP地址是`192.168.99.100`。

### 不同主机间的开放端口

以下的端口必须可用。在一些系统上，这些端口会被默认打开：

* 为集群管理通信的**TCP 端口 2377** 
* 为节点间通信的**TCP** 和 **UDP 端口 7946**
* overlay网络通信的**TCP** 和 **UDP port 4789** 

如果你计划使用加密的overlay网络（`--opt encrypted`），你将需要确保端口50（ESP）被打开了。


## 下一步是什么？

如果在你已经准备好了环境，你可以开始[创建一个swarm](create-swarm.md)