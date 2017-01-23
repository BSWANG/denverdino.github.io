---
description: 如何使用您Docker镜像。
keywords: 文档，Docker指南，docker指南，docker，docker平台，docker.io，Docker镜像，Docker镜像，镜像管理，Docker repos，Docker仓库，docker，docker标签，docker标签，
redirect_from:
- /engine/userguide/containers/dockerimages/
- /engine/userguide/dockerimages/
title: 构建您自己的镜像
---

每次您运行`docker run`命令的时候,您需要指定您想使用的镜像。在本指南的前面部分您使用的是已经存在的Docker镜像，例如`ubuntu`镜像和`training/webapp`镜像。
您还会发现Docker会在Docker主机上存储下载好的镜像。 如果
镜像不是已经存在于主机上，那么它将从镜像仓库下载:默认情况下是[Docker Hub Registry](https://hub.docker.com)。

在本节中，我们将更多地探讨Docker镜,包含以下部分:

* 在Docker本地主机上管理和使用镜像。
* 创建基本镜像。
* 上传镜像到[Docker Hub Registry](https://hub.docker.com)。

## 列出主机上的镜像

让我们从列出本地主机上的镜像开始。 您可以使用`docker images`命令列出本地主机镜像:

    $ docker images

    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              14.04               1d073211c498        3 days ago          187.9 MB
    busybox             latest              2c5ac3f849df        5 days ago          1.113 MB
    training/webapp     latest              54bb4e8718e8        5 months ago        348.7 MB

您可以看到您在之前的用户指南中看到过的镜像.每个镜像都是在您用它运行容器的时候从[Docker Hub](https://hub.docker.com) 上下载的.当您列出镜像时，您会在列表中看到三个关键的信息。


* 他们来自哪个镜像仓库，例如`ubuntu'。
* 每个镜像的标签，例如`14.04`。
* 每个镜像的镜像ID。


> **提示:**
> 您可以使用[第三方dockviz工具](https://github.com/justone/dockviz)
> 或[Image layers site](https://imagelayers.io/)来显示
> 镜像数据的可视化信息。

镜像仓库可能包含镜像的多个版本。 就`ubuntu`镜像而言，您可以看到多个版本涵盖Ubuntu 10.04,12.04，12.10,13.04,13.10和14.04。 每个版本由标签标识，您可以通过如下方式引用一个镜像的某个版本:

    ubuntu:14.04

所以当您运行一个容器，您可以这样引用一个镜像:

    $ docker run -t -i ubuntu:14.04 /bin/bash

如果您想运行一个Ubuntu 12.04镜像，您可以使用:

    $ docker run -t -i ubuntu:12.04 /bin/bash

如果您没有指定一个版本标签，例如您只使用了`ubuntu`，这时Docker会默认使用`ubuntu:latest`镜像。

> **提示:**
> 您应该总是指定一个镜像标签，例如`ubuntu:14.04`。
> 这样，您总是知道您使用的是镜像的哪个版本。
> 这对于故障排除和调试非常有用。

## 获取新镜像

那么如何获得新的镜像？ Docker会自动下载那些您想要使用的但是目前不存在于您Docker主机上的镜像。 但这可能潜在的增加启动容器的时间。 如果您想预加载一个镜像,您
可以使用`docker pull`命令下载它。 假设您想要下载`centos`镜像,使用以下命令。

    $ docker pull centos

    Using default tag: latest
    latest: Pulling from library/centos
    f1b10cd84249: Pull complete
    c852f6d61e65: Pull complete
    7322fbe74aa5: Pull complete
    Digest: sha256:90305c9112250c7e3746425477f1c4ef112b03b4abe78c612e092037bfecc3b7
    Status: Downloaded newer image for centos:latest

您可以看到，镜像的每一层都被下载下来，现在您可以从这个镜像运行一个容器而不必等待下载镜像。

    $ docker run -t -i centos /bin/bash

    bash-4.1#

## 查找镜像

Docker的一个特点是很多人因为各种各样的目的创建Docker镜像，其中许多已上传到[Docker Hub](https://hub.docker.com)。 您可以在[Docker Hub](https://hub.docker.com)网站上搜索这些镜像。

![indexsearch](search.png)

您也可以通过`docker search`命令行搜索镜像.假设您的团队需要一个已经安装好Ruby和Sinatra的镜像来做Web应用开发相关的工作.您可以通过`docker search`命令去搜索所有合适的名字包含`sinatra`的镜像.

    $ docker search sinatra
    NAME                                   DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    training/sinatra                       Sinatra training image                          0                    [OK]
    marceldegraaf/sinatra                  Sinatra test app                                0
    mattwarren/docker-sinatra-demo                                                         0                    [OK]
    luisbebop/docker-sinatra-hello-world                                                   0                    [OK]
    bmorearty/handson-sinatra              handson-ruby + Sinatra for Hands on with D...   0
    subwiz/sinatra                                                                         0
    bmorearty/sinatra                                                                      0
    . . .

您会看到命令返回了许多名字包含`sinatra`的镜像列表.您已经收到了一个包含镜像名称,镜像描述,Stars数量(它衡量该镜像在社区的受喜爱程度,如果一个用户喜欢这个镜像,他们会给它加一颗星)以及自动构建状态的镜像列表.
[Official Repositories](/docker-hub/official_repos)是一个由Docker官方支持并细心维护的Docker仓库.自动构建仓库允许您验证一个镜像的源和内容.

您已查看可供使用的镜像，并决定使用
`training/sinatra`镜像。 到目前为止，您已经看到了两种类型的镜像仓库，
像`ubuntu`，它们被称为基础镜像或根镜像。 这些基础镜像由Docker Inc提供，并且已构建，验证和支持。 这些都可以通过它们的单个单词的名称来标识。

您还看到过用户镜像，例如您选择的`training/sinatra`镜像。 用户镜像归属于Docker社区里的某个成员，并且由他们构建和维护。 您可以很容易的识别哪些是用户的镜像,因为他们镜像名称前面都加上了创建它们的用户的用户名，这里是`training`。

## 拉取我们的镜像

您已经确定了一个合适的镜像，`training/sinatra`，现在您可以使用`docker pull`命令下载它。

    $ docker pull training/sinatra

您的团队现在可以通过运行自己的容器来使用此镜像。

    $ docker run -t -i training/sinatra /bin/bash

    root@a8cb6ce02d85:/#

## 创建我们自己的镜像

您的团队已经发现`training/sinatra`镜像非常有用，但它不完全
符合他们的需求，您需要对它做一些修改。有两种方法可以更新和创建镜像。

1.您可以更新一个从该镜像创建的容器，并将结果提交到该镜像。

2.您可以使用`Dockerfile`来指定创建镜像的指令。


### 更新和提交镜像

要更新镜像，您需要先从要更新的镜像创建容器。

    $ docker run -t -i training/sinatra /bin/bash

    root@0b2616b0e5a8:/#

> **注意:**
>记下已创建的容器ID，'0b2616b0e5a8`，因为您在接下来会用到它。

在我们运行的容器中，我们先更新Ruby:

    root@0b2616b0e5a8:/# apt-get install -y ruby2.0-dev ruby2.0

现在让我们添加`json` gem。

    root@0b2616b0e5a8:/# gem2.0 install json

一旦这已经完成，我们使用`exit`命令退出我们的容器。

现在您有一个容器，您想要改变它的内容。然后，您可以使用`docker commit`命令将此容器的副本提交到镜像。
	
	$ docker commit -m“添加了json gem”-a“Kate Smith” \
	0b2616b0e5a8 ouruser/sinatra:v2
	
	4f177bd27a9ff0f6dc2a830403925b5360bfe0b93d476f7fc3231110e7f71b1c

这里您使用了`docker commit`命令。您指定了两个标志:`-m`和`-a`。 `-m`标志允许我们指定一个提交消息，就像您在版本控制系统上提交一样。 `-a'标志允许我们为我们的更新指定一个作者。

您还指定了要从ID为“0b2616b0e5a8”(您之前记录的ID)的容器创建此新镜像，并且已为镜像指定了Target:

    ouruser/sinatra:v2

将这个镜像Target拆解下。您正在写这个镜像由一个新用户`ouruser`组成。您还指定了镜像的名称，在这里您保留原始镜像名称“sinatra”。最后，您要为新镜像指定一个标签:`v2`。

然后，您可以使用`docker images`命令查看我们的新的`ouruser/sinatra`镜像。    

	$ docker images
    REPOSITORY          TAG     IMAGE ID       CREATED       SIZE
    training/sinatra    latest  5bc342fa0b91   10 hours ago  446.7 MB
    ouruser/sinatra     v2      3c59e02ddd1a   10 hours ago  446.7 MB
    ouruser/sinatra     latest  5db5f8471261   10 hours ago  446.7 MB

要使用我们的新镜像创建容器，您可以:

    $ docker run -t -i ouruser/sinatra:v2 /bin/bash

    root@78e82f680994:/#

### 从`Dockerfile`构建一个镜像

使用`docker commit`命令是一个简单的扩展镜像的方法，但是它不容易在一个团队中共享镜像的开发过程。相反，您可以使用一个新的命令，`docker build`，从头开始构建新的镜像。

为此，您创建一个`Dockerfile`，它包含一组指令信息，告诉Docker如何构建我们的镜像。

首先，创建一个目录和一个`Dockerfile`。

	$ mkdir sinatra

	$ cd sinatra

	$ touch Dockerfile

如果您在Windows上使用Docker Machine，您可以通过`cd`到`/c/Users/your_user_name`访问您的主机目录。

每个指令创建镜像的新层。尝试一个简单的例子，为您的虚拟开发团队构建自己的Sinatra镜像。

	#这是一个注释
	从ubuntu:14.04
	MAINTAINER Kate Smith <ksmith@example.com>
	RUN apt-get update && apt-get install -y ruby​​ ruby​​-dev
	RUN gem install sinatra

检查您的`Dockerfile`。每个指令以一个大写的指令作为语句的前缀。

    INSTRUCTION 语句


> **注意:**使用`#`表示注释

第一个指令`FROM`告诉Docker我们的基础镜像来源是什么，在这种情况下，我们的新镜像基于Ubuntu 14.04镜像。 该指令使用`MAINTAINER`指令来指定是谁维护的这个新的镜像。

最后，您指定了两个`RUN`指令。 每个`RUN`指令在镜像中执行一个命令，例如安装一个包。 在这里，您要更新我们的APT缓存，安装Ruby和RubyGems，然后安装Sinatra gem。


现在让我们使用我们的`Dockerfile`并使用`docker build`命令来构建一个镜像。

    $ docker build -t ouruser/sinatra:v2 .

    Sending build context to Docker daemon 2.048 kB
    Sending build context to Docker daemon
    Step 1 : FROM ubuntu:14.04
     ---> e54ca5efa2e9
    Step 2 : MAINTAINER Kate Smith <ksmith@example.com>
     ---> Using cache
     ---> 851baf55332b
    Step 3 : RUN apt-get update && apt-get install -y ruby ruby-dev
     ---> Running in 3a2558904e9b
    Selecting previously unselected package libasan0:amd64.
    (Reading database ... 11518 files and directories currently installed.)
    Preparing to unpack .../libasan0_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking libasan0:amd64 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package libatomic1:amd64.
    Preparing to unpack .../libatomic1_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking libatomic1:amd64 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package libgmp10:amd64.
    Preparing to unpack .../libgmp10_2%3a5.1.3+dfsg-1ubuntu1_amd64.deb ...
    Unpacking libgmp10:amd64 (2:5.1.3+dfsg-1ubuntu1) ...
    Selecting previously unselected package libisl10:amd64.
    Preparing to unpack .../libisl10_0.12.2-1_amd64.deb ...
    Unpacking libisl10:amd64 (0.12.2-1) ...
    Selecting previously unselected package libcloog-isl4:amd64.
    Preparing to unpack .../libcloog-isl4_0.18.2-1_amd64.deb ...
    Unpacking libcloog-isl4:amd64 (0.18.2-1) ...
    Selecting previously unselected package libgomp1:amd64.
    Preparing to unpack .../libgomp1_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking libgomp1:amd64 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package libitm1:amd64.
    Preparing to unpack .../libitm1_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking libitm1:amd64 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package libmpfr4:amd64.
    Preparing to unpack .../libmpfr4_3.1.2-1_amd64.deb ...
    Unpacking libmpfr4:amd64 (3.1.2-1) ...
    Selecting previously unselected package libquadmath0:amd64.
    Preparing to unpack .../libquadmath0_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking libquadmath0:amd64 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package libtsan0:amd64.
    Preparing to unpack .../libtsan0_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking libtsan0:amd64 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package libyaml-0-2:amd64.
    Preparing to unpack .../libyaml-0-2_0.1.4-3ubuntu3_amd64.deb ...
    Unpacking libyaml-0-2:amd64 (0.1.4-3ubuntu3) ...
    Selecting previously unselected package libmpc3:amd64.
    Preparing to unpack .../libmpc3_1.0.1-1ubuntu1_amd64.deb ...
    Unpacking libmpc3:amd64 (1.0.1-1ubuntu1) ...
    Selecting previously unselected package openssl.
    Preparing to unpack .../openssl_1.0.1f-1ubuntu2.4_amd64.deb ...
    Unpacking openssl (1.0.1f-1ubuntu2.4) ...
    Selecting previously unselected package ca-certificates.
    Preparing to unpack .../ca-certificates_20130906ubuntu2_all.deb ...
    Unpacking ca-certificates (20130906ubuntu2) ...
    Selecting previously unselected package manpages.
    Preparing to unpack .../manpages_3.54-1ubuntu1_all.deb ...
    Unpacking manpages (3.54-1ubuntu1) ...
    Selecting previously unselected package binutils.
    Preparing to unpack .../binutils_2.24-5ubuntu3_amd64.deb ...
    Unpacking binutils (2.24-5ubuntu3) ...
    Selecting previously unselected package cpp-4.8.
    Preparing to unpack .../cpp-4.8_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking cpp-4.8 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package cpp.
    Preparing to unpack .../cpp_4%3a4.8.2-1ubuntu6_amd64.deb ...
    Unpacking cpp (4:4.8.2-1ubuntu6) ...
    Selecting previously unselected package libgcc-4.8-dev:amd64.
    Preparing to unpack .../libgcc-4.8-dev_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking libgcc-4.8-dev:amd64 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package gcc-4.8.
    Preparing to unpack .../gcc-4.8_4.8.2-19ubuntu1_amd64.deb ...
    Unpacking gcc-4.8 (4.8.2-19ubuntu1) ...
    Selecting previously unselected package gcc.
    Preparing to unpack .../gcc_4%3a4.8.2-1ubuntu6_amd64.deb ...
    Unpacking gcc (4:4.8.2-1ubuntu6) ...
    Selecting previously unselected package libc-dev-bin.
    Preparing to unpack .../libc-dev-bin_2.19-0ubuntu6_amd64.deb ...
    Unpacking libc-dev-bin (2.19-0ubuntu6) ...
    Selecting previously unselected package linux-libc-dev:amd64.
    Preparing to unpack .../linux-libc-dev_3.13.0-30.55_amd64.deb ...
    Unpacking linux-libc-dev:amd64 (3.13.0-30.55) ...
    Selecting previously unselected package libc6-dev:amd64.
    Preparing to unpack .../libc6-dev_2.19-0ubuntu6_amd64.deb ...
    Unpacking libc6-dev:amd64 (2.19-0ubuntu6) ...
    Selecting previously unselected package ruby.
    Preparing to unpack .../ruby_1%3a1.9.3.4_all.deb ...
    Unpacking ruby (1:1.9.3.4) ...
    Selecting previously unselected package ruby1.9.1.
    Preparing to unpack .../ruby1.9.1_1.9.3.484-2ubuntu1_amd64.deb ...
    Unpacking ruby1.9.1 (1.9.3.484-2ubuntu1) ...
    Selecting previously unselected package libruby1.9.1.
    Preparing to unpack .../libruby1.9.1_1.9.3.484-2ubuntu1_amd64.deb ...
    Unpacking libruby1.9.1 (1.9.3.484-2ubuntu1) ...
    Selecting previously unselected package manpages-dev.
    Preparing to unpack .../manpages-dev_3.54-1ubuntu1_all.deb ...
    Unpacking manpages-dev (3.54-1ubuntu1) ...
    Selecting previously unselected package ruby1.9.1-dev.
    Preparing to unpack .../ruby1.9.1-dev_1.9.3.484-2ubuntu1_amd64.deb ...
    Unpacking ruby1.9.1-dev (1.9.3.484-2ubuntu1) ...
    Selecting previously unselected package ruby-dev.
    Preparing to unpack .../ruby-dev_1%3a1.9.3.4_all.deb ...
    Unpacking ruby-dev (1:1.9.3.4) ...
    Setting up libasan0:amd64 (4.8.2-19ubuntu1) ...
    Setting up libatomic1:amd64 (4.8.2-19ubuntu1) ...
    Setting up libgmp10:amd64 (2:5.1.3+dfsg-1ubuntu1) ...
    Setting up libisl10:amd64 (0.12.2-1) ...
    Setting up libcloog-isl4:amd64 (0.18.2-1) ...
    Setting up libgomp1:amd64 (4.8.2-19ubuntu1) ...
    Setting up libitm1:amd64 (4.8.2-19ubuntu1) ...
    Setting up libmpfr4:amd64 (3.1.2-1) ...
    Setting up libquadmath0:amd64 (4.8.2-19ubuntu1) ...
    Setting up libtsan0:amd64 (4.8.2-19ubuntu1) ...
    Setting up libyaml-0-2:amd64 (0.1.4-3ubuntu3) ...
    Setting up libmpc3:amd64 (1.0.1-1ubuntu1) ...
    Setting up openssl (1.0.1f-1ubuntu2.4) ...
    Setting up ca-certificates (20130906ubuntu2) ...
    debconf: unable to initialize frontend: Dialog
    debconf: (TERM is not set, so the dialog frontend is not usable.)
    debconf: falling back to frontend: Readline
    debconf: unable to initialize frontend: Readline
    debconf: (This frontend requires a controlling tty.)
    debconf: falling back to frontend: Teletype
    Setting up manpages (3.54-1ubuntu1) ...
    Setting up binutils (2.24-5ubuntu3) ...
    Setting up cpp-4.8 (4.8.2-19ubuntu1) ...
    Setting up cpp (4:4.8.2-1ubuntu6) ...
    Setting up libgcc-4.8-dev:amd64 (4.8.2-19ubuntu1) ...
    Setting up gcc-4.8 (4.8.2-19ubuntu1) ...
    Setting up gcc (4:4.8.2-1ubuntu6) ...
    Setting up libc-dev-bin (2.19-0ubuntu6) ...
    Setting up linux-libc-dev:amd64 (3.13.0-30.55) ...
    Setting up libc6-dev:amd64 (2.19-0ubuntu6) ...
    Setting up manpages-dev (3.54-1ubuntu1) ...
    Setting up libruby1.9.1 (1.9.3.484-2ubuntu1) ...
    Setting up ruby1.9.1-dev (1.9.3.484-2ubuntu1) ...
    Setting up ruby-dev (1:1.9.3.4) ...
    Setting up ruby (1:1.9.3.4) ...
    Setting up ruby1.9.1 (1.9.3.484-2ubuntu1) ...
    Processing triggers for libc-bin (2.19-0ubuntu6) ...
    Processing triggers for ca-certificates (20130906ubuntu2) ...
    Updating certificates in /etc/ssl/certs... 164 added, 0 removed; done.
    Running hooks in /etc/ca-certificates/update.d....done.
     ---> c55c31703134
    Removing intermediate container 3a2558904e9b
    Step 4 : RUN gem install sinatra
     ---> Running in 6b81cb6313e5
    unable to convert "\xC3" to UTF-8 in conversion from ASCII-8BIT to UTF-8 to US-ASCII for README.rdoc, skipping
    unable to convert "\xC3" to UTF-8 in conversion from ASCII-8BIT to UTF-8 to US-ASCII for README.rdoc, skipping
    Successfully installed rack-1.5.2
    Successfully installed tilt-1.4.1
    Successfully installed rack-protection-1.5.3
    Successfully installed sinatra-1.4.5
    4 gems installed
    Installing ri documentation for rack-1.5.2...
    Installing ri documentation for tilt-1.4.1...
    Installing ri documentation for rack-protection-1.5.3...
    Installing ri documentation for sinatra-1.4.5...
    Installing RDoc documentation for rack-1.5.2...
    Installing RDoc documentation for tilt-1.4.1...
    Installing RDoc documentation for rack-protection-1.5.3...
    Installing RDoc documentation for sinatra-1.4.5...
     ---> 97feabe5d2ed
    Removing intermediate container 6b81cb6313e5
    Successfully built 97feabe5d2ed

在这里您指定了`docker build`命令，并使用`-t`标志来识别我们的新镜像属于用户`ouruser`，仓库名称`sinatra`，并给它打上标签`v2`。

您还使用`.`指定了我们的`Dockerfile`的位置，表示当前目录中的一个`Dockerfile`。

> **注意:**
>您也可以指定一个`Dockerfile`的路径。

现在您可以看到构建正在进行。Docker做的第一件事是上传构建上下文:基本上是您正在构建的目录的内容。这是因为实际上构建了镜像的工作是由Docker守护进程做的，并且它需要本地环境的上下文来完成构建工作。

接下来您可以看到`Doc​​kerfile`中的每条指令都是逐步执行的。您可以看到每个步骤创建一个新容器，在该容器中运行指令，然后提交该更改，就像您之前看到的`docker commit`工作流程。当所有的指令执行后，剩下一个ID为`97feabe5d2ed`镜像(也标记为`ouruser/sinatra:v2`)，所有的中间容器将被清除干净。

> **注意:**
>无论存储驱动程序如何，镜像都不能超过127个层。
>此限制应用到全局设置，以鼓励优化镜像的整体大小。

然后，您可以从我们的新镜像创建一个容器。

	$ docker run -t -i ouruser/sinatra:v2 /bin/bash
	root@8196968dac35:/＃

> **注意:**
>这只是创建镜像的简单介绍。我们
>跳过了一大堆您可以使用的其他说明。我们在本指南后面部分的说明将会看到更多
>或者您可以参考[`Dockerfile`](../reference/builder.md)
>参考每个指令的详细描述和示例。
>为了帮助您写一个清晰，可读，可维护的“Dockerfile”，我们也
>写了一个[`Dockerfile`最佳实践指南](../userguide/eng-image/dockerfile_best-practices.md)。

## 在镜像上设置标签

您还可以在提交或构建现有镜像成功之后为它添加标签。我们可以使用`docker tag`命令来做到这一点。现在，给您的`ouruser/sinatra`镜像添加一个新标签。

	$ docker tag 5db5f8471261 ouruser/sinatra:devel

`docker tag`命令使用镜像的ID，这里是“5db5f8471261”，以及我们的用户名，仓库名称和新标签。

现在，使用`docker images`命令查看您的新标签。

	$ docker images ouruser/sinatra

	REPOSITORY TAG IMAGE ID CREATED SIZE
	ouruser/sinatra最新的5db5f8471261 11小时前446.7 MB
	ouruser/sinatra devel 5db5f8471261 11小时前446.7 MB
	ouruser/sinatra v2 5db5f8471261 11小时前446.7 MB

## 镜像摘要

每个使用v2或最新版本格式的镜像具有一个内容可寻址的标识符，这个标识符称为`digest`。只要用于生成镜像的输入不变，则摘要值是可预测的不变的。要列出镜像的摘要值，请使用`--digests`标志:

	$ docker images --digests | head

	REPOSITORY TAG DIGEST IMAGE ID CREATED SIZE
	ouruser/sinatra:latest sha256:cbbf2f9a99b47fc460d422812b6a5adff7dfee951d8fa2e4a98caa0382cfbdbf 5db5f8471261 11小时前446.7 MB

当向2.0 registry仓库推或拉取镜像的时候，`push`或`pull`命令的输出包括镜像摘要。您可以使用摘要值`pull`镜像。

	$ docker pull ouruser/sinatra@sha256:cbbf2f9a99b47fc460d422812b6a5adff7dfee951d8fa2e4a98caa0382cfbdbf

您还可以通过在`create`，`run`和`rmi`命令中的通过摘要引用镜像，以及在Dockerfile中的`FROM`引用镜像。
## 将镜像推送到Docker Hub

一旦您构建或创建了一个新的镜像，您可以使用`docker push`命令将其推送到[Docker Hub](https://hub.docker.com)。这允许您与其他人共享镜像，公开这个镜像或推送到[私有仓库](https://hub.docker.com/account/billing-plans/)。

	$ docker push ouruser / sinatra
    The push refers to a repository [ouruser/sinatra] (len: 1)
    Sending image list
    Pushing repository ouruser/sinatra (3 tags)
    . . .

## 从主机中删除镜像

您也可以使用docker rmi命令以某种方式删除Docker主机上的镜像[类似于containers](usingdocker.md)。

删除`training/sinatra`镜像，因为您不再需要它。

	$ docker rmi training/sinatra

	Untagged: training/sinatra:latest
	Deleted: 5bc342fa0b91cabf65246837015197eecfa24b2213ed6a51a8974ae250fedd8d
	Deleted: ed0fffdcdae5eb2c3a55549857a8be7fc8bc4241fb19ad714364cbfd7a56b22f
	Deleted: 5c58979d73ae448df5af1d8142436d81116187a7633082650549c52c3a2418f0

> **注意:**要从主机中删除镜像，请确保没有基于它的容器还处于活跃状态。

## 检查镜像和容器的大小

镜像是[按层存储](../userguide/storagedriver/imagesandcontainers.md)，并且与主机上的其他镜像共享这些层，因此真实磁盘使用量取决于主机上的镜像之间有多少层共享。在一个只读rootfs之上添加一个[可写层](../userguide/storagedriver/imagesandcontainers.md＃/container-and-layers)，容器就运行在这个可写层之上。

使用`docker history`查看主机上镜像层的大小:

	$ docker history centos:centos7
	IMAGE               CREATED             CREATED BY                                      SIZE
	970633036444        6 weeks ago         /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B
	<missing>           6 weeks ago         /bin/sh -c #(nop) LABEL name=CentOS Base Imag   0 B
	<missing>           6 weeks ago         /bin/sh -c #(nop) ADD file:44ef4e10b27d8c464a   196.7 MB
	<missing>           10 weeks ago        /bin/sh -c #(nop) MAINTAINER https://github.c   0 B

使用`docker ps -s`检查容器的大小:

	$ docker ps -s

	CONTAINER ID        IMAGE                                                          COMMAND                  CREATED              STATUS              PORTS                    NAMES               SIZE
	cb7827c19ef7        docker-docs:is-11160-explain-image-container-size-prediction   "hugo server --port=8"   About a minute ago   Up About a minute   0.0.0.0:8000->8000/tcp   evil_hodgkin        0 B (virtual 949.2 MB)


# 下一步

到现在为止，您已经看到了如何在Docker容器中构建单独的应用程序。现在，您可以通过将多个Docker容器网络连接在一起来了解如何使用Docker构建整个应用程序栈。

转到[网络容器](networkingcontainers.md)。
