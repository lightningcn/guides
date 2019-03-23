[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [Lightning]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[Bonus]（raspibolt_60_bonus.md）]  -  [[FAQ]（raspibolt_faq.md）]  -  [**更新** ]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

# 更新

我试着在这里跟踪我对指南的更改，以便您可以看到我稍后更新了哪些部分。

### 2018-07

* ** Raspberry Pi **：由于反复出现问题而更改了交换文件设置。谢谢，** masonicboom **！
* **奖金**：
  * 系统概述：检查比特币核心公共可用性的新方法。谢谢，** robclark56 **！
  * Shango钱包：新设置和公共互联网访问
  * Electrum Personal Server：更新的安装程序
* **常见问题**：如何升级LND。谢谢，** giulnz **！

### 2018-05

* **奖金**：添加了Shango手机钱包
* **奖金**：其他助手脚本lnbalance / lnchannels。谢谢，** robclark56 **！

### 2018年4月4日

* ** Raspberry Pi **：添加了使用现有MacOS HFS +硬盘的部分。谢谢，** masonicboom **！
* ** Lightning **：将`lnd`安装更新到版本0.4.1-beta（解决“巨大的日志文件”问题）
* **奖金**：使用lnd public ip扩展“系统概述”脚本并包含`jq`安装。
  谢谢，** robclark56 **和** zavan **！

### 2018-03-30：Electrum Personal Server

* ** Raspberry Pi **：为Electrum Personal Server（EPS）添加了UFW规则，并配置`sudo`无需密码输入即可工作
* **比特币**：为EPS启用钱包
* **奖金**：添加如何设置EPS的部分

### 2018年3月28日

* ** Raspberry Pi **：增加系统文件描述符限制
* ** Lightning **：添加`LimitNOFILE = 128000`以增加文件描述符的数量
* **主网**：第一个条目“大日志文件”的新部分“已知问题”
* **奖金**：添加到网站导航，修复raspibolt-welcome脚本中的小问题

### 2018年3月25日

* ** Raspberry Pi **：另外在`raspi-config`中设置`等待网络启动'
* ** Lightning **：更改了`getpublicip.service`以进一步改善端口绑定问题。

### 2018年3月23日

* ** Raspberry Pi **：不是禁用交换文件，而是将其移动到外部硬盘。如果已经删除，请首先使用`sudo apt-get install dphys-swapfile`重新安装交换实用程序。
* **比特币**和**闪电**：调整系统单位文件`bitcoind.service`，`getpublicip.service`和`lnd.service`以解决将bitcoind绑定到端口18333的问题（参见[讨论]在bitcointalk.org上的（https://bitcointalk.org/index.php?topic=3179045.msg32917243#msg32917243）。谢谢，** @ whywefightnet **！
* **比特币**：添加了“掌握比特币”和“从命令行学习比特币”的PDF版本
* ** Lightning **：在“lnd.conf”中将“debuglevel”设置为“info”以避免大量日志文件
* **奖金**：添加了新的[奖励部分]（raspibolt_60_bonus.md）（尚未进入网站导航）

### 2018年3月22日

* **主网**：更稳定地切换到主网，更保守的钱包创建
* **主网**：`lncli`的有用示例。谢谢** @ raindogdance **！
* ** Mainnet **：将更新的`bitcoin.conf`复制到用户“admin”以获取凭据

### 二零一八年三月二十零日

* **照明**：当未创建蛋白杏仁饼干时，添加对LND问题890的引用。
* **比特币**和**闪电**：将“bitcoind”和“lnd”的凭证复制到用户“admin”主目录。因为这是超级用户无论如何总是切换到用户“比特币”会话是没有意义的。
