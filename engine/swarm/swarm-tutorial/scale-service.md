---
description: Scale the service running in the swarm
keywords: tutorial, cluster management, swarm mode, scale
title: Scale the service in the swarm
---

当你已经在swarm上[部署了一个服务](deploy-service.md)，你就可以使用Docker Cli扩展在swarm中列举出的服务个数。


1. 如果你还没有准备好，打开一个终端，然后ssh到你运行swarm manager的节点机器。例如，在指南中使用的 `manager1`的机器。

2.  运行以下的命令，改变在swarm中运行的服务的期望状态：

    ```bash
    $ docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>
    ```

    例如：

    ```bash
    $ docker service scale helloworld=5

    helloworld scaled to 5
    ```

3.  运行`docker service ps <SERVICE-ID>`检查更新后的任务列表:

    ```
    $ docker service ps helloworld

    ID                         NAME          SERVICE     IMAGE   LAST STATE          DESIRED STATE  NODE
    8p1vev3fq5zm0mi8g0as41w35  helloworld.1  helloworld  alpine  Running 7 minutes   Running        worker2
    c7a7tcdq5s0uk3qr88mf8xco6  helloworld.2  helloworld  alpine  Running 24 seconds  Running        worker1
    6crl09vdcalvtfehfh69ogfb1  helloworld.3  helloworld  alpine  Running 24 seconds  Running        worker1
    auky6trawmdlcne8ad8phb0f1  helloworld.4  helloworld  alpine  Running 24 seconds  Accepted       manager1
    ba19kca06l18zujfwxyc5lkyn  helloworld.5  helloworld  alpine  Running 24 seconds  Running        worker2
    ```

    你可以看到swarm扩建了4个新的任务，最终达到了5个Alpine Linux的运行实例。这些任务分布在swarm的3个节点上。还有1个运行在了`manager1`的节点上。

4.  运行`docker ps`查看你连接到的节点上运行了哪些容器，以下的例子展示了`manager1`节点上运行的任务：

    ```
    $ docker ps

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    528d68040f95        alpine:latest       "ping docker.com"   About a minute ago   Up About a minute                       helloworld.4.auky6trawmdlcne8ad8phb0f1
    ```

    如果你想查看别的节点上运行的容器，你可以ssh到这些节点，然后运行`docker ps`的命令。

## 下一步是什么？

看到这里，你已经完成了`helloworld`的服务教程。下一步将会为你介绍[如何删除服务](delete-service.md)。