---
description: 如何连通Docker容器的网络。
keywords: 示例，用法，卷，docker，文档，用户指南，数据，卷
redirect_from:
- /engine/userguide/containers/networkigncontainers/
- /engine/userguide/networkigncontainers/
title: 容器网络连通
---


如果您正在看用户指南，您刚刚构建并运行了一个简单的应用程序。您还构建了您自己的镜像。本节教您如何为您的容器连通他们的网络。

## 在默认网络上启动容器

Docker通过使用**网络驱动程序**连通容器的网络。默认情况下，Docker为您提供了两个网络驱动程序，`bridge`和`overlay`驱动程序。您还可以编写网络驱动程序插件，以便您可以创建自己的驱动程序，但这是一个高级任务。

每次安装Docker Engine都会自动包含三个默认网络。您可以列出:

	$ docker network ls

    NETWORK ID          NAME                DRIVER
    18a2866682b8        none                null
    c288470c46f6        host                host
    7b369448dccb        bridge              bridge

名为`bridge`的网络是一个特殊的网络。除非您另有说明，Docker总是在这个网络中启动您的容器。立即尝试:

	$ docker run -itd --name=networktest ubuntu

	74695c9cea6d9810718fddadc01a727a5dd3ce6a69d09752239736c030599741

检查网络是找出容器的IP地址最简单的方法。

```bash
$ docker network inspect bridge

[
    {
        "Name": "bridge",
        "Id": "f7ab26d71dbd6f557852c7156ae0574bbf62c42f539b50c8ebde0f728a253b6f",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.17.0.1/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Containers": {
            "3386a527aa08b37ea9232cbcace2d2458d49f44bb05a6b775fba7ddd40d8f92c": {
                "EndpointID": "647c12443e91faf0fd508b6edfe59c30b642abb60dfab890b4bdccee38750bc1",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "94447ca479852d29aeddca75c28f7104df3c3196d7b6d83061879e339946805c": {
                "EndpointID": "b047d090f446ac49747d3c37d63e4307be745876db7f0ceef7b311cbba615f48",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "9001"
        }
    }
]
```

您可以通过断开容器的网络连接来从网络中删除该容器。为此，您需要提供网络名称和容器名称。您还可以使用容器ID。在这个例子中，名称更快。

	$ docker network disconnect bridge networktest

虽然可以从网络中断开该容器的网络连接，但是您无法删除名为`bridge`的内置`bridge`网络。网络是将容器与其他容器或其他网络隔离开来的最自然方式。所以，当您对Docker有更多的经验时，您会想创建您自己的网络。

## 创建自己的网桥

Docker引擎原生的支持bridge网络和overlay网络。bridge网络局限于运行Docker Engine的单个主机。overlay网络可以让多个主机上的容器网络相互连通，并且是更高级的主题。对于此示例，您将创建一个bridge网络:

	$ docker network create -d bridge my-bridge-network

`-d`标志告诉Docker为新网络使用`bridge`驱动程序。您可以把这个标志关掉，因为`bridge`是这个标志的默认值。继续并在您的机器上列出网络:


    $ docker network ls

    NETWORK ID          NAME                DRIVER
    7b369448dccb        bridge              bridge
    615d565d498c        my-bridge-network   bridge
    18a2866682b8        none                null
    c288470c46f6        host                host

如果您检查网络，您会发现它什么都没有。
	
	$ docker network inspect my-bridge-network

    [
        {
            "Name": "my-bridge-network",
            "Id": "5a8afc6364bccb199540e133e63adb76a557906dd9ff82b94183fc48c40857ac",
            "Scope": "local",
            "Driver": "bridge",
            "IPAM": {
                "Driver": "default",
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1/16"
                    }
                ]
            },
            "Containers": {},
            "Options": {}
        }
    ]


## 添加容器到网络

要构建协同工作并且安全的Web应用程序，请创建网络。根据定义，网络为容器提供完全的隔离。您可以在首次运行容器时向网络添加容器。

启动一个运行PostgreSQL数据库的容器，并通过`-net = my-bridge-network'标志将它连接到您的新网络:

	$ docker run -d --net = my-bridge-network --name db training / postgres

如果您检查您的`my-bridge-network`，您会看到它有一个容器。您也可以检查您的容器，看看它的连接:

    {% raw %}
    $ docker inspect --format='{{json .NetworkSettings.Networks}}'  db
    {% endraw %}

    {"my-bridge-network":{"NetworkID":"7d86d31b1478e7cca9ebed7e73aa0fdeec46c5ca29497431d3007d2d9e15ed99",
    "EndpointID":"508b170d56b2ac9e4ef86694b0a76a22dd3df1983404f7321da5649645bf7043","Gateway":"172.18.0.1","IPAddress":"172.18.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02"}}

现在继续，开始您现在熟悉的Web应用程序。这次不指定网络。

	$ docker run -d --name web training/webapp python app.py

您的`web`应用程序运行在哪个网络？`docker inspect`应用程序，您会发现它运行在默认的`bridge`网络。

    {% raw %}
    $ docker inspect --format='{{json .NetworkSettings.Networks}}'  web
    {% endraw %}

    {"bridge":{"NetworkID":"7ea29fc1412292a2d7bba362f9253545fecdfa8ce9a6e37dd10ba8bee7129812",
    "EndpointID":"508b170d56b2ac9e4ef86694b0a76a22dd3df1983404f7321da5649645bf7043","Gateway":"172.17.0.1","IPAddress":"172.17.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02"}}

然后，获取您的`web`的IP地址


    {% raw %}
    $ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web
    {% endraw %}

    172.17.0.2

现在，打开一个shell到您运行的`db`容器:

	$ docker exec -it db bash

    root@a205f0dd33b2:/# ping 172.17.0.2
    ping 172.17.0.2
    PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
    ^C
    --- 172.17.0.2 ping statistics ---
    44 packets transmitted, 0 received, 100% packet loss, time 43185ms

一会儿后，使用`CTRL-C`结束`ping`，您会发现ping失败。这是因为两个容器在不同的网络上运行。您可以解决这个问题。然后，使用`exit`命令关闭容器。

Docker网络允许您将容器附加到任意数量的网络。您还可以附加已在运行的容器到网络中。继续，将您运行的“web”应用程序附加到`my-bridge-network`。

	$ docker network connect my-bridge-web web

再次打开一个shell进入到`db`应用程序并尝试ping命令。这次只使用容器名称`web`而不是IP地址。


    $ docker exec -it db bash

    root@a205f0dd33b2:/# ping web
    PING web (172.18.0.3) 56(84) bytes of data.
    64 bytes from web (172.18.0.3): icmp_seq=1 ttl=64 time=0.095 ms
    64 bytes from web (172.18.0.3): icmp_seq=2 ttl=64 time=0.060 ms
    64 bytes from web (172.18.0.3): icmp_seq=3 ttl=64 time=0.066 ms
    ^C
    --- web ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2000ms
    rtt min/avg/max/mdev = 0.060/0.073/0.095/0.018 ms
    
`ping`显示它正在联系一个不在`bridge`网络上的IP地址，这个地址来自于`my-bridge-network`。

## 下一步

现在您知道如何为容器连通网络，参见[如何管理容器中的数据](dockervolumes.md)。