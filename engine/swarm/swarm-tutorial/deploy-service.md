---
description: Deploy a service to the swarm
keywords: tutorial, cluster management, swarm mode
title: Deploy a service to the swarm
---

当你已经[创建swarm](create-swarm.md)之后，你可以部署一个服务到swarm中。指南中，你也可以[增加worker节点](add-nodes.md)的相关介绍，但是那并不是部署一个服务所必须的条件。

1. 打开终端，ssh到你运行manager节点的machine上。例如，本指南中使用的`manager1`machine。

2. 运行以下命令：

    ```bash
    $ docker service create --replicas 1 --name helloworld alpine ping docker.com

    9uk4639qpg7npwf3fn2aasksr
    ```

    * `docker service create` 命令将创建服务。
    * `--name` 标签指定了服务的名称 `helloworld`。
    * `--replicas` 标签指定了需要1个运行状态的实例。
    * `alpine ping docker.com` 参数定义了服务是一个Alpine Linux 容器，并且在容器中运行命令`ping docker.com`。

3.  运行`docker service ls` 查看运行服务的列表：

    ```
    $ docker service ls

    ID            NAME        SCALE  IMAGE   COMMAND
    9uk4639qpg7n  helloworld  1/1    alpine  ping docker.com
    ```

## 下一步是什么?

现在你已经部署了一个服务到swarm，你可以[查看服务信息](inspect-service.md)。