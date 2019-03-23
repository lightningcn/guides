[[简介]（README.md）]  -  [[筹备工作]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [**比特币**]  -  [[闪电] （raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[Bonus]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

-------
### Raspberry Pi上的️⚡Lightning️⚡初学者指南
--------

# 比特币
Lightning节点的基础是一个完全不信任的[比特币核心]（https://bitcoin.org/en/bitcoin-core/）节点。它保留区块链的完整副本并验证所有事务和块。通过自己完成所有这些工作，没有其他人需要被信任。

一开始，我们将使用比特币测试网络来熟悉其操作。此同步由Pi直接处理，不应超过几个小时。让它在一夜之间同步。

### 安装
我们将直接从bitcoin.org下载该软件，验证其签名以确保我们使用正式版本并安装它。

* 以“admin”身份登录并创建下载文件夹
  `$ mkdir / home / admin / download`
  `$ cd / home / admin / download`
* 如果您升级并先前已下载文件，请先删除它们
  `$ rm *`

我们下载最新的比特币核心二进制文件（应用程序）并将文件与签名的校验和进行比较。这是一个预防措施，以确保这是一个正式版本，而不是试图窃取我们的钱的恶意版本。

* 获取[bitcoincore.org/en/download](https://bitcoincore.org/en/download）（ARM Linux 32位）的最新下载链接，它们随每次更新而变化。然后运行以下命令（带有调整后的文件名）并检查输出位置：

```庆典
＃下载比特币核心二进制文件
$ wget https://bitcoincore.org/bin/bitcoin-core-0.17.1/bitcoin-0.17.1-arm-linux-gnueabihf.tar.gz
$ wget https://bitcoincore.org/bin/bitcoin-core-0.17.1/SHA256SUMS.asc
$ wget https://bitcoin.org/laanwj-releases.asc

＃检查引用校验和是否与实际校验和匹配
＃（忽略“行格式不正确”警告）
$ sha256sum --check SHA256SUMS.asc --ignore-missing
> bitcoin-0.17.1-arm-linux-gnueabihf.tar.gz：好的

＃导入Wladimir van der Laan的公钥，验证签名的校验和文件
＃并在恶意密钥的情况下再次检查指纹
$ gpg --import ./laanwj-releases.asc
$ gpg  - 验证SHA256SUMS.asc
> gpg：来自“Wladimir J. van der Laan ......”的好签名
>主键指纹：01EA 5486 DE18 A882 D4C2 6845 90C8 019E 36C2 E964
```

！[检查bitcoind签名的命令]（images / 30_checksum.png）

* 现在我们知道bitcoin.org的密钥是有效的，所以我们也可以验证Windows二进制校验和。将以下输出与Windows比特币核心下载的校验和进行比较。

```庆典
$ cat /home/admin/download/SHA256SUMS.asc | grep win

e9245e682126ef9fa4998eabbbdd1c3959df811dc10df60be626a5e5ffba9b78 bitcoin-0.17.1-win32-setup.exe
6464aa2d338f3697950613bb88124e58d6ce78ead5e9ecacb5ba79d1e86a4e30 bitcoin-0.17.1-win32.zip
fa1e80c5e4ecc705549a8061e5e7e0aa6b2d26967f99681b5989d9bd938d8467 bitcoin-0.17.1-win64-setup.exe
1abbe6aa170ce7d8263d262f8cb0ae2a5bb3993aacd2f0c7e5316ae595fe81d7 bitcoin-0.17.1-win64.zip
```

* 提取比特币核心二进制文件，安装它们并检查版本。

```庆典
$ tar -xvf bitcoin-0.17.1-arm-linux-gnueabihf.tar.gz
$ sudo install -m 0755 -o root -g root -t / usr / local / bin bitcoin-0.17.1 / bin / *
$ bitcoind --version
>比特币核心守护进程版本v0.17.1
```

### 准备比特币核心目录
我们使用名为“bitcoind”的比特币守护进程，它在没有用户界面的后台运行，并将所有数据存储在目录`/ home / bitcoin / .bitcoin`中。我们创建一个指向外部硬盘上的目录的链接，而不是创建一个真实的目录。

* 使用用户“admin”登录后，更改为用户“比特币”
  `$ sudo su  - 比特币`

* 我们添加一个指向外部硬盘的符号链接。
  `$ ln -s / mnt / hdd / bitcoin / home / bitcoin / .bitcoin`

* 导航到主目录并检查符号链接（目标不能为红色）。该目录的内容实际上位于外部硬盘上。
  `$ ls -la`

！[验证.bitcoin符号链接]（images / 30_show_symlink.png）

### 组态
现在，需要创建bitcoind的配置文件。用Nano打开它并粘贴下面的配置。保存并退出。
`$ nano / home / bitcoin / .bitcoin / bitcoin.conf`

```庆典
#RaspiBolt：bitcoind配置
#home/bitcoin/.bitcoin/bitcoin.conf

＃删除以下行以启用比特币主网
testnet = 1

#Bitcoind选项
服务器= 1
守护进程= 1

＃连接设置
rpcuser = raspibolt
rpcpassword = PASSWORD_ [B]

onlynet = IPv4的
zmqpubrawblock = TCP：//127.0.0.1：28332
zmqpubrawtx = TCP：//127.0.0.1：28333

#Raspberry Pi优化
dbcache = 100
maxorphantx = 10
maxmempool = 50
MaxConnections最大= 40
maxuploadtarget = 5000
```
：警告：将rpcpassword更改为您的安全“密码[B]`，否则您的资金可能会被盗。

：point_right：其他信息：[配置选项]（https://en.bitcoin.it/wiki/Running_Bitcoin#Command-line_arguments）在比特币维基

### 开始比特币

仍以用户“比特币”身份登录，让我们手动启动“bitcoind”。监视日志文件几分钟，看看它是否正常工作（它可能停在“dnsseed线程退出”，没关系）。使用`Ctrl-C`退出日志文件监视，检查区块链信息，如果没有错误，再次停止“bitcoind”。

```
$ bitcoind
$ tail -f /home/bitcoin/.bitcoin/testnet3/debug.log
$ bitcoin-cli getblockchaininfo
$ bitcoin-cli停止
```

### 自动启动bitcoind

系统需要在后台自动运行比特币守护进程，即使没有人登录。我们使用“systemd”，一个使用配置文件控制启动过程的守护进程。

* 退出“比特币”用户会话回到用户“admin”
  `$ exit`

* 在Nano文本编辑器中创建配置文件并复制以下段落。
  `$ sudo nano / etc / systemd / system / bitcoind.service`

```庆典
#RaspiBolt：bitcoind的systemd单元
＃/ etc / system / system / bitcoind.service

[单元]
描述=比特币守护进程
之后= network.target

[服务]
ExecStartPre = / bin / sh -c'睡30'
ExecStart = / usr / local / bin / bitcoind -daemon -conf = / home / bitcoin / .bitcoin / bitcoin.conf -pid = / home / bitcoin / .bitcoin / bitcoind.pid
PIDFILE = /家庭/比特币/ .bitcoin / bitcoind.pid
用户=比特币
组=比特币
类型=分叉
KillMode =过程
重启=始终
TimeoutSec = 120
RestartSec = 30

[安装]
WantedBy = multi-user.target
```
* 保存并退出
* 启用配置文件
  `$ sudo systemctl enable bitcoind.service`
* 将`bitcoin.conf`复制到用户“admin”主目录以获取RPC凭据
  `$ mkdir / home / admin / .bitcoin`
  `$ sudo cp /home/bitcoin/.bitcoin/bitcoin.conf / home / admin / .bitcoin /`
* 重启Raspberry Pi
  `$ sudo shutdown -r now`

### 验证比特币操作
重新启动后，bitcoind应该启动并开始同步并验证比特币区块链。

* 稍等一下，通过SSH重新连接并使用用户“admin”登录。

* 检查systemd启动的比特币守护进程的状态（使用`Ctrl-C`退出）

  `$ systemctl status bitcoind.service`


！[比特币状态]（images / 30_status_bitcoind.png）

* 通过监视其日志文件来查看bitcoind（使用`Ctrl-C`退出）
  `$ sudo tail -f / home / bitcoin / .bitcoin / testnet3 / debug.log`

* 使用比特币核心客户端`bitcoin-cli`获取有关当前区块链的信息
  `$ bitcoin-cli getblockchaininfo`

* 请注意：
  * 当“bitcoind”仍在启动时，您可能会收到类似“验证块”的错误消息。这是正常的，只需几分钟。
  * 在其他信息中，显示了“验证进展”。一旦该值达到差不多1（0.999 ...），区块链就是最新的并且经过充分验证。

### 探索比特币cli
如果一切顺利进行，这是熟悉比特币核心并玩“比特币 -  cli”的最佳时机，直到区块链是最新的。

* 一个很好的开始是由Andreas Antonopoulos编写的掌握比特币**的书 - 这是开源的 - 在这方面尤其是第3章（忽略第一部分如何从源代码编译）：
  * 你肯定需要这本书的[真实副本]（https://bitcoinbook.info/）！
  * 在[Github]上在线阅读（https://github.com/bitcoinbook/bitcoinbook）

！[掌握比特币]（images / 30_mastering_bitcoin_book.jpg）

* 要深入了解，请查看[**从命令行学习比特币**]（https://github.com/ChristopherA/Learning-Bitcoin-from-the-Command-Line/blob/master/README.md克里斯托弗艾伦。



👉其他信息：[bitcoin-cli reference]（https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_calls_list）

一旦区块链在testnet上同步，就可以设置Lightning节点。

-----

### 比特币核心升级
如果您希望将来升级到新版比特币核心，请查看常见问题解答部分：
[如何升级比特币核心]（https://github.com/Stadicus/guides/blob/master/raspibolt/raspibolt_faq.md#how-to-upgrade-bitcoin-core）

-----
下一个：[闪电>>]（raspibolt_40_lnd.md）
