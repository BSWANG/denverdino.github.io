---
description: Initialize the swarm
keywords: tutorial, cluster management, swarm mode
title: Create a swarm
---

当你完成了[安装指南](index.md)中的步骤后，你就可以创建swarm了。请确保Docker Engine daemon已经在宿主机器上启动。

1. 打开终端，ssh到你想运行manager节点的机器。例如，在指南中使用的名为`manager1`的机器。

    ```bash
    docker-machine ssh manager1
    ```

2. 运行下面的命令，创建一个新的swarm：
    ```bash
    docker swarm init --advertise-addr <MANAGER-IP>
    ```

     >**注意:** 如果你在使用Mac下的Docker或者是Windows下的Docker测试单节点的swarm，只需要简单运行 `docker swarm init`，不用带参数。在这种情况下，没有必要指定`advertise-addr`。想了解更多，可以查看swarm的相关话题[如何使用Mac下的Docker或者Windows下的Docker](index.md#use-docker-for-mac-or-docker-for-windows)。

	在指南中，以下的命令能在`manager1`的机器上创建一个swarm：

    ```bash
    $ docker swarm init --advertise-addr 192.168.99.100
    Swarm 初始化：当前节点(dxn1zf6l61qsb1josjja83ngz)现在是manager。

    运行以下的命令，可以将一个worker节点加入swarm：

        docker swarm join \
        --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
        192.168.99.100:2377

    运行 'docker swarm join-token manager'并且跟从以下指令，可以将一个manager节点加入swarm。
    ```

    '--advertise-addr'这个标签配置manager节点广播它的地址是'192.168.99.100'。其他在swarm里的节点必须能连通manager的这个IP地址。

    输出包括让新节点加入swarm的命令。'--token'标签决定节点加入后作为manager节点还是worker节点。
    ```

3.  运行 `docker info` 查看当前swarm的状态：

    ```bash
    $ docker info

    Containers: 2
    Running: 0
    Paused: 0
    Stopped: 2
      ...snip...
    Swarm: active
      NodeID: dxn1zf6l61qsb1josjja83ngz
      Is Manager: true
      Managers: 1
      Nodes: 1
      ...snip...
    ```

4.  运行 `docker node ls` 查看node节点的信息：

    ```bash
    $ docker node ls

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader

    ```

    节点id旁边的`*`号表明你当前正连接到这个节点。

    Docker引擎的swarm mode会自动用主机名命名节点的名称。指南将在后续介绍其他的列。
    

## 下一步是什么?


在指南的下一个小节，我们将[增加两个节点](add-nodes.md) 到这个集群。
