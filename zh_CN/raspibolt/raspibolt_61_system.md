[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 奖金指南：系统概述

*难度：容易*

为了快速了解系统状态，我创建了[shell脚本]（https://github.com/Stadicus/guides/blob/master/raspibolt/resources/20-raspibolt-welcome）作为“消息”运行当天“（motd）登录或按需显示。

！[MotD系统概述]（images / 60_status_overview.png）

这个脚本将以root身份运行，因此请在盲目信任我之前检查它。

```
$ sudo apt-get install jq net-tools
$ cd / home / admin / download /
$ wget https://raw.githubusercontent.com/Stadicus/guides/master/raspibolt/resources/20-raspibolt-welcome

#check脚本并退出
$ nano 20-raspibolt-welcome

＃删除现有的欢迎脚本并安装
$ sudo mv /etc/update-motd.d /etc/update-motd.d.bak
$ sudo mkdir /etc/update-motd.d
$ sudo cp 20-raspibolt-welcome /etc/update-motd.d/
$ sudo chmod + x /etc/update-motd.d/20-raspibolt-welcome
$ sudo ln -s /etc/update-motd.d/20-raspibolt-welcome / usr / local / bin / raspibolt
```

如果脚本遇到问题，理论上可能会阻止您登录。因此，我们禁用“root”用户的所有motd执行，因此您始终可以以“root”身份登录以禁用它。

```
$ sudo su
$ touch /root/.hushlogin
$退出
```

您现在可以使用`sudo raspibolt`启动脚本，每次登录时都会显示该脚本。

------

<<返回：[奖励指南]（raspibolt_60_bonus.md）
