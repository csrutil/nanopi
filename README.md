# NanoPi R2C &amp; NanoPi R2S

此文只为记录使用NanoPi一些心得体会，如果能帮助读者，那我万分荣幸。

![](http://wiki.friendlyarm.com/wiki/images/6/68/NanoPi_R2C-1.jpg)

## 设备情况

我其实有非常多的*路由*设备

- NanoPi R2S / NanoPi R2C
- J4125软路由
- TP的Wi-Fi路由器（尤其喜欢大道系列，作为AP无敌）
- 双盘位的NAS(插了一个4T的垂直盘)，准备通过syncthing做异地raid1
- 祖传K2P(自己编译的毛子固件)
- 四台矿机，一直在服役(R5 3600)
- 攒钱买Gen10 PLUS中

那么我为什么买NanoPi呢，首先第一点是小巧。

- 性能还可以
- 核心是RK3328 ARMv8，有AES指令
- 双千兆，LAN口为USB转接
- 1G的内存
- R2C的网卡换为了YT8521S

但是也有很多缺点，比如LAN口不稳定，发热量大，但是这两个都可以通过一些小手段解决。起初我是使用自己编译的OpenWrt，但是5.4内核的r8152驱动非常不稳定，只要长期处于满载状态，十分钟左右就掉线。
后来我发现，官方friendlyarm并不存在这个问题。friendlyarm的内核为5.10，驱动版本为1.11.11。因为friendlyarm的内核并不跟着mainline走，我就没尝试过自己去编译它。OpenWrt前段时间刚刚把内核升级为
5.10，但是还是测试版本，我花了一些时间去编译它，但是缺少一些驱动，并没有成功（我尝试过把5.10内核的驱动拿到5.4里面使用，但是API变动了，没成功）。还有一点就是，可以使用螃蟹🦀️的驱动，但是稳定性还是不行🙅。

## AES指令集

这里科普一下AES指令加速。首先硬件会提供一种能力，这种能力处理特定的任务会非常得快。在Linux上面，内核可以直接调用这部分能力（前提是有驱动），但是我们大部分的应用程序都是在跑在*用户层*，是没办法直接调用
硬件加速的，那么怎么办呢？为了解决这个问题，社区有两个方案，一个是cryptodev，一个是af_alg，通过这个胶水，可以让应用层的任务使用到底层的硬件加速。但是这个是远远不够的，你的应用得适配这两个胶水的API。

所以，比如你使用一些应用，是没办法硬件加速的，是纯软件加密和解密。openssl与wolfssl可以通过特定的编译参数来启用这一特性。

## r8152驱动

螃蟹🦀️的驱动负载高，必挂。所以怎么办呢，你可以等OpenWrt发布下一个大版本（5.10内核），大概率不会有掉线的问题。第二个方式是，使用armbian，但是armbian的驱动也是螃蟹的驱动。你可以自己编译一个固件，去除螃蟹的驱动
使用5.10内核的驱动。也可以加群，我发你稳定的版本。

```bash
# git clone --depth 1 https://github.com/armbian/build
sed -i '/rtl8152ver=/a return 0' lib/compilation-prepare.sh # 禁用螃蟹官方的驱动
```

那么效果怎么样呢? iperf3 满载`115G`。

![](https://i.imgur.com/wysURIC.png)

## 超频

此款Soc的频率是1.5，但是固件大都是压到了1.3。但是你可以通过魔改下dts来超频。但是随之温度也上升很多，功耗其实提高的不是很多。我的机器加了小风扇，最高频率是1.6。我使用`stress -c 4` 压测了一个小时左右，
温度为60.4°C度左右。

那么性能提升有多少呢?(其实没多少 ;)

```bash
openssl speed -evp aes-256-gcm # OpenSSL 1.1.1f  31 Mar 2020
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes  16384 bytes
aes-256-gcm      80450.22k   230070.61k   416466.86k   533012.48k   577901.91k   579873.45k

# 官方的版本
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes  16384 bytes
aes-256-gcm      80187.19k   222333.58k   399549.27k   504449.02k   542182.06k   545855.70k
```
## 日常使用

使用这个设备作为一个DNS/路由使用，上面跑了smartdns, AdGuardHome。设置好策略路由之后，稳定性非常的好。那么是否能作为主路由使用呢？如果你是千兆宽带，大量BT，那么有可能稳定性不那么好。
但是随着最新的版本发布，有可能这个情况会改善。如果你动手能力很强，手搓iptables规则，编译固件，那么毫无问题。

## TELEGRAM
Please join Telegram [Join Telegram](https://t.me/hackintash), if you have any questions

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Credits

- https://forum.armbian.com/topic/15165-nanopi-r2s-lan0-goes-offline-with-high-traffic/page/2/
- http://wiki.friendlyarm.com/wiki/index.php/NanoPi_R2C/zh
