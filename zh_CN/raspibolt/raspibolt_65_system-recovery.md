[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 奖励指南：系统恢复

*难度：容易*

如果您的SD卡损坏或者您对节点进行了处理，那么手头有快速恢复图像非常方便，因此您可以快速将SD卡闪存到之前的状态。它不是完整的备份解决方案，但允许系统恢复。

### 备份外部硬盘上的必备文件

如果一切都向南，恢复还应包括来自外部硬盘的基本配置文件。区块链不能以这种方式备份，需要再次使用SCP复制，如[主要指南]（raspibolt_50_mainnet.md）中所述。

⚠️请注意LND还不能被备份：即使是稍微过时的频道也会导致你的同伴强行关闭，你会失去所有频道的资金。因此，需要手动恢复钱包。

```庆典
$ sudo su  - 比特币
$ mkdir backup_hdd
$ tar cvf backup_hdd / bitcoin.tar .bitcoin / bitcoin.conf .bitcoin / wallet.dat .bitcoin / peers.dat .bitcoin / banlist.dat
$ tar cvf backup_hdd / lnd.tar .lnd / lnd.conf
$退出
```

### 创建SD卡图像

* 关闭你的RaspiBolt
  `$ sudo shutdown now`
* 取出SD卡并将其连接到普通计算机
* 按照本指南创建磁盘映像：
  https://lifehacker.com/how-to-clone-your-raspberry-pi-sd-card-for-super-easy-r-1261113524

### 系统恢复

如果您有备用SD卡，则应通过将磁盘映像写入备份SD卡并使用它启动RaspiBolt来测试系统恢复。

如果只是SD卡有缺陷，则无需恢复外部硬盘上的文件。事实上，它会带来更多弊大于利。 ：heavy_check_mark：

---

只有当您的外部硬盘出现故障，或者您想要从头开始使用新硬盘快速设置系统时，您才能恢复外部硬盘上的基本文件，如下所示：

* 使用ssh和用户“admin”登录
* 如果连接其他硬盘，请回溯[主要指南]（raspibolt_20_pi.md）中的连接步骤并创建必要的目录。
* 停止运行服务，打开“比特币”用户会话并恢复备份的文件

```庆典
$ sudo systemctl stop lnd.service
$ sudo systemctl stop bitcoind.service

$ sudo su  - 比特币
$ cd / home / bitcoin / backup_hdd /
$ tar xvf bitcoin.tar -C / home / bitcoin
$ tar xvf lnd.tar -C / home / bitcoin
$退出
```

* ：警告：在启动服务之前，请确保在外部硬盘上准备好比特币区块链的最新副本。即使赶上几周，Raspberry Pi也需要几天时间。有关如何通过SCP下载和复制数据的信息，请参阅[主要指南]（raspibolt_50_mainnet.md）。


* 完成所有设置后，重新启动服务
  `$ sudo systemctl start bitcoind.service`
  `$ sudo systemctl start lnd.service`

您的节点现在应该赶上并很快再次运行。

------

<<返回：[奖励指南]（raspibolt_60_bonus.md）
