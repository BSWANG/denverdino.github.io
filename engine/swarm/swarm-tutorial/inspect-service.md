---
description: Inspect the application
keywords: tutorial, cluster management, swarm mode
title: Inspect a service on the swarm
---

当你已经在你的swarm中部署了一个服务，你可以使用Docker的命令行接口查看运行在swarm中服务的细节。

1. 如果你还没有准备好，打开一个终端并且ssh到你运行manager节点的machine。例如，本指南中使用的名为`manager1`的machine。

2.  运行`docker service inspect --pretty <SERVICE-ID>`，将以简单可读的方式展示服务的细节。

    查看`helloworld`服务的细节：

    ```
    $ docker service inspect --pretty helloworld

    ID:		9uk4639qpg7npwf3fn2aasksr
    Name:		helloworld
    Mode:		REPLICATED
     Replicas:		1
    Placement:
    UpdateConfig:
     Parallelism:	1
    ContainerSpec:
     Image:		alpine
     Args:	ping docker.com
    ```

    >**提示**: 可以通过运行去掉`--pretty`后同样的命令，得到json格式的服务细节：
    
    ```json
    $ docker service inspect helloworld
    [
    {
        "ID": "9uk4639qpg7npwf3fn2aasksr",
        "Version": {
            "Index": 418
        },
        "CreatedAt": "2016-06-16T21:57:11.622222327Z",
        "UpdatedAt": "2016-06-16T21:57:11.622222327Z",
        "Spec": {
            "Name": "helloworld",
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "alpine",
                    "Args": [
                        "ping",
                        "docker.com"
                    ]
                },
                "Resources": {
                    "Limits": {},
                    "Reservations": {}
                },
                "RestartPolicy": {
                    "Condition": "any",
                    "MaxAttempts": 0
                },
                "Placement": {}
            },
            "Mode": {
                "Replicated": {
                    "Replicas": 1
                }
            },
            "UpdateConfig": {
                "Parallelism": 1
            },
            "EndpointSpec": {
                "Mode": "vip"
            }
        },
        "Endpoint": {
            "Spec": {}
        }
    }
    ]
    ```

4.  运行`docker service ps <SERVICE-ID>`查看哪些节点正在运行服务：

    ```
    $ docker service ps helloworld

    ID                         NAME          SERVICE     IMAGE   LAST STATE         DESIRED STATE  NODE
    8p1vev3fq5zm0mi8g0as41w35  helloworld.1  helloworld  alpine  Running 3 minutes  Running        worker2
    ```

    在这个例子中，`helloworld`的其中一个实例运行在`worker2`节点上。你可能看到这个服务运行在你的manager节点上。默认情况下，swarm中的manager节点也可以像其他的worker节点一样运行任务。

    swarm也展示了服务的`DESIRED STATE` 和 `LAST STATE`，你可以由此来判断服务是否按照要求运行。
    
4.  在运行了任务的节点上输入`docker ps`，可以查看到任务执行容器的具体细节。

    >**提示**: 如果`helloworld`运行在非manager节点上，你需要ssh到那个节点。

    ```bash
    $docker ps

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    e609dde94e47        alpine:latest       "ping docker.com"   3 minutes ago       Up 3 minutes                            helloworld.1.8p1vev3fq5zm0mi8g0as41w35
    ```

## 下一步是什么？

下一步，你可以在swarm上[改变服务的规模](scale-service.md).