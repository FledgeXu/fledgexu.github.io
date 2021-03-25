---
title: IPFS 官方中文 Wikipedia 使用指南
date: 2021-03-25 10:13:43
tags:
---

## 什么是 IPFS？

> **星际文件系统**（**InterPlanetary File System**，缩写**IPFS**）是一个旨在创建持久且分布式存储和共享文件的[网络传输协议](https://zh.wikipedia.org/wiki/网络传输协议)。它是一种[内容可寻址](https://zh.wikipedia.org/w/index.php?title=内容可寻址&action=edit&redlink=1)的[对等](https://zh.wikipedia.org/wiki/對等網路)[超媒体](https://zh.wikipedia.org/wiki/超媒體)分发协议。在IPFS网络中的节点将构成一个[分布式文件系统](https://zh.wikipedia.org/wiki/集群文件系统)。它是一个[开放源代码](https://zh.wikipedia.org/wiki/开放源代码)项目，自2014年开始由[Protocol Labs](https://zh.wikipedia.org/w/index.php?title=Protocol_Labs&action=edit&redlink=1)在开源社区的帮助下发展。其最初由[Juan Benet](https://zh.wikipedia.org/w/index.php?title=Juan_Benet&action=edit&redlink=1)设计。

用最简单的话来说 IPFS 是个 P2P网络，和我们日常可能会使用的 BT 下载的原理类似，但是 IPFS 相比于 BT 来说做了非常多的改进，使得 IPFS 的性能和扩展性都有很大的提升。

在 IPFS 网络中每一个文件都有一个独特的 CID，当你把一个文件放入 IPFS 网络中，其他的用户就可以通过 CID 来获取到这个文件而不必考虑这个文件存放在何处。此外任何用户可以选择 pin 住一个文件的 CID，从而帮助 IPFS 网络长期的保存某个文件。

 ## 什么是 Distributed Wikipedia Mirror？

[Distributed Wikipedia Mirror](https://github.com/ipfs/distributed-wikipedia-mirror) 是 IPFS 官方团队维护的一个项目。这个项目旨在于将 Wikipedia 带入 IPFS 网络，以及最终构建出一个纯分布式的 Wikipedia。目前该项目已经提供了：英语、土耳其语、缅甸语和中文的 Wikipedia 镜像。

## 如何使用？

本文将介绍如何使用中文版的 Wikipedia IPFS镜像。

镜像的地址为:

- DNSLink: `zh.wikipedia-on-ipfs.org`
- CID: `bafybeiazgazbrj6qprr4y5hx277u4g2r5nzgo3jnxkhqx56doxdqrzms6y`

**请注意本项目的 CID 地址会随着分发的 Wikipedia 镜像版本更新而改变，你可以通过访问[此地址](https://github.com/ipfs/distributed-wikipedia-mirror/blob/main/snapshot-hashes.yml)或者使用 `ipfs name resolve zh.wikipedia-on-ipfs.org` 获取到最新的 CID**

我接下来会介绍3种不同的方式来访问本镜像。

---

### 公共网关

公共网关是目前访问 IPFS 网络上内容最简单的方式，但这个也是最容易被封锁的方式。我将以官方的网关为例来演示如何使用公共网关来访问 IPFS 网络上的内容。

**官方公共网关地址: `https://ipfs.io`**

#### 使用 CID 访问镜像

如果你决定使用 CID 地址来访问镜像的话，你需要在浏览器地址栏按照如下格式输入:

```http
https://ipfs.io/ipfs/<CID>
```

在我们的例子里就是：

```http
https://ipfs.io/ipfs/bafybeiazgazbrj6qprr4y5hx277u4g2r5nzgo3jnxkhqx56doxdqrzms6y
```

#### 使用 DNSLink 地址访问

如果你决定使用 DNSLink 地址来访问镜像的话，你需要在浏览器地址栏按照如下格式输入:

```http
https://ipfs.io/ipns/<DNSLink>
```

在我们的例子里就是：

```http
https://ipfs.io/ipns/zh.wikipedia-on-ipfs.org
```

---

### Brave 浏览器

如果你在使用最新版的 [Brave 浏览器](https://brave.com/)，你可以直接使用 Brave 内置的 IPFS 节点来访问 IPFS 网络上的内容。你在第一次使用 Brave 浏览器访问 IPFS 内容时，Brave 浏览器可能会询问你是否要启用本地 IPFS 节点，建议选择启用，如果没有启用，Brave 会自动使用公共网关来访问 IPFS 网络上的内容。此外你可以通过 Brave 设置页面中 IPFS 相关的选项和内置的 IPFS-Companion 插件中的选项来调整 IPFS 节点类型。

#### 使用 CID 访问镜像

如果你决定使用 CID 地址来访问镜像的话，你需要在浏览器地址栏按照如下格式输入:

```http
ipfs://<CID>
```

在我们的例子里就是：

```http
ipfs://bafybeiazgazbrj6qprr4y5hx277u4g2r5nzgo3jnxkhqx56doxdqrzms6y
```

#### 使用 DNSLink 地址访问

如果你决定使用 DNSLink 地址来访问镜像的话，你需要在浏览器地址栏按照如下格式输入:

```http
ipns://<DNSLink>
```

在我们的例子里就是：

```http
ipns://zh.wikipedia-on-ipfs.org
```

---

### IPFS Desktop

IPFS Desktop 对于是目前普通用户使用本地 IPFS 最容易的方法，你可以在[这里](https://github.com/ipfs-shipyard/ipfs-desktop/releases/latest)下载最新版的 IPFS Desktop。在启动成功之后， IPFS-Dekstop 会默认在你本地地址的 `8080` 端口启动一个网关服务器，之后我们就可以使用这个本地的网关服务来访问 IPFS 网络上的内容了。

当然你可以通过修改 IPFS-Desktop 中的 `Gateway` 项，来修改默认的端口地址。

#### 使用 CID 访问镜像

如果你决定使用 CID 地址来访问镜像的话，你需要在浏览器地址栏按照如下格式输入:

```http
http://127.0.0.1:<port>/ipfs/<CID>
```

在我们的例子里就是：

```http
http://127.0.0.1:8080/ipfs/bafybeiazgazbrj6qprr4y5hx277u4g2r5nzgo3jnxkhqx56doxdqrzms6y
```

#### 使用 DNSLink 地址访问

如果你决定使用 DNSLink 地址来访问镜像的话，你需要在浏览器地址栏按照如下格式输入:

```http
http://127.0.0.1:<port>/ipns/<DNSLink>
```

在我们的例子里就是：

```http
http://127.0.0.1:8080/ipns/zh.wikipedia-on-ipfs.org
```