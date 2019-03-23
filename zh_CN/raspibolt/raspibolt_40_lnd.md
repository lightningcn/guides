[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ **闪电**]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[Bonus]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

-------
### Raspberry Pi上的️⚡Lightning️⚡初学者指南
--------

# 闪电：LND

我们现在将[Lightning Labs]（http://lightning.engineering/）下载并安装LND（Lightning Network Daemon）。查看他们的[Github存储库]（https://github.com/lightningnetwork/lnd/blob/master/README.md），了解有关其开源项目和Lightning的大量信息。

```
$ cd / home / admin / download
$ rm -rf / home / admin / download / *
$ wget https://github.com/lightningnetwork/lnd/releases/download/v0.5.2-beta/lnd-linux-armv7-v0.5.2-beta.tar.gz
$ wget https://github.com/lightningnetwork/lnd/releases/download/v0.5.2-beta/manifest-v0.5.2-beta.txt
$ wget https://github.com/lightningnetwork/lnd/releases/download/v0.5.2-beta/manifest-v0.5.2-beta.txt.sig
$ wget https://keybase.io/roasbeef/pgp_keys.asc

$ sha256sum --check manifest-v0.5.2-beta.txt --ignore-missing
> lnd-linux-armv7-v0.5.2-beta.tar.gz：好的

$ gpg ./pgp_keys.asc
> BD599672C804AF2770869A048B80CD2BB8BD8132

$ gpg --import ./pgp_keys.asc
$ gpg --verify manifest-v0.5.2-beta.txt.sig
> gpg：来自“Olaoluwa Osuntokun <laolu32@gmail.com>”的好签名[未知]
>主键指纹：BD59 9672 C804 AF27 7086 9A04 8B80 CD2B B8BD 8132
>子键指纹：F803 7E70 C12C 7A26 3C03 2508 CE58 F7F8 E20F D9A2

$ tar -xzf lnd-linux-armv7-v0.5.2-beta.tar.gz
$ sudo install -m 0755 -o root -g root -t / usr / local / bin lnd-linux-armv7-v0.5.2-beta / *
$ lnd --version
> lnd版本0.5.2-beta commit = v0.5.2-beta-rc7
```
！[Checksum LND]（images / 40_checksum_lnd.png）

### LND配置
现在已经安装了LND，我们需要将其配置为使用比特币核心并在启动时自动运行。

* 仍然作为用户'admin'，创建一个指向`/ bin / ip`中的`ip`二进制文件的符号链接，因为在某些情况下LND似乎无法找到它。
  `$ sudo ln -s / bin / ip / usr / bin / ip`

* 打开“比特币”用户会话
  `$ sudo su  - 比特币`

* 创建LND工作目录和相应的符号链接
  `$ mkdir / mnt / hdd / lnd`
  `$ ln -s / mnt / hdd / lnd / home / bitcoin / .lnd`
  `$ ls -la`

！[检查符号链接LND]（images / 40_symlink_lnd.png）

* 创建LND配置文件并粘贴以下内容（调整您的别名）。保存并退出。
  `$ nano / home / bitcoin / .lnd / lnd.conf`

```庆典
#RaspiBolt：lnd配置
#home/bitcoin/.lnd/lnd.conf

[应用选项]
DEBUGLEVEL =信息
maxpendingchannels = 5
别名= YOUR_NAME [LND]
颜色=＃68F442

＃您的路由器必须支持并启用UPnP，否则删除此行
NAT =真

[比特币]
bitcoin.active = 1

＃启用testnet或mainnet
bitcoin.testnet = 1
＃bitcoin.mainnet = 1

bitcoin.node = bitcoind

[自动驾驶仪]
autopilot.active = 1
autopilot.maxchannels = 5
autopilot.allocation = 0.6
```
关于此配置的一些解释：

* 配置选项** nat = true **期望您的互联网路由器支持Universal Plug'n'Play（UPnP）并启用它。这允许LND通过设置端口转发使您的节点从网络外部到达，宣布您的外部IP地址并在您的IP地址更改时更新此信息。这是目前唯一具有路由Lightning节点的可靠配置。

  **如果您的路由器不支持UPnP **，LND仍然可以使用，但您的节点将成为您自己付款的私人Lightning节点，并且无法为其他人路由付款。在这种情况下，您需要删除上面配置文件中的`nat = true`行，否则LND将无法启动。另一个选择是在LND启动时传递您的IP地址（参见[external guide]（https://github.com/robclark56/RaspiBolt-Extras/blob/master/RB_extra_02.md））。

