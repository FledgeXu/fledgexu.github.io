---
title: X86 软路由配置 IPv6 踩坑小记
date: 2020-11-11 12:46:28
tags:
---

## 背景故事

这一次踩坑之旅的起源是一段来自内核恐慌 Telegram 群的关于 IPv6 的讨论，[Rio](https://twitter.com/RioJot) 发了关于他配置 IPv6 时候的踩坑[帖子](https://www.v2ex.com/t/722411)，而我正好一直想把家里软路由的 IPv6 配置起来，就有了这一次经历。这里非常感谢 Rio 和听众群的朋友，没有他们的帮助也就没有这次的经历。

## 关于 IPv6 的小介绍

在开始配置环境之前，我想先做一个关于 IPv6 的小介绍。介绍一下之后会涉及到的一些概念，比如：RA，slaac 等。这个介绍不会涉及到 IPv6 整体是怎么工作的，主要介绍一下在 IPv6 设备是如何获取 IPv6 地址的。

在开始介绍在 IPv6 环境之前，得先介绍一下什么是 RA。RA 也就是：Router Advertisement（路由器通告报文）是一种 ICMPv6 报文，ICMP 也就是我们日常 Ping 命令使用的报文。在 IPv6 点环境中路由发出的 RA 会携带一系列的信息告知设备如何配置自己的 IP 地址。

在 IPv6 中有多种自动配置 IP 的方式，这里我们只会接触到 slaac 和 DHCPv6，下面有个关于这两种方式区别的解释。

> **其中“自动配置”根据获取方式，又分为**
>
> ▷ 无状态（Stateless）：根据路由通告报文RA（Router Advertisement）包含的prefix前缀信息自动配置IPv6地址，组成方式是Prefix + (EUI64 or 随机)。Stateless也可以称为SLAAC（Stateless address autoconfiguration）
>
> ▷ 有状态（Stateful）：通过DHCPv6方式获得IPv6地址
>
> ——[IPv6系列-详解自动分配IPv6地址](https://cloud.tencent.com/developer/article/1517325)

因为安卓设备只支持 slaac，所以我使用了 slaac 方式配置局域网内设备的IP。

## 环境

我这里使用系统是：

```
Linux version 4.19.0-9-amd64 (debian-kernel@lists.debian.org) (gcc version 8.3.0 (Debian 8.3.0-6)) #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07)
```

机器本身是一台四个网口的软路由。

上网方式是 PPPoE 拨号上网

## 环境配置

首先配置网卡。

`/etc/network/interfaces`

```
source /etc/network/interfaces.d/*

# 本地接口
auto lo
iface lo inet loopback

# 广域网接口
allow-hotplug enp1s0

# 局域网接口
auto br0
allow-hotplug br0
iface br0 inet static
      address 192.168.2.1
      network 192.168.2.0
      netmask 255.255.255.0
      brocast 192.168.2.255
      bridge-ports enp2s0 enp3s0 enp4s0
      
# PPPoE 接口，由 pppoeconf 自动生成
auto dsl-provider
iface dsl-provider inet ppp
pre-up /bin/ip link set enp1s0 up # line maintained by pppoeconf
provider dsl-provider
iface enp1s0 inet manual
```

这里是我的的软路由的接口配置，可以看到出口网卡是 enp1s0，我会通过这个网卡进行 PPPoE 拨号上网。这个配置最后的 `dsl-provider` 是由 pppoeconf 自动生成的，我们之后会讲到。

请注意，在这里不要配置 DHCP 连接，不然内置的  dhclient 会和之后我们用到的 `wide-dhcpv6-client` 冲突。

因为我们是要配置软路由，所以我们需要启用 IPv6 转发。

在  `/etc/sysctl.conf` 添加上：

```
net.ipv6.conf.all.forwarding=2
net.ipv6.conf.default.forwarding=2

net.ipv6.conf.all.accept_ra=2
net.ipv6.conf.default.accept_ra=2

net.ipv6.conf.all.use_tempaddr=2
net.ipv6.conf.default.use_tempaddr=2
```

我们通过设置: `net.ipv6.conf.all.forwarding=2` 和 `net.ipv6.conf.default.forwarding=2` 启用了 IPv6 转发，但是根据注释：

> Uncomment the next line to enable packet forwarding for IPv6
>
> Enabling this option disables Stateless Address Autoconfiguration
>
> based on Router Advertisements for this host

开启了这个选项之后，系统将不会进行 RA 处理，也就是我们的广域网将不会有 IPv6 地址，所以我们这里手动设置了：`net.ipv6.conf.all.accept_ra=2` 和 `net.ipv6.conf.default.accept_ra=2` 来启用 RA 处理。

最后两行是启用 IPv6 的隐私扩展，具体可以阅读 Arch Wiki 的相关[介绍](https://wiki.archlinux.org/index.php/IPv6_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))。

因为 RA 是 ICMPv6 报文，所以我们要在防火墙上允许 ICMPv6 的通过。

```
iptables -A INPUT -p ipv6-icmp -j ACCEPT
iptables -A FORWARD -p ipv6-icmp -j ACCEPT
iptables -A OUTPUT -p ipv6-icmp -j ACCEPT
ip6tables -A INPUT -p ipv6-icmp -j ACCEPT
ip6tables -A FORWARD -p ipv6-icmp -j ACCEPT
ip6tables -A OUTPUT -p ipv6-icmp -j ACCEPT
```

接下来我们来配置 PPPoE 上网，运行:

```bash
apt install pppoeconf
```

在安装 `pppoeconf` 的时候会自动安装 `pppd`，`pppoeconf` 是个帮助你配置 `pppd` 的小工具，安装完成后输入：

```
pppoeconf
```

然后按照指示输入你宽带的账号密码，如果其他选项的含义不清楚请选择默认，在配置成功后，你可以通过: `poff`，`pon` 和 ` plog`，来关闭、开启 PPoE 以及显示 log。

但是在默认的情况下，`pppoeconf` 自动生成的配置文件不会启用 IPv6，我们还需要对配置文件进行一些修改。

配置文件在 `/etc/ppp/peers/` 目录下，我这里自动生成的是 `dsl-provider`。

`/etc/ppp/peers/dsl-provider`

```
noipdefault
defaultroute
replacedefaultroute
hide-password
noauth
persist
persist
maxfail 0
plugin rp-pppoe.so
nic-enp1s0
user "宽带账号"
usepeerdns
+ipv6
debug
```

这里我在末尾添加上了`+ipv6`，请注意加号，其次我还添加上了 `debug`，用于之后使用 `plog` 来 Debug 问题，此时你重新启动 PPPoE，然后输入 `ip addr show ppp0`，观察 `ppp0` 接口应该就能看到分配的 IPv6 地址了，因为我们启用了隐私扩展，所以你能看到有两个 IPv6 地址。

![ppp0](ppp0.jpeg)

接下来我们需要给内网设备也分配对应的 IPv6 地址。这里我们用到了 Prefix delegation（[前缀代理](https://zh.wikipedia.org/wiki/%E5%89%8D%E7%BC%80%E4%BB%A3%E7%90%86)），简称 PD。简单来说就是我们向我们的上级路由发送 PD 请求，上级路由会分给我们一个前缀长度小于等于64的网段，然后我们就能将个网段划分成一个或者一些 /64 的网段接着向局域网内的设备分配，此时局域网内的设备的上级路由就是我们的网关。

这里有个需要注意的地方，我们向局域网设备分配的 IP 地址也是公网地址，而不是 IPv4 时代的私有地址，不过因为上级路由是我们的网关，所以这些设备其实是在一个局域网内，并且因为这些地址都是公网地址，所以我们不需要做 NAT 转化的操作。

为了实现这个功能，我们需要使用 `wide-dhcpv6-client`。

首先安装：

```
apt install wide-dhcpv6-client
```

在安装完成后，我们需要配置 PD。

编辑：`/etc/wide-dhcpv6/dhcp6c.conf`

```
interface ppp0 {
  send ia-pd 0;
};
id-assoc pd 0 {
  # use the interface connected to your LAN
  prefix-interface br0 {
    sla-id 1;
    sla-len 4;
  };
};
```

这段配置也是来自 [Archi Wiki](https://wiki.archlinux.org/index.php/IPv6_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))，其中的 `ppp0` 应该是你广域网的接口，而 `br0` 应该是你局域网的接口，关于 `sla-len`的长度有个注释需要注意：

> **注意：** `sla-len` 应设置为满足 `(WAN-prefix) + (sla-len) = 64` 的值。这里示范的情况是针对一个长度 `/56` 的前缀，56+8=64。对于前缀长度 `/64` 的网络，`sla-len` 应为 `0`。

因为我的 ISP 分配的是个 `/60` 的网段，所以 `sla-len` 的值是 `4`，我建议大家可以先填成 `0`，然后通过运行：`dhcp6c -f -D ppp0` 命令，观察你的 ISP 分配的网段大小，然后再修改对应的值。

这里非常感谢 Rio 提供的一个新的现代化的 DHCP6c System Service 用来替换自带的 `wide-dhcpv6-client.service` 。

添加到 `/etc/systemd/system/dhcp6c.service`。

```
[Unit]
Description=WIDE DHCPv6 Client
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/sbin/dhcp6c -f ppp0
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=3
NoNewPrivileges=yes
PrivateTmp=yes
ProtectHome=yes
ProtectSystem=strict
ReadWritePaths=/run/ /var/log/
ProtectKernelTunables=yes
ProtectControlGroups=yes
SystemCallFilter=~@mount
SystemCallArchitectures=native
LockPersonality=yes
MemoryDenyWriteExecute=yes
RestrictRealtime=yes
RemoveIPC=yes

[Install]
WantedBy=multi-user.target
```

然后运行:

```
systemctl stop wide-dhcpv6-client.service
systemctl disable wide-dhcpv6-client.service
systemctl enable dhcp6c.service
```

你就可以通过 `dhcp6c.service` 来控制 `wide-dhcpv6-client` 了。

运行成功后，观察你局域网的接口，应该就能看到对应分配的地址了。

![lan](lan.jpeg)

最后需要向局域网设备发送 RA ，使用 slaac 来分配IP地址，这里我们使用了 Dnsmasq，因为 Dnsmasq 是个非常常用的软件，就不多介绍了，在 Dnsmasq 的配置文件里加上:

```
enable-ra
dhcp-range=::,constructor:br0,ra-only,slaac
```

`br0` 填入你的局域网接口。

这时你的局域网设备应该也能分配到全球唯一的 IPv6 地址了。

## 更新

DHCPv6C 在我这里的环境里会会出现一个不知为什么的 Bug，体现就是 IPV6 的在成功运行一段时间后断开。我切换到了系统自带的 DHCPCD 上，具体的配置方法可以参见 [Archi Wiki](https://wiki.archlinux.org/index.php/IPv6#With_dhcpcd)。