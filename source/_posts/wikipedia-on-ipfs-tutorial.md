---
title: 如何在IPFS上部署Wikipedia
date: 2021-02-09 17:33:19
tags:
---

## 原理

在 IPFS 上部署 Wikipedia 的原理相当的简单，主要利用了 [Kiwix](https://wiki.kiwix.org/wiki/Main_Page/zh-cn) 提供的离线版 Wikipedia 文件，将其解压并转换成静态文件之后就可以放到 IPFS 上了。这里我们主要依靠了 IPFS 团队提供的 [distributed-wikipedia-mirror](https://github.com/ipfs/distributed-wikipedia-mirror) 项目以及 OpenZIM 团队提供的 [ZIM-Tools](https://github.com/openzim/zim-tools) 工具，同样也非常感谢 IPFS 和 OpenZIM 团队在我折腾过程中给我提供的帮助。

## 配置

目前来说整个过程只能在 AMD64 Linux 上完成，主要是 ZIM-Tools 项目目前还没有在其他系统下的构建。我们将以中文版的 Wikipedia 为例。这个过程大概会消耗你大约 90G 左右的空间，所以请至少保证你的磁盘有大约 100G 左右的余量，此外这个大小会随着中文 Wikipedia 的内容增加而增长，另外你还在保证你的部署系统有至少4G的内存（我不清楚调整OOM相关的设置可不可以缓解这个问题，从而实现在小容量内存上的使用，不过我还是建议加大内存）。

首先你肯定需要一个 IPFS 的节点，这里不介绍 IPFS 节点的搭建了。除此之外为了能在 IPFS 托管 Wikiepedia，我们还需要启用几个IPFS的实验特性，这里我们以 `go-ipfs` 作为 IPFS 节点软件。

你可以通过`IPFS_PATH`环境变量来指定IPFS仓库的位置

```bash
export IPFS_PATH=/somewhere
```

其次建议使用 `badgerds`数据来存放数据，从而提升性能

```bash
ipfs init -p badgerds --empty-repo
```

首先我们需要启用 `Experimental.ShardingEnabled`

```bash
ipfs config --json 'Experimental.ShardingEnabled' true
```

这个选项解除了一个文件夹内可以存放文件数量的限制，你可以在[这个页面](https://github.com/ipfs/go-ipfs/blob/master/docs/experimental-features.md)找到 `go-ipfs` 的所有实验性特性的说明和配置。

然后你得保证你的电脑上安装了 `Cargo`、`Nodejs`、`yarn`、`git`以及一系列的构建工具（也就是 Debian 系的 `build-essential` 包中的工具）。

接下来你需要克隆并且进入 [distributed-wikipedia-mirror](https://github.com/ipfs/distributed-wikipedia-mirror)  项目。

```bash
git clone https://github.com/ipfs/distributed-wikipedia-mirror && cd distributed-wikipedia-mirror
```

然后输入

```bash
yarn
```

来构建配置环境。

因为我们之后需要用到 [ZIM-Tools](https://github.com/openzim/zim-tools) 来解压我们下载完成的 ZIM 文件，所以我们预先下载好 ZIM-Tools，这里我们以 2.1.0 为例，你可以在这里找到 ZIM-Tools 预编译的二进制包。

```bash
wget https://download.openzim.org/release/zim-tools/zim-tools_linux-x86_64-2.1.0.tar.gz
tar -xf zim-tools_linux-x86_64-2.1.0.tar.gz
mv zim-tools_linux-x86_64-2.1.0 zim
```

接下来需要下载 ZIM 文件，ZIM 文件是一种离线版 Wikipedia 保存格式。

 [distributed-wikipedia-mirror](https://github.com/ipfs/distributed-wikipedia-mirror) 项目默认提供了一种直接下载的方式，你可以输入:

```bash
bash ./tools/getzim.sh cache_update
bash ./tools/getzim.sh choose
```

进入选择菜单输入数字选择，比如你要中文版的全量 Wikipedai 的 ZIM文件，你可以选择`[12] wikipedia => [294] zh => [0] all => [0] maxi => [0] latest` 来获取到最新的中文 Wikipedia 全量 ZIM 文件，下载完成后默认存放在  `snapshots` 文件夹中。

当然因为下载服务在海外，如果你想在国内的机器上配置的话，我更推荐使用 BitTorrent 的方式来下载，这里我们将以 `wikipedia_zh_all_maxi_2021-01.zim` 为例，你可以[在这里](https://wiki.kiwix.org/wiki/Content_in_all_languages)获取到所有语言的 BitTorrent 种子文件。

为了方便起见我们这里直接使用了`aria2` 来进行 BitTorrent 下载。

```bash
wget https://download.kiwix.org/zim/wikipedia_zh_all_maxi.zim.torrent 
aria2c wikipedia_zh_all_maxi.zim.torrent -d snapshots
```

在下载完毕之后我们需要解压 zim 文件，在我们的例子里就是

```bash
mkdir tmp
./zim/zimdump dump ./snapshots/wikipedia_zh_all_maxi_2021-01.zim --dir ./tmp/wikipedia_zh_all_maxi_2021-01
```

这个解压过程会很漫长，并且会消耗大量的磁盘空间，以我的体验来说，zim文件的压缩比大概是 4:1，也就是 1G 的 ZIM 文件大概会解压出 4G 的原始文件。

在解压完毕之后，我们还需要稍微处理一下解压好的文件，这样才能让用户正常的浏览网页。

```bash
node ./bin/run ./tmp/wikipedia_zh_all_maxi_2021-01 \
  --zimfilesourceurl=https://download.kiwix.org/zim/wikipedia_zh_all_maxi.zim \
  --kiwixmainpage=User:The_other_Kiwix_guy/Landing \
  --mainpage=Wikipedia:%E9%A6%96%E9%A1%B5
```

这里的 `--zimfilesourceurl`、`--kiwixmainpage`、`--mainpage` 是三个必须选项，这里还有其他的选择可以选择，你可以输入如下命令查看所有的选项。

```bash
node ./bin/run -h
```

上面我们填的三个参数的作用分别是

- `--zimfilesourceurl`：指定 ZIM 文件的下载地址
- `--kiwixmainpage`：Kiwix 的默认主页是什么，中文 Wikipedia 这里的值是 `User:The_other_Kiwix_guy/Landing`
- `--mainpage`：网页版 Wikipedia 的默认主页是什么，中文 Wikipedia 这里的值是`Wikipedia:%E9%A6%96%E9%A1%B5`

之后应该就会在之前的解压目录中生成`index.html`等文件，来让用户可以向访问正常网页一样访问了。

在添加文件之前我们需要提升一下系统的可打开文件数量，不然中途会失败。

```
ulimit -a
ulimit -n 8096
```

```bash
ipfs add -r --cid-version 1 ./tmp/wikipedia_zh_all_maxi_2021-01/
```

这个过程可能会持续大约50个小时（我的部署），建议用类似`screen`的软件保证部署不会中断。

