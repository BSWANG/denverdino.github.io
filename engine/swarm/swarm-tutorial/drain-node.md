---
description: Drain nodes on the swarm
keywords: tutorial, cluster management, swarm, service, drain
title: Drain a node on the swarm
---

在指南的前几个章节，所有的节点都以`活跃`的状态在运行。swarm manager可以将任务分配给任何一个`活跃`的节点，所以到目前为止所有的节点都是可以接受任务的调度的。


有时候，例如一些计划中的保养时间，你需要将节点设置成`排水`的模式。`排水`模式阻止节点从swarm manager中接收新任务。这也意味着，swarm manager将停止在这个节点上运行任务，或者部署任务副本任务在一个`排水`模式中的节点。


1. 如果你还没有准备好，打开一个终端并且ssh到你运行manager 节点的机器上。例如，在指南中使用的`manager1`机器。
    
    
2.  检查你所有的节点都处于活跃状态。

    ```bash
    $ docker node ls

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    1bcef6utixb0l0ca7gxuivsj0    worker2   Ready   Active
    38ciaotwjuritcdtn9npbnkuz    worker1   Ready   Active
    e216jshn25ckzbvmwlnh5jr3g *  manager1  Ready   Active        Leader
    ```
    

3.  如果你还停留在[滚动更新](rolling-update.md)指南中运行`redis`服务，那么现在可以运行

    ```bash
    $ docker service create --replicas 3 --name redis --update-delay 10s redis:3.0.6

    c5uo6kdmzpon37mgj9mwglcfw
    ```
   
    
4.  运行`docker service ps redis`，检查swarm manager怎样分配任务到不同的节点上：

    ```bash
    $ docker service ps redis

    ID                         NAME     SERVICE  IMAGE        LAST STATE          DESIRED STATE  NODE
    7q92v0nr1hcgts2amcjyqg3pq  redis.1  redis    redis:3.0.6  Running 26 seconds  Running        manager1
    7h2l8h3q3wqy5f66hlv9ddmi6  redis.2  redis    redis:3.0.6  Running 26 seconds  Running        worker1
    9bg7cezvedmkgg6c8yzvbhwsd  redis.3  redis    redis:3.0.6  Running 26 seconds  Running        worker2
    ```

    此时，swarm manager会分配任务到每一个节点。你可能在你的环境下看到节点中不同的任务分配情况。

        
5.  为一个已经分配的任务的节点进行排水，只需运行 `docker node update --availability drain <NODE-ID>`:

    ```bash
    docker node update --availability drain worker1

    worker1
    ```


6.  检查该节点的可用性:

    ```bash
    $ docker node inspect --pretty worker1

    ID:			38ciaotwjuritcdtn9npbnkuz
    Hostname:		worker1
    Status:
     State:			Ready
     Availability:		Drain
    ...snip...
    ```

    这个节点显示`AVAILABILITY`为`Drain`，它的可用性为排水模式。


7.  运行 `docker service ps redis`， 检查swarm manager如何更新`redis`服务的任务分配：

    ```bash
    $ docker service ps redis

    ID                         NAME          IMAGE        NODE      DESIRED STATE  CURRENT STATE           ERROR
    7q92v0nr1hcgts2amcjyqg3pq  redis.1       redis:3.0.6  manager1  Running        Running 4 minutes
    b4hovzed7id8irg1to42egue8  redis.2       redis:3.0.6  worker2   Running        Running About a minute
    7h2l8h3q3wqy5f66hlv9ddmi6   \_ redis.2   redis:3.0.6  worker1   Shutdown       Shutdown 2 minutes ago
    9bg7cezvedmkgg6c8yzvbhwsd  redis.3       redis:3.0.6  worker2   Running        Running 4 minutes
    ```

    为了达到预期的状态，swarm manager结束了排水模式下的节点任务，并且创建新任务给活跃的节点。


8.  运行 `docker node update --availability active <NODE-ID>`，将一个排水模式下的节点恢复成活跃的状态:

    ```bash
    $ docker node update --availability active worker1

    worker1
    ```


9.  检查节点是否更新状态：

    ```bash
    $ docker node inspect --pretty worker1

    ID:			38ciaotwjuritcdtn9npbnkuz
    Hostname:		worker1
    Status:
    State:			Ready
    Availability:		Active
    ...snip...
    ```

  当你设置节点回到活跃的状态，它将可以接收新的任务：

  * 当一个服务准备扩大规模时
  * 当滚动升级时
  * 当你设置另一个节点为排水模式时
  * 当在另一个活跃节点上的任务失败时