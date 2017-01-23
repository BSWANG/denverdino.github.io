---
description: Apply rolling updates to a service on the swarm
keywords: tutorial, cluster management, swarm, service, rolling-update
title: Apply rolling updates to a service
---

在指南的之前部分，你已经[扩展了一个服务的实例数目](scale-service.md)。在本指南，你会基于Redis 3.0.6的容器镜像部署一个服务。之后，使用滚动更新的方式，将服务升级成Redis 3.0.7的容器镜像

1. 如果你还没有准备好，打开一个终端，ssh到运行manager节点的machine上。例如，本指南中使用的`manager1`的machine。

2.  将Redis 3.0.6部署到swarm中，并且配置swarm使用10秒延迟的更新方式：

    ```bash
    $ docker service create \
      --replicas 3 \
      --name redis \
      --update-delay 10s \
      redis:3.0.6

    0u6a4s31ybk7yw2wyvtikmu50
    ```
    你可以配置滚动更新策略中的部署时间。

    `--update-delay`标签可以配置一个或者多个任务服务更新时的时间延迟。可以使用`T`来组合描述时间，秒`Ts`，分钟 `Tm`，或者小时`Th`。所以，`10m30s`意味着10分钟30秒的延迟。

    默认情况下，这种调度机制只会一次更新一个任务。你可以通过使用`--update-parallelism`这个标签配置服务同时更新的最大个数。
    
    默认情况下，当某个任务更新返回`RUNNING`后，调度器将调度其他任务来更新直到所有的任务都更新了。如果，在更新的过程中，某一个任务返回`FAILED`更新失败，则调度器将暂停更新。如果你想控制这种行为，可以在`docker service create`或者`docker service update`的时候，使用`--update-failure-action`标签。

3.  检查`redis`服务：

    ```bash
    $ docker service inspect --pretty redis

    ID:             0u6a4s31ybk7yw2wyvtikmu50
    Name:           redis
    Mode:           Replicated
     Replicas:      3
    Placement:
     Strategy:	    Spread
    UpdateConfig:
     Parallelism:   1
     Delay:         10s
    ContainerSpec:
     Image:         redis:3.0.6
    Resources:
    ```

4. 现在你可以更新`redis`容器镜像了. swarm manager将按照`UpdateConfig`中的策略，更新节点：

    ```bash
    $ docker service update --image redis:3.0.7 redis
    redis
    ```

    默认情况下，调度器将以以下的规则来滚动更新：

    * 停止第一个任务。
    * 为停止的任务，调度更新。
    * 为停止的任务，开启容器。
    * 调度器将等待指定的延迟时间，然后开始停止下一个任务。
    * 如果在更新的过程中，一个任务返回`FAILED`，则会停止更新。

5.  运行`docker service inspect --pretty redis`查看新镜像容器的状态。

    ```bash
    $ docker service inspect --pretty redis

    ID:             0u6a4s31ybk7yw2wyvtikmu50
    Name:           redis
    Mode:           Replicated
     Replicas:      3
    Placement:
     Strategy:	    Spread
    UpdateConfig:
     Parallelism:   1
     Delay:         10s
    ContainerSpec:
     Image:         redis:3.0.7
    Resources:
    ```

    `service inspect`命令输出将显示是否你的更新由于失败而停止了：

    ```bash
    $ docker service inspect --pretty redis

    ID:             0u6a4s31ybk7yw2wyvtikmu50
    Name:           redis
    ...snip...
    Update status:
     State:      paused
     Started:    11 seconds ago
     Message:    update paused due to failure or early termination of task 9p7ith557h8ndf0ui9s0q951b
    ...snip...
    ```

    使用`docker service update <SERVICE-ID>`重启一个被停止的更新任务。例如：
    
    ```bash
    docker service update redis
    ```

    未来避免某个更新的重复失败，你可能需要通过`docker service update`来重新配置服务。

6.  运行`docker service ps <SERVICE-ID>`查看滚动更新的情况：

    ```bash
    $ docker service ps redis

    ID                         NAME         IMAGE        NODE       DESIRED STATE  CURRENT STATE            ERROR
    dos1zffgeofhagnve8w864fco  redis.1      redis:3.0.7  worker1    Running        Running 37 seconds
    88rdo6pa52ki8oqx6dogf04fh   \_ redis.1  redis:3.0.6  worker2    Shutdown       Shutdown 56 seconds ago
    9l3i4j85517skba5o7tn5m8g0  redis.2      redis:3.0.7  worker2    Running        Running About a minute
    66k185wilg8ele7ntu8f6nj6i   \_ redis.2  redis:3.0.6  worker1    Shutdown       Shutdown 2 minutes ago
    egiuiqpzrdbxks3wxgn8qib1g  redis.3      redis:3.0.7  worker1    Running        Running 48 seconds
    ctzktfddb2tepkr45qcmqln04   \_ redis.3  redis:3.0.6  mmanager1  Shutdown       Shutdown 2 minutes ago
    ```
    在swarm更新所有节点之前，你可以发现一些任务在使用`redis:3.0.6`的镜像版本，而其他的在使用`redis:3.0.7`的镜像版本。以上的输出展示了当滚动更新开始后的状态。

下面，可以查看在swarm中如何[排水](drain-node.md)。