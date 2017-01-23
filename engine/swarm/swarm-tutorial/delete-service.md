---
description: Remove the service from the swarm
keywords: tutorial, cluster management, swarm, service
title: Delete the service running on the swarm
---

指南中的后续步骤将不会再使用`helloworld`的服务，所以你可以从swarm中删除该服务。

1. 如果你还没有准备好，打开你的终端，ssh到你运行manager节点的machine。例如，指南中叫`manager1`的machine。

2.  运行`docker service rm helloworld`去删除`helloworld`服务。

    ```
    $ docker service rm helloworld

    helloworld
    ```

3.  运行 `docker service inspect <SERVICE-ID>` 去校验swarm manager是否已经删除服务。终端将返回服务未被发现的信息：

    ```
    $ docker service inspect helloworld
    []
    Error: no such service: helloworld
    ```

4.  即使服务已经不再存在，但是对应的任务容器可能仍需要一些时间去销毁。你可以使用`docker ps`去校验是否容器已经清除。
    
    ```bash
    $ docker ps
    
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
        db1651f50347        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.5.9lkmos2beppihw95vdwxy1j3w
        43bf6e532a92        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.3.a71i8rp6fua79ad43ycocl4t2
        5a0fb65d8fa7        alpine:latest       "ping docker.com"        44 minutes ago      Up 45 seconds                           helloworld.2.2jpgensh7d935qdc857pxulfr
        afb0ba67076f        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.4.1c47o7tluz7drve4vkm2m5olx
        688172d3bfaa        alpine:latest       "ping docker.com"        45 minutes ago      Up About a minute                       helloworld.1.74nbhb3fhud8jfrhigd7s29we
    
    $ docker ps
       CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               

    ```
    
## 下一步是什么?

在指南的下一小节，你将创建一个新的服务，并且使用[滚动更新](rolling-update.md).