* 上面的配置启用了**自动驾驶仪**，并且会自动打开** 5 **频道，资金为** 60％**。您可能需要根据自己的需要进行调整。使用`autopilot.active = 0`，您可以完全禁用自动驾驶仪并手动管理频道。

：point_right：其他信息：LND项目存储库中的[sample-lnd.conf]（https://github.com/lightningnetwork/lnd/blob/master/sample-lnd.conf）

### 运行LND

再次，我们切换到用户“比特币”并首先手动启动程序以检查一切是否正常。

```
$ sudo su  - 比特币
$ lnd
```

守护程序将状态信息直接打印到命令行。这意味着我们不能在不停止服务器的情况下使用该会话。我们需要打开第二个SSH会话。

### LND钱包设置

再次启动SSH程序（例如PuTTY），连接到Pi并以“admin”身份登录。 **第二个会话**的命令以提示符“$ 2”开头（不得输入）。

一旦LND启动，该过程等待我们创建集成的比特币钱包（它不使用“比特币”钱包）。

* 启动“比特币”用户会话
  `$ 2 sudo su  - 比特币`

* 创建LND钱包

  `$ 2 lncli --network = testnet create`

* 如果你想创建一个新的钱包，输入你的`密码[C]`作为钱包密码，选择关于现有种子的`n`并输入可选的`密码[D]`作为种子密码短语。创建由24个单词组成的新密码种子。

！[LND新密码种子]（images / 40_cipher_seed.png）

这24个单词与您的密码（可选`密码[D]`）相结合，是您恢复比特币钱包和所有照明频道所需的全部内容。但是，无法从此种子重新创建通道的当前状态，这需要连续备份，并且仍在开发LND。

：警告：此信息必须始终保密。 **将这24个单词手动写在一张纸上并将其存放在安全的地方。**这张纸是攻击者需要完全清空你的钱包！不要将其存放在计算机上。不要用手机拍照。 **此信息绝不应以数字形式存储在任何地方。**

* 退出“比特币”用户会话
  `$ 2退出`

让我们使用命令行界面`lncli`授权“admin”用户使用LND。为此，我们需要将传输层安全性（TLS）证书和权限文件（macaroons）复制到admin主文件夹。

* 检查是否已创建TLS证书
  `$ 2 sudo ls -la / home / bitcoin / .lnd /`

* 检查是否已创建权限文件`admin.macaroon`和`readonly.macaroon`。
  `$ 2 sudo ls -la / home / bitcoin / .lnd / data / chain / bitcoin / testnet /`

！[检查蛋白杏仁饼干]（images / 40_ls_macaroon.png）

* 将权限文件和TLS证书复制到用户“admin”
  `$ 2 cd / home / bitcoin /`
  `$ 2 sudo cp --parents .lnd / data / chain / bitcoin / testnet / admin.macaroon / home / admin /`
  `$ 2 sudo cp /home/bitcoin/.lnd/tls.cert / home / admin / .lnd`
  `$ 2 sudo chown -R admin：admin / home / admin / .lnd /`
* 确保`lncli`通过解锁你的钱包（输入`password [C]`）并获得一些节点信息来工作。
  `$ 2 lncli --network = testnet unlock`
* 检查LND的当前状态
  `$ 2 lncli --network = testnet getinfo`

您还可以在第一个SSH会话中查看LND与比特币初始同步的进度。

检查以下两行以确保使用UPnP成功设置端口转发。如果LND无法配置您的路由器（例如，可能不支持UPnP），您的节点仍然可以工作，但它无法为其他网络参与者路由器事务。

```
[INF] SRVR：扫描本地网络以获取启用UPnP的设备
[INF] SRVR：使用UPnP自动设置端口转发以通告外部IP
```

让我们暂时停止服务器并再次关注我们的主要SSH会话。

* `$ 2 lncli --network = testnet stop`
* `$ 2退出`

