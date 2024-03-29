---
title: IPFS 网络如何保存文件
date: 2021-03-26 14:04:43
tags:
---

因为我经常写一些和 IPFS 相关的文章，我发现不了解 IPFS 的普通人经常问的一个问题是：「是不是我只要把文件上传到 IPFS 上，这个文件就能分布式保存、永不消失了？」。这个是个不易回答的问题，因为正确的答案是：「是也不是」。为了解释这个答案，我们需要了解一下 IPFS 网络是如何运作的。

假设下图是 IPFS 网络。

![](how-ipfs-save-files/empty_network.png)

在 IPFS 网络中有三种操作：Add（添加）、Get（获取）以及 Pin（固定）。

假设我们使用 Add 操作，将一个文件添加到 IPFS 网络中之后，IPFS 网络会变成下面的样子。（我们用绿色来表示，一个 IPFS 节点里有文件的缓存）
![](how-ipfs-save-files/added_network.png)

这个时候你只是将文件添加进了 IPFS 的缓存之中，当缓存满了以后，IPFS 会触发所谓的「GC（垃圾回收）」将多余的没有被「固定」的文件删除。所以如果你只是添加了文件没有固定文件而且还没有向别人分享文件的话，在触发垃圾回收之后，IPFS 网络又会变成空的状态了。
![](how-ipfs-save-files/empty_network.png)

当然你也可以选择固定你缓存中的文件。固定完成之后，就不会被垃圾回收所删除了。（我们用蓝色来表示，IPFS 节点固定了某个文件）

![](how-ipfs-save-files/pined_network.png)

这时候你就会问，那么这样不还是只有本地保存了文件，说好的分布式保存呢？对，确实没错。如果你想要 IPFS 的分布式发挥它的能力，你得分享你的文件。

当你分享了文件之后，有一部分人获取了这个文件之后 IPFS 网络的样子。
![](how-ipfs-save-files/shared_network.png)

当然这些人中，可能会有人觉得这个文件对他也很重要，他也选择了固定这个文件。

![](how-ipfs-save-files/shared_and_pined_network.png)

因为 IPFS 网络每个节点只要有缓存就可以帮助分发一个文件（实际上情况下，你的缓存里有这个文件的碎片就可以帮助分发，不一定需要完整的文件，因为不同文件可能会有相同的分块，这样可以加速整个网络和节约储存空间）。假设我们遇见一个情况，我们上传且分享了一个文件，但是一个不小心，弄丢了本地的IPFS缓存。也就是像这样。
![](how-ipfs-save-files/lost_network.png)

但是因为这时 IPFS 网络中有其他的节点帮助你保存了这个文件，你仍旧可以重新获得这个文件。

![](how-ipfs-save-files/fetched_network.png)

当然在这种情况下，即使文件最初的发布者不再拥有这个文件了，想要获取这个文件的人，也可以从其他节点那里获取到这个文件。（我们假设左上角的节点为最初的发布者）。
![](how-ipfs-save-files/other_nodes_network.png)