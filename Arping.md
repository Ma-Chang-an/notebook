# Ping: ICMP vs. ARP

转载自[Ping: ICMP vs. ARP - Linux.com](https://www.linux.com/news/ping-icmp-vs-arp/)

网络和系统管理员一般都精通使用 ping 命令进行故障排除，但是当 ping 不起作用时，该如何处理呢？

Ping 主机通常是确定主机是否正确连接到网络的第一步。如果主机未响应 ping 请求，则通常假定主机处于离线状态。

**但是事实确实如此吗？**

如今几乎每个组织都采用防火墙来增强安全性。防火墙可以设置为阻止 Internet 控制消息协议 (ICMP) 请求，这意味着传统的 ping 命令将不起作用。之所以设置防火墙来阻止 ICMP 请求是基于这样一种理论，即如果潜在的黑客无法“看到”目标，他可能就不会攻击该目标。

但是这使得系统和网络管理更加困难。ping 命令失败业不能完全确定该主机不在线 — 用户可能启用了阻止 ping 的防火墙，但主机可能仍处于运行状态。网络管理员如果想要准确地确定主机是否关闭，需要让该主机的控制者关闭所有防火墙程序——或者至少关闭任何阻止 ICMP 的规则——这对普通用户来说可能要求太多了。

## ICMP vs. ARP

除非你能够事先确定目标主机没有使用防火墙来阻止ICMP响应，否则传统基于 ICMP的 ping 命令就不再是可靠的。那么还有其他选择吗？答案是肯定的。其中一种选择便是基于地址解析协议(Address Resolution Protocal, ARP)的 ping 命令—— `arping`命令。

要知道为什么 ARP ping 几乎可以保证有效而 ICMP ping 可能无效，我们应该了解 [ARP 在网络中的重要性](http://www.geocities.com/SiliconValley/Vista/8672/network/arp.html)。网络上的主机使用 ARP 将 IP 地址解析为媒体访问控制 (Media Access Control，MAC) 地址，MAC)地址可以解释为网络接口的唯一序列号。以太网上的主机使用 MAC 地址而不是 IP 地址进行通信。

当一台主机试图与另一台主机（在同一子网上）建立连接时，它首先需要获取第二台主机的 MAC 地址。在此过程中，主机 A 向其连接的子网的广播地址发送 ARP 请求。子网上的每台主机都会收到这个广播，具有相关 IP 地址的主机会向主机 A 发送一个带有其 MAC 地址的 ARP 回复。收到主机 B 的 ARP 回复后，主机 A 可以连接到主机 B。
**ARP 是以太网网络正常运行所必需的，因此它通常不会被防火墙阻止**。如果 ARP 请求被阻止，则没有主机能够“找到”网络上的计算机并连接到它。出于所有意图和目的，系统将从网络中拔出。

（确实存在过滤 ARP 的工具。ebtables 项目提供了这些工具。Ebtables 在功能和语法上都与 iptables 相似，但是 iptables 使用 TCP 和 UDP 协议，而 ebtables 使用 ARP。）

这种使用 ARP ping 主机的系统的一个可能缺点是 ARP 协议不是路由协议。如果您与尝试连接的主机不在同一个子网上，那么如果不先加入该子网，此方法将无法工作。因此，通过发送 ARP 请求而不是 ICMP 回声，您实际上可以保证得到答复。

下面让我们探索获取 ARP 回复的最便捷方法。

## ARP表

当您尝试 ping 一个 IP 地址时，会同时发送一个 ARP 请求。您的防火墙可能会阻止 ICMP 响应，但计算机可能会收到 ARP 回复。您计算机的 ARP 表将包含您尝试访问的主机的 IP 地址和 MAC 地址。

让我们来看看查看 ARP 表的一些方法。第一个选项是使用命令：`ip neighbor`

```shell
gerard@garion:~$ ip neighbor | grep 192.168.1.100
192.168.1.100 dev eth0 lladdr 00:40:05:01:fc:1e nud reachable
```

此处使用的实用程序是相对较新的包 iproute2 的一部分，旨在替代传统实用程序，例如 、 和 。如果你的 Linux 系统没有安装 iproute2，你可以使用 `ip ifconfig route arp arp ip` 代替。

在此示例中，IP 地址在 ARP 表中具有 MAC 地址 (lladdr) 和一个可达状态的 nud（Neighbor Unreachability Detection）。这意味着属于 IP 地址 192.168.1.100 的主机在线且处于活动状态，但显然正在阻止 ICMP 请求响应。

如果这台主机真的离线，`ip neighbor` 命令的输出将类似于：

```shell
gerard@garion:~$ ip neighbor | grep 192.168.1.101
192.168.1.101 dev eth0 nud failed
```

值为`failed`的nud意味着在发出 ARP 请求以找到此主机后，没有 ARP 回复。

## Ethereal 和 tcpdump

另一种方法是使用 tcpdump 和 Ethereal 工具查看实时网络流量，包括 ARP 请求和回复。

如果您在 192.168.1.100 IP 地址上运行常规命令 ，并运行该命令`ping tcpdump -n`或 Ethereal，您将看到类似于以下内容的输出：

```shell
12:28:59.632396 arp who-has 192.168.1.100 tell 192.168.1.1
12:28:59.632592 arp reply 192.168.1.100 is-at 00:40:05:01:fc:1e
```

第一行显示 192.168.1.1 正在尝试查找 192.168.1.100。第二行显示主机回复其 MAC 地址。这台主机肯定是在线的，即使它阻止了 ICMP 回声。

同时运行两个这样的程序可能有点麻烦。这就是 `arping` 程序的用武之地。

## arping

Arping 程序的工作方式类似于传统的 ping 命令。你给它一个 IP 地址来 ping，然后 arping 发送正确的 ARP 请求。 Arping 然后侦听 ARP 回复并打印它们（如果有），包括 ping 回复时间：

```shell
root@garion:/home/gerard# arping 192.168.1.100
ARPING 192.168.1.100
60 bytes from 00:40:05:01:fc:1e (192.168.1.100): index=0 time=190.973
usec
```

该工具使 ping 主机变得快速而简单。您不再需要运行其他工具来查看您的 ARP 表或以其他方式查看 ARP 回复。

`arping` 的另一个很好的用途是能够检测是否有多个主机配置为使用相同的 IP 地址：

```shell
root@garion:/home/gerard# arping 192.168.1.100
ARPING 192.168.1.100
60 bytes from 0a:00:3e:f1:bf:49 (192.168.1.100): index=0 time=191.151
usec
60 bytes from 00:02:b3:99:2c:f8 (192.168.1.100): index=1 time=192.419
usec
```

在这里，两台机器正在回复对同一个 IP 地址的查询，这会导致各种问题。
ARP ping 可以作为 ICMP ping 有效的替代。有了它，您就可以放心地将防火墙排除在外，并且知道失败的 ARP ping 表明一个真正必须调查的问题。