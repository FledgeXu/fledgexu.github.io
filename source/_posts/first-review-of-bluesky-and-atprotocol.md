---
title: Bluesky 的 AT Protocol 随想
date: 2023-01-06 21:42:57
tags:
---

今天和朋友聊起 Twitter 最近发生的事情，想起来 Bluesky，发现他们已经[公布](https://blueskyweb.xyz/blog/10-18-2022-the-at-protocol)他们的协议 [AT Protocol（Authenticated Transfer Protocol）](https://atproto.com/)了。于是去看了一下，下面是感想。

首先这个名字一下子就让我想起 A.T. Field，不知道只是巧合还是团队也是 EVA 的粉丝，这个是个有意设计出来的名字。这个协议和 Mastodon 用的 ActivityPub 一样是个Federated 的协议，团队在FAQ里解释了[为什么不用ActivityPub](https://atproto.com/guides/faq#why-not-use-activitypub)，我觉得他们指出的确实是ActivityPub的痛点。特别是用户数据保存和迁移这件事，我在实际使用Mastodon的时候见过很多真实的案例，是切实的痛点。在AT Protocol下，用户数据确实可以做到任意的迁移保存，而且不会丢失社交关系（这点其实对于社交网络协议来说来说很重要，Mastodon 在迁移之后会丢失被关注者的数据，相当于重新做一个新号）。另外因为 Bluesky 的团队有之前 IPFS 团队的成员，用户数据（Data Repository）在 AT Protocol 里使用 IPLD 储存的，如果是这样的话，AT Protocol 应该对于 IPFS 有原生的支持，之后兼容其他在 IPFS 网络上的东西应该也不困难。

因为这个协议是个 Federated 的协议，在他们的语境下 Mastodon 的 Instance 叫做 Personal Data Servers（PDS），交换方式使用的是 XRPC，我们之后再来讲 XPRC。PDS 似乎会把用户数据存储在 IPFS 上，但是 PDS 和用户之间的通讯应该是普通的HTTPS，PDS 和 PDS 之前的同步，目前还不是很清楚，因为用户数据是个 Merkle Search Tree，估计同步方式是是通过 HTTPS 传输 Root 的 CID，然后判断要不要同步这个 CID 吧。

只是 XPRC 其实就是一个是个带有 Schema 的 JSON-LD，在他们的语境里这个Schema 叫做 Lexicon（从某种意义上来说，这个是用 JSON 把 XML 重新发明了一次，不过确实 XML 的语法太繁琐了）。作为对比 ActivityPub 应该直接用的是 JSON-LD，如果我没记错的话。

在 AT Protocol，开发团队提到了一个叫做 Small-World 和 Big-World 的区别，这个我稍微有点没看懂。似乎这个 Small-world 的意思就是 PDS 只负责简单的传输和同步用户数据，而一些总体性的功能（比如热门、热搜之类的）交给爬虫和 index 服务来完成，这个倒是一个非常实际的选择。

总体来说 AT Protocol 比起 ActivityPub 来说是个更加轻便的协议，它在功能上没有 ActivityPub 那么大而全，但是协议在设计中也预留了很多可以扩展的空间，在AT Protocol 里应该只需要通过添加 Lexicon 就能添加新的功能了。不过我看了一眼现在已有的 Lexicon 没有锁推的功能😂，不过有一个可以邀请加入 Group 的功能，还挺神奇的。

AT Protocol 比起 ActivityPub 也更加注重用户数据的保存和迁移。在 AT Protocol 里用户的数据应该很容易的被保存，也很应该很容易的迁移，AT Protocol 在设计的时候也考虑到了 PDS 突然消失后，用户数据丢失的风险，在这点上比 ActivityPub 想的更加周到。另外按照协议的样子，PDS 的服务端应该不会很复杂，Self-Host 可能也会更加容易。

当用户的数据存在在 IPFS 上，那么用户可以很容易的备份和转移他们的数据，Sir Tim Berners-Lee 的 Solid 的愿景，也成了事实。

当然我这个只是一个粗略的感想，具体还有很多技术上的细节没有提及。
