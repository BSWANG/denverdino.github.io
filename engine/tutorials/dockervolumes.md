---
description: 如何管理Docker容器中的数据。
keywords: 示例，用法，卷，docker，文档，用户指南，数据，卷
redirect_from:
- /engine/userguide/containers/dockervolumes/
- /engine/userguide/dockervolumes/
- /userguide/dockervolumes/
title: 用容器管理数据
---

到目前为止，您已经了解了一些[基本的Docker概念](usingdocker.md)，看看如何使用[Docker镜像](dockerimages.md)以及了解[网络和容器之间的链接](../userguide/networking/default_network/dockerlinks.md)。在本节中，您将学习如何在Docker容器内部和容器之间管理数据。

您将了解使用Docker Engine管理数据的两种主要方法。

* 数据卷
* 数据卷容器

## 数据卷

一个*数据卷*是一个或多个容器中的特别指定的目录，这个目录绕过了[*Union File System*](../reference/glossary.md＃union-file-system)。数据卷为数据持久化与数据共享提供了几个有用的功能:

- 在容器被创建的时候数据卷被初始化。如果容器的基础镜像在指定安装点包含数据，则在数据卷初始化时将现有数据复制到新卷中。(请注意，这不适用于[安装主机目录](dockervolumes.md＃mount-a-host-directory-as-a-data-volume))。
- 数据卷可以在容器之间共享和重用。
- 数据卷的更改是直接的。
- 当您更新一个镜像时，对数据卷的更改将不会包含到镜像中。
- 即使容器本身已删除，数据卷仍然会被保留。

数据卷被设计为持久化数据，独立于容器的生命周期。因此，Docker *从不*在删除容器时自动删除卷，也不会对容器不再引用的卷做`垃圾收集`。

### 添加数据卷

您可以使用`docker create`和`docker run`命令的`-v`标志将数据卷添加到容器。您可以多次使用`-v`来挂载多个数据卷。现在，在Web应用程序容器中挂载单个卷。

```bash
$ docker run -d -P --name web -v /webapp training/webapp python app.py
```

这将在`/webapp`的容器内创建一个新卷。

> **注意:**
>您也可以在`Dockerfile`中使用`VOLUME`指令来添加一个或多个新卷到从该镜像创建的任何容器中。

### 查找卷

您可以通过使用`docker inspect`命令在主机上找到卷。

```bash
$ docker inspect web
```

命令的输出将提供有关容器配置(包括卷)的详细信息。输出应类似于以下内容:

```json
...
"Mounts": [
    {
        "Name": "fac362...80535",
        "Source": "/var/lib/docker/volumes/fac362...80535/_data",
        "Destination": "/webapp",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
]
...
```

您会注意到上面的`source`是指定主机上的源位置，`目标`是指定容器内的卷目标位置。 `RW`显示卷是否为可读/写。

### 将主机目录作为数据卷

除了使用`-v`标志创建卷，您还可以将一个目录从Docker引擎所在的主机挂载到容器中。

```bash
$ docker run -d -P --name web -v /src/webapp:/webapp training/webapp python app.py
```

此命令将主机目录`/src/webapp`挂载到容器中`/webapp`目录上。如果路径`/webapp`已经存在于容器的镜像中，`/src/webapp`安装覆盖原先的目录，但不删除之前已有的内容。一旦该安装被删除，内容可再次被访问。这与`mount`命令的预期行为一致。

`container-dir`必须是绝对路径，如`/src/docs`。 `host-dir`可以是绝对路径或`name`值。如果您为`host-dir`提供一个绝对路径，Docker绑定到您指定的路径。如果您提供一个`name`，Docker用这个`name`创建一个命名卷。

`name`值必须以字母数字字符开头，后跟`a-z0-9`，`_`(下划线)，`.`(句号)或`-`(连字符)。绝对路径以`/`(正斜线)开头。

例如，您可以为`host-dir`值指定`/foo`或`foo`。如果您提供`/foo`值，Docker引擎创建一个绑定安装(bind mount)。如果您提供不带反斜杠的`foo`，Docker引擎创建一个命名卷。

如果您在Mac或Windows上使用Docker Machine，则Docker daemon对您的MacOS或Windows文件系统的只有有限的访问权限。 Docker Machine试图自动共享您的`/Users`(macOS)或`C:\Users`(Windows)目录。因此，您可以在macOS上使用以下命令安装文件或目录。

