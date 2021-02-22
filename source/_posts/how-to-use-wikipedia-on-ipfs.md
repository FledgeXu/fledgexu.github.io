---
title: 如何使用IPFS版的中文Wikipedia
date: 2021-02-22 15:25:48
tags:
---

## 简介

如果你熟悉 IPFS 的话，以下是地址

CID: `bafybeierrdxblmthjga6wap3tpk53icgzb7owstz5gbd6qxcy3kspktymm`
IPNS: `k51qzi5uqu5dirl3inwnhrl6nicsaq9snlzg2aj7blt2komssvff4c9ln8tch3`

#### 这是什么？

简单来说这是个用IPFS运行的中文版 Wikipedia 的镜像，这里的镜像用的是 Kiwix 提供的 `wikipedia_zh_all_maxi_2021-01` 版本镜像。IPFS 是一个类似 BT 的分布式文件系统，具体的介绍可以参照 Wikipedia 的「星际文件系统」条目。

## 使用方法

接下来的我会介绍三种使用方法，难度从易到难。

#### 公共网关

目前来说使用公共网关来访问 IPFS 上的内容是最为简单的方法。

首先你需要找到一个公共网关的地址，我们以 IPFS 官方提供的公共网关举例:

```
https://ipfs.io
```

接下来看你需要访问 CID 的内容还是 IPNS 的内容

如果是 CID：

```
https://ipfs.io/ipfs/bafybeierrdxblmthjga6wap3tpk53icgzb7owstz5gbd6qxcy3kspktymm
```

如果是 IPNS:

```
https://ipfs.io/ipns/k51qzi5uqu5dirl3inwnhrl6nicsaq9snlzg2aj7blt2komssvff4c9ln8tch3
```

当然官方提供的公共网关已经被墙了，你可以在访问[这里](https://contributionls.github.io/public-gateway-checker/?cid=bafybeierrdxblmthjga6wap3tpk53icgzb7owstz5gbd6qxcy3kspktymm)寻找没有被墙的公共网关。

#### Brave 浏览器

目前 Brave 浏览器原生支持了 IPFS 链接，使用 Brave 浏览器访问 IPFS 上的内容也很简单。

首先你需要[下载](https://brave.com/) Brave 浏览器，然后下载完成后，在你的地址栏里输入

```
ipfs://bafybeierrdxblmthjga6wap3tpk53icgzb7owstz5gbd6qxcy3kspktymm
```

或者

```
ipns://k51qzi5uqu5dirl3inwnhrl6nicsaq9snlzg2aj7blt2komssvff4c9ln8tch3
```

第一次访问 IPFS 的链接 Brave 可能会问你要不要使用本地 IPFS 节点，建议选择使用，如果不使用本地节点，Brave 会自动 fallback 到公共网关。

#### IPFS-Desktop 

IPFS-Desktop 是 IPFS 官方开发的一款 IPFS 的图形化操作软件，你可以在[这里](https://github.com/ipfs-shipyard/ipfs-desktop/releases/latest)下载到最新版。

在你下载启动 IPFS-Desktop 之后，IPFS 会自动监听`127.0.0.1:8080`作为本地的网关。

所以你可以打开你日常使用的浏览器输入（你可以安装 [IPFS- Companion](https://github.com/ipfs-shipyard/ipfs-companion) 插件来方便访问）:

```
http://127.0.0.1:8080/ipfs/bafybeierrdxblmthjga6wap3tpk53icgzb7owstz5gbd6qxcy3kspktymm
```

或者

```
http://127.0.0.1:8080/ipns/k51qzi5uqu5dirl3inwnhrl6nicsaq9snlzg2aj7blt2komssvff4c9ln8tch3
```

来访问。

## 一些问题

本中文 Wikipedia 只是一个静态的镜像，无法添加新的内容，另外目前也不支持搜索功能。不过你可以通过自己改变URL来访问不同的页面，比如:

```
ipfs://bafybeierrdxblmthjga6wap3tpk53icgzb7owstz5gbd6qxcy3kspktymm/wiki/计算机科学
```

你只需要改变链接 `wiki` 后面的名字就能访问不同的页面。

## 这个项目是如何持续的？

这个项目是完全免费的，现在也是之后也是，随着用户的增多这个项目消失的可能性也就越小。如果你愿意帮助分发本项目或者想自己建立一个镜像请看之后的链接。但是目前维护这个项目每个月还是会有一定的成本。目前这个成本我可以自己承担，另外我也持有了一定的 FileCoin（IPFS 上层的激励币，另外这个不是投资建议，请不要盲目投资加密货币），如果你希望这个项目可以长期维持，我非常建议你了解甚至开发 IPFS 相关的项目，这样我手上的 FileCoin 就可以升值了，也就可以抵消相关的成本了。

另外我也接受虚拟货币的捐款。

FileCoin 捐款地址: `f1sbsblwklyr4drmi3ajzez5vglgvpwxudgtuyqqy`

## 如何帮助分发和如何建立自己的镜像

请参照我写的[另一篇文章](https://blog.otakusaikou.com/2021/02/09/wikipedia-on-ipfs-tutorial/).