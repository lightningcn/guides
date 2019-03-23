[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 额外指南：附加脚本

*难度：容易*

以下脚本由[RobClark56]（https://github.com/robclark56）创建，有助于获得更好的系统概述。

作为用户“admin”，下载脚本，使其可执行并将它们复制到全局bin文件夹。

### 平衡

！[]（图像/ 60_balance.png）

```
$ cd / home / admin / download /
$ wget https://raw.githubusercontent.com/Stadicus/guides/master/raspibolt/resources/lnbalance
$ chmod + x lnbalance
$ sudo cp lnbalance / usr / local / bin
$ cd
$ lnbalance
```


### 通道

！[]（图像/ 60_channels.png）

```
$ cd / home / admin / download /
$ wget https://raw.githubusercontent.com/Stadicus/guides/master/raspibolt/resources/lnchannels
$ chmod + x lnchannels
$ sudo cp lnchannels / usr / local / bin
$ cd
$ lnchannels
```

### 别名
别名是命令的快捷方式，可以节省时间并使执行常见和频繁命令变得更容易。以下别名不以奇特的方式显示信息，但它们使执行命令更容易。

#####  -  Testnet
* 以Admin身份登录，以nano打开.bashrc文件
```sudo nano / home / admin / .bashrc```
* 将以下行添加到.bashrc文件的末尾
```
alias lndstatus ='sudo journalctl -f -u lnd'
alias bitcoindstatus ='sudo tail -f /home/bitcoin/.bitcoin/testnet3/debug.log'
别名unlock ='lncli unlock'
别名newaddress ='lncli --network = testnet newaddress np2wkh'
别名txns ='lncli --network = testnet listchaintxns'
别名getinfo ='lncli --network = testnet getinfo'
别名walletbalance ='lncli --network = testnet walletbalance'
alias peers ='lncli --network = testnet listpeers'
alias channels ='lncli --network = testnet listchannels'
别名channelbalance ='lncli --network = testnet channelbalance'
别名pendingchannels ='lncli --network = testnet pendingchannels'
别名openchannel ='lncli --network = testnet openchannel'
别名connect ='lncli --network = testnet connect'
alias payinvoice ='lncli --network = testnet payinvoice'
alias addinvoice ='lncli --network = testnet addinvoice'
```
* 执行source命令以注册对.bashrc文件的更改
```> source /home/admin/.bashrc```

#####  - 主网
切换到mainnet时，请遵循** Testnet **部分，但从代码中删除所有的--network = testnet实例。

------

<<返回：[奖励指南]（raspibolt_60_bonus.md）