```bash
docker run -v /Users/<path>:/<container path> ...
```

在Windows上，使用以下命令安装目录:

```bash
docker run -v c:\<path>:/c:\<container path>
```

所有其他路径都来自您的虚拟机的文件系统，因此，如果您想要使其他主机文件夹可用于共享，则需做额外的工作。在VirtualBox的下，您需要让主机文件夹变成VirtualBox中的共享文件夹。然后，您可以使用Docker`-v`标志来挂载它。

挂载主机目录在测试的时候非常有用。例如，您可以在容器将主机上的源代码挂载进入容器。然后，更改源代码并实时了解其对应用程序的影响。主机上的目录必须指定为绝对路径，如果目录不存在，Docker Engine守护程序会自动为您创建该目录。

Docker卷默认以读写模式安装，但也可以将其设置为只读。

```bash
$ docker run -d -P --name web -v /src/webapp:/webapp:ro training/webapp python app.py
```

这里您已经安装了相同的`/src/webapp`目录，但是您添加了`ro`选项来指定安装应该是只读的。

> **Note**:主机目录是和主机相关的。因为这个原因，您不能从`Dockerfile`装载主机目录，`VOLUME`指令不支持传递`host-dir`，因为构建的镜像应该是可移植的。主机目录不会在所有潜在的主机上可用。


### 将共享存储卷作为数据卷

除了在容器中装载主机目录外，一些Docker [卷插件](../ extend / plugins_volume.md)允许您配置和装载共享存储，如iSCSI，NFS或FC。

使用共享卷的一个好处是它们是与主机无关的。这意味着只要有权限访问共享存储后端并安装了插件，就可以在容器启动的任何主机上使用卷。

使用卷驱动程序的一种方法是通过`docker run`命令。卷驱动程序按名称创建卷，而不是像其他示例中一样按路径创建卷。