这应该在SSH会话1中“优雅地”终止LND，现在可以再次以交互方式使用。

### 自动启动LND

现在，让我们设置LND以在系统启动时自动启动。

* 退出“比特币”用户会话回“管理员”
  `$ exit`
* 创建LND systemd单元并具有以下内容。保存并退出。
  `$ sudo nano / etc / systemd / system / lnd.service`

```庆典
#RaspiBolt：lnd的systemd单元
＃/ etc / system / system / lnd.service

[单元]
描述= LND闪电守护进程
想要= bitcoind.service
之后= bitcoind.service

[服务]
ExecStart =在/ usr / local / bin目录/ LND

用户=比特币
组=比特币
类型=简单
KillMode =过程
LimitNOFILE = 128000
TimeoutSec = 240
重启=始终
RestartSec = 60

[安装]
WantedBy = multi-user.target
```

* 启用，启动和解锁LND
  `$ sudo systemctl enable lnd`
  `$ sudo systemctl start lnd`
  `$ systemctl status lnd`
  `$ lncli --network = testnet unlock`

* 现在，守护程序信息不再显示在命令行上，而是写入系统日志。您可以监视LND启动进度，直到它赶上testnet区块链（此刻大约1.3m块）。这可能需要长达2个小时，之后您会看到很多非常快速的聊天（以“Ctrl-C”退出）。
  `$ sudo journalctl -f -u lnd`

！[LND启动日志]（C：/Users_withBackup/Roland/Documents/GitHub/guides/raspibolt/images/40_start_lnd.png）



### 获取一些testnet比特币

现在您的Lightning节点已准备就绪。要在testnet中使用它，您可以从水龙头获得一些免费的testnet比特币。
* 生成新的比特币地址以接收链上资金
  `$ lncli --network = testnet newaddress np2wkh`
  `>“address”：“2NCoq9q7 ............ dkuca5LzPXnJ9NQ”`

* 获取testnet比特币：
  https://testnet-faucet.mempool.co

* 检查您的LND钱包余额
  `$ lncli --network = testnet walletbalance`

* 在区块链资源管理器上监控您的交易（水龙头显示TX ID）：
  https://testnet.smartbit.com.au

### LND在行动
一旦您的资金交易被开采和确认，LND将开始开放和维护渠道。此功能称为“自动驾驶”，并在“lnd.conf”文件中配置。如果您想手动维护频道，可以禁用自动驾驶仪。

在[StarBlocks]（https://starblocks.acinq.co/#/）或[Y'alls]（https://testnet.yalls.org/）上获取付款请求并移动一些硬币！

* `$ lncli --network = testnet listpeers`
* `$ lncli --network = testnet listchannels`
* `$ lncli --network = testnet sendpayment --pay_req = lntb32u1pdg7p ... y0gtw6qtq0gcpk50kww`
* `$ lncli --network = testnet listpayments`

：point_right：有关其他信息，请参阅[Lightning API参考]（http://api.lightning.community/）

### （可选）添加别名以获得更简单的命令
如果您不想每次都输入完整的命令，别名将有所帮助。有关别名设置，请参见[Additional scripts]（raspibolt_67_additional-scripts.md）部分。

-----

### LND升级
如果您希望将来升级到LND的新版本，请查看常见问题解答部分：
[如何升级LND]（https://github.com/Stadicus/guides/blob/master/raspibolt/raspibolt_faq.md#how-to-upgrade-lnd-bin-）

-----

### 在进入mainnet之前
这是不归路。到目前为止，你可以重新开始。用testnet比特币进行实验。在testnet上打开和关闭通道。

一旦你切换到主网并将真正的比特币发送到你的RaspiBolt，你就有了“游戏中的皮肤”。

* 确保您的RaspiBolt按预期工作。
* 使用`bitcoin-cli`及其选项进行一些练习（参见[比特币核心RPC文档]（https://bitcoin-rpc.github.io/））
* 使用`lncli`进行干运行及其众多选项（参见[Lightning API参考]（http://api.lightning.community/））
* 尝试几次重启（`sudo shutdown -r now`），一切都很好吗？

---
下一页：[Mainnet >>]（raspibolt_50_mainnet.md）