以下命令使用`flocker`卷驱动程序(`flocker`是多主机可移植卷的插件)创建一个名为`my-named-volume`的命名卷，并使其在`/webapp`目录上可用。在运行命令之前，安装flocker（[install flocker](https://flocker-docs.clusterhq.com/en/latest/docker-integration/manual-install.html)）。
如果您不想安装`flocker`，在下面的示例命令中用`local`替换`flocker`来使用`local`驱动。

```bash
$ docker run -d -P \
  --volume-driver=flocker \
  -v my-named-volume:/webapp \
  --name web training/webapp python app.py
```


您还可以使用`docker volume create`命令，在使用卷之前创建容器卷。

以下示例还创建了`my-named-volume`卷，这次使用`docker volume create`命令。选项的格式为`o=<key>=<value>`的键值对。

```bash
$ docker volume create -d flocker --opt o=size=20GB my-named-volume

$ docker run -d -P \
  -v my-named-volume:/webapp \
  --name web training/webapp python app.py
```

可用的插件列表，包括卷插件，可在[这里](../ extend / legacy_plugins.md)找到。

### 卷标签

像SELinux这样的标签系统需要在安装到容器中的卷内容上放置适当的标签。没有标签，安全系统可能会阻止容器内运行的进程使用卷的内容。默认情况下，Docker不会更改操作系统设置的标签。

要更改容器上下文中的标签，您可以将为挂载的卷提供任意一个后缀`:z`或`:Z`。这些后缀告诉Docker重新标记共享卷上的文件对象。 `z`选项告诉Docker两个容器共享卷内容。因此，Docker使用一个共享的内容标签标记内容。共享卷标签允许所有容器读取/写入内容。
`Z`选项告诉Docker使用私有的非共享标签来标记内容。只有当前容器可以使用私有卷。

### 将主机文件作为数据卷

`-v`标志也可以用于从主机挂载单个文件，而*不只是*目录。

```bash
$ docker run --rm -it -v ~/.bash_history:/root/.bash_history ubuntu /bin/bash
```

这将启动一个新的容器并提供给您一个bash shell，您将拥有主机的bash历史记录，当您退出容器时，主机将有在容器中键入的命令的历史记录。

> **注意**
>用于编辑文件的许多工具，包括`vi`和`sed --in-place`可能会导致inode更改。从Docker v1.1.0，这将产生一个错误，如"*sed:不能重命名./sedKdJ9Dy:设备或资源繁忙*"。如果您想要编辑已挂载的文件，通常最简单的方式是挂载他的父目录而不是挂载该文件。

## 创建和安装数据卷容器

如果您有一些要在容器之间共享的持久化数据，或者想要在非持久化的容器中使用，最好创建一个命名的数据卷容器，然后从中挂载数据。

让我们创建一个新的命名容器，这个命名容器有一个共享的数据卷。虽然此容器不运行应用程序，但它会重复使用`training/postgres`镜像，以便所有容器都使用共同的层，从而节省磁盘空间。

```bash
$ docker create -v /dbdata --name dbstore training/postgres /bin/true
```

然后可以使用`--volumes-from`标志将`/dbdata`卷挂载到另一个容器中。

```bash
$ docker run -d --volumes-from dbstore --name db1 training/postgres
```

另一个:

```bash
$ docker run -d --volumes-from dbstore --name db2 training/postgres
```

在这种情况下，如果`postgres`镜像包含一个名为`/dbdata`的目录，那么从`dbstore`容器挂载卷时会隐藏`postgres`镜像的`/dbdata`的文件。结果是只有来自`dbstore`容器的文件是可见的。

您可以使用多个`--volumes-from`参数组合来自多个容器的数据卷。要查找有关`--volumes-from`的详细信息，请参阅`run`命令中的[从容器装载卷](../reference/commandline/run.md＃mount-volumes-from-container-volumes-from)参考。

您还可以在另一个容器中通过`db1`或`db2`容器挂载来自`dbstore`容器的卷来扩展该挂载链。

```bash
$ docker run -d --name db3 --volumes-from db1 training/postgres
```

如果删除挂载了该卷的容器(包括初始`dbstore`容器或后续容器`db1`和`db2`)，卷将不会被删除。要从磁盘删除卷，您必须在最后一个拥有该卷的引用的容器上显式地调用`docker rm -v`。这允许您升级容器或在容器之间有效地迁移数据卷。

> **注意:**当移除容器时，Docker不会警告您*没有提供`-v`选项来删除它的卷*。如果在不使用`-v`选项的情况下删除容器，您可能会遇到"悬挂"卷;那些不再由容器引用的卷。
>您可以使用`docker volume ls -f dangling=true`查找悬挂卷，并使用`docker volume rm <volume name>`删除不再需要的卷。

## 备份，恢复或迁移数据卷

我们可以对卷执行的另一个有用的功能是使用它们进行备份，恢复或迁移。您可以通过使用`--volumes-from`标志创建一个新的容器来挂载该卷，如下所示:

```bash
$ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```

在这里，您已经启动了一个新容器并从`dbstore`容器中挂载了该卷。然后，您将本地主机目录挂载到`/backup`目录。最后，您传递了一个使用`tar`将`dbdata`卷的内容备份到`/backup`目录中的`backup.tar`文件的命令给容器。当命令完成并且容器停止时，我们将会有一个拥有备份数据的`dbdata`数据卷。

然后，您可以将其恢复到同一个容器，或其他地方的容器。创建一个新容器。

```bash
$ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
```

然后在新容器的数据卷中解压缩备份文件。

```bash
$ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

您可以使用上述技术，使用首选工具自动备份，迁移和恢复测试。

## 删除卷

删除容器后，Docker数据卷仍然存在。您可以创建命名卷或匿名卷。命名卷在容器外具有特定的源形式，例如`awesome:/bar`。匿名卷没有特定的源。当容器被删除时，您应该指示Docker Engine守护程序清除匿名卷。要做到这一点，使用`--rm`选项，例如:

```bash
$ docker run --rm -v /foo -v awesome:/bar busybox top
```

此命令创建一个匿名的`/foo`卷。当容器被删除时，Docker引擎删除`/foo`卷，但不删除`awesome`卷。

## 使用共享卷的重要提示

多个容器还可以共享一个或多个数据卷。但是，写入单个共享卷的多个容器可能会导致数据损坏。确保您的应用程序设计为可以读写共享数据存储。

数据卷可以直接从Docker主机访问。这意味着您可以使用普通的Linux工具读写它们。在大多数情况下，您不应该这样做，因为如果您的容器和应用程序不知道您的直接访问可能会导致数据损坏。

# 下一步

现在您已经学习了更多关于如何使用Docker的内容，我们将看到如何将Docker与[Docker Hub](https://hub.docker.com)上提供的服务相结合，包括自动构建和私有镜像仓库。

转到[在Docker Hub中存储图像](dockerrepos.md)。