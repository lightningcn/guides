[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [Lightning]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[Bonus]（raspibolt_60_bonus.md）]  -  [**疑难解答**]

-------
### Raspberry Pi上的️⚡Lightning️⚡初学者指南
--------

# 故障排除

此附加故障排除指南的目的是帮助您调试系统，验证所有重要配置并找到frikkin'小问题，使您无法运行完美的Lightning节点。

在可能的情况下，我将链接到指南的相关部分。如果您发现预期输出存在任何差异，请仔细检查您在该特定区域中设置节点的方式。



### [一般常见问题]（raspibolt_faq.md）

我在单独的[常见问题解答]（raspibolt_faq.md）部分收集了与问题没有直接关系的常见问题：

* 我可以通过路由Lightning付款致富吗？
* 我可以将Ext4格式的硬盘连接到我的Windows计算机吗？
* 所有Linux命令都做了什么？
* 我在哪里可以获得更多信息？
* 如何升级比特币核心或LND？
* 当我使用带有64位处理器的Raspberry Pi 3时，为什么需要32位版本的比特币？



### 硬件和操作系统

#### 你能用SSH登录吗？

如果您以某种方式将自己锁定在Pi之外，您可以将其连接到显示器和键盘以直接登录而无需任何证书。

#### 你有兼容的Raspberry Pi吗？

```
$ cat / proc / device-tree / model
Raspberry Pi 3 Model B Rev 1.2

$ uname -a
Linux RaspiBolt 4.14.71-v7 +＃1145 SMP Fri Sep 21 15:38:35 BST 2018 armv7l GNU / Linux
```

重要的是，您的Raspberry Pi使用** armv7 ** CPU架构。

#### 你的根文件系统是只读的吗？

如果您收到“无法.....：只读文件系统”这样的错误，则指向错误的SD卡。如果linux检测到损坏的根文件系统，它将进入只读模式。尝试再次刷新SD卡，或使用其他卡。

#### 硬盘是否安装？

检查硬盘是否已安装到`/ mnt / hdd`并且大小是否正确。其他线路可能因您的Pi而异。

```
admin @ RaspiBolt：〜$ df -h
使用的文件系统大小可用使用％挂载
/ dev / root 15G 2.9G 12G 21％/
devtmpfs 484M 0 484M 0％/ dev
tmpfs 489M 0 489M 0％/ dev / shm
tmpfs 489M 50M 440M 11％/运行
tmpfs 5.0M 4.0K 5.0M 1％/运行/锁定
tmpfs 489M 0 489M 0％/ sys / fs / cgroup
/ dev / mmcblk0p1 43M 22M 21M 52％/ boot
/ dev / sda1 458G 232G 203G 54％/ mnt / hdd
tmpfs 98M 0 98M 0％/ run / user / 1001
```

#### 硬盘故障导致的错误，例如复制区块链时？

要检查硬盘的文件系统，您需要卸载它，并且因为我们在硬盘上也有交换文件，所以在没有`bitcoind` /`lnd` autostart的情况下重启。

禁用服务并在`/ etc / fstab`中的mount条目前面添加一个`#`，保存并退出并重新启动。

```
$ sudo systemctl禁用bitcoind
$ sudo systemctl disable lnd
$ sudo nano / etc / fstab
#UUID = [your_hdd_UUID] / mnt / hdd ext4 noexec，默认值0 0
$ sudo shutdown -r now
```

现在，让我们检查一下hdd是否真的没有安装（没有`/ mnt / hdd`的条目），获取设备的名称（例如``sda1`）并检查文件系统。

```
$ df -h
$ lsblk -o UUID，NAME，FSTYPE，SIZE，LABEL，MODEL
$ sudo fsck / dev / sda1 -f
```

尝试修复可能检测到的任何错误。如果硬盘出现物理故障，则可能需要更换硬盘。

要重新启动并运行，请反转第一步：

```
$ sudo systemctl启用bitcoind
$ sudo systemctl enable lnd
$ sudo nano / etc / fstab
UUID = [your_hdd_UUID] / mnt / hdd ext4 noexec，默认值0 0
$ sudo shutdown -r now
```

#### ip端口是否可通过防火墙访问？

最重要的端口是22,8333,9735和1900 / udp。其他可能是奖金指南所必需的，你的Pi上可能还有其他端口打开（例如`（v6）`变种）。

拥有正确的子网掩码至关重要，例如`192.168.0.0 / 24`（参见[guide]（raspibolt_20_pi.md #enable-the-uncomplicated-firewall））。

```
$ sudo ufw status
状态：有效

来自行动
 -  ------ ----
22允许192.168.0.0/24＃允许来自本地LAN的SSH
50002允许192.168.0.0/24＃允许来自本地LAN的电子
9735 ALLOW Anywhere＃允许闪电
8333 ALLOW Anywhere＃允许比特币主网
18333 ALLOW Anywhere＃允许比特币测试网
Anywhere ALLOW 192.168.0.0/24 1900 / udp＃允许本地LAN SSDP用于UPnP发现
10009允许192.168.0.0/24＃允许来自本地LAN的LND grpc
```

### 用户和目录

#### 是否已创建所有用户和主目录？

```
$ cat / etc / passwd
。
。
管理员：X：1001：1001：,,,：/首页/管理：/斌/庆典
比特币：X：1002：1002：,,,：/首页/比特币：/斌/庆典
```

#### 应用程序目录是否已正确创建和链接？

以下符号链接位于“比特币”用户的主目录中非常重要。应显示浅蓝色和深蓝色，并且不得为红色（表示链接断开）。

用户“比特币”必须具有写访问权限（在下面的“lr ** w ** xwwxrwx”中以第3个字符“w”表示）。用`touch`检查。

```
$ sudo su  - 比特币
$ touch / home / bitcoin / test
$ ls -la / home / bitcoin /
。
lrwxrwxrwx 1 bitcoin bitcoin 16 Nov 13 21:47 .bitcoin  - > / mnt / hdd / bitcoin
lrwxrwxrwx 1比特币比特币12月13日22:28 .lnd  - > / mnt / hdd / lnd
-rw-r  -  r-- 1比特币比特币0月27日18:36测试
。
```

并非所有比特币文件都必须存在，但比特币必须具有当前目录的写访问权限（使用`touch`检查）。

```
$ touch /home/bitcoin/.bitcoin/test
$ ls -la /home/bitcoin/.bitcoin/
总共7232
drwxr-xr-x 6比特币比特币4096年11月27日16:11。
drwxr-xr-x 5比特币比特币4096年11月13日22:28 ..
-rw ------- 1 bitcoin bitcoin 145 Nov 27 15:56 banlist.dat
-rw-r  -  r-- 1 bitcoin bitcoin 467 Nov 25 09:07 bitcoin.conf
-rw ------- 1比特币比特币5月25日09:18 bitcoind.pid
drwxr-xr-x 3比特币比特币73728 11月27日10:58块
drwxr-xr-x 2比特币比特币69632 11月27日16:12链状态
drwx ------ 2比特币比特币4096年11月25日09:22数据库
-rw ------- 1比特币比特币0月14日18:50 db.log
-rw ------- 1 bitcoin bitcoin 1038648 11月27日16:16 debug.log
-rw ------- 1 bitcoin bitcoin 247985 11月25日09:18 fee_estimates.dat
-rw ------- 1 bitcoin bitcoin 0 Nov 14 18:50 .lock
-rw ------- 1 bitcoin bitcoin 353600 11月25日09:18 mempool.dat
-rw ------- 1 bitcoin bitcoin 4170378 11月27日16:11 peers.dat
-rw-r  -  r-- 1比特币比特币0月27日18:24测试
drwxr-xr-x 5比特币比特币4096年11月25日05:31 testnet3
-rw ------- 1 bitcoin bitcoin 1409024 11月27日09:19 wallet.dat
-rw ------- 1比特币比特币0月14日18:50 .walletlock
```

如果LND至少启动一次，则应存在以下文件和目录。

```
$ touch /home/bitcoin/.lnd/test
$ ls -la /home/bitcoin/.lnd/
共28
drwxr-xr-x 4比特币比特币4096年11月16日16:28。
drwxr-xr-x 5比特币比特币4096年11月13日22:28 ..
drwx ------ 4比特币比特币4096年11月13日22:37数据
-rw-r  -  r-- 1 bitcoin bitcoin 425 Nov 16 16:28 lnd.conf
drwx ------ 3比特币比特币4096年11月13日22:30日志
-rw-r  -  r-- 1比特币比特币0月27日18:37测试
-rw-r  -  r-- 1比特币比特币733年11月13日22:30 tls.cert
-rw ------- 1比特币比特币227 Nov 13 22:30 tls.key
```

不要忘记退出“比特币”用户会话：

```
$退出
```



### 比特币核心

首先，让我们禁用systemd自动启动并重新启动Pi以手动运行所有内容。

```
$ sudo systemctl禁用bitcoind
$ sudo systemctl disable lnd
$ sudo shutdown -r now
```

如果基本设置似乎正常，让我们打开一个“比特币”用户会话，检查比特币核心配置，启动程序并检查输出。

```
$ sudo su  - 比特币

$ cat /home/bitcoin/.bitcoin/bitcoin.conf
#RaspiBolt LND Mainnet：bitcoind配置
#home/bitcoin/.bitcoin/bitcoin.conf

＃删除以下行以启用比特币主网
testnet = 1

#Bitcoind选项
服务器= 1
守护进程= 1

＃连接设置
rpcuser = raspibolt
rpcpassword = PASSWORD_B

onlynet = IPv4的
zmqpubrawblock = TCP：//127.0.0.1：28332
zmqpubrawtx = TCP：//127.0.0.1：28333

#Raspberry Pi优化
dbcache = 100
maxorphantx = 10
maxmempool = 50
MaxConnections最大= 40
maxuploadtarget = 5000

$ bitcoind --version
比特币核心守护程序版本v0.17.0.1

$ bitcoind
比特币服务器启动

$ tail -f /home/bitcoin/.bitcoin/debug.log
2018-11-25T19：31：57Z比特币核心版本v0.17.0.1（发布版本）
2018-11-25T19：31：57Z InitParameterInteraction：参数交互：-whitelistforcerelay = 1  - > setting -whitelistrelay = 1
2018-11-25T19：31：57Z假设块0000000000000000002e63058c023a9a1de233554f28c7b21380b6c9003f36a8的祖先具有有效签名。
2018-11-25T19：31：57Z设置nMinimumChainWork = 0000000000000000000000000000000000000000028822fef1c230963535a90d
2018-11-25T19：31：57Z使用'标准'SHA256实现
2018-11-25T19：31：57Z默认数据目录/home/bitcoin/.bitcoin
2018-11-25T19：31：57Z使用数据目录/home/bitcoin/.bitcoin
2018-11-25T19：31：57Z使用配置文件/home/bitcoin/.bitcoin/bitcoin.conf
2018-11-25T19：31：57Z最多使用40个自动连接（128000个文件描述符可用）
2018-11-25T19：31：57Z在32/2中使用16 MiB请求签名缓存，能够存储524288个元素
2018-11-25T19：31：57Z在32/2中使用16 MiB请求脚本执行缓存，能够存储524288个元素
2018-11-25T19：31：57Z使用4个线程进行脚本验证
2018-11-25T19：31：57Z调度程序线程启动
2018-11-25T19：31：57Z HTTP：创建深度为16的工作队列
2018-11-25T19：31：57Z配置选项rpcuser和rpcpassword很快就会被弃用。本地运行的实例可以删除rpcuser以使用基于cookie的身份验证，或者可以用rpcauth替换。请参阅share / rpcauth了解rpcauth auth生成。
2018-11-25T19：31：57Z HTTP：启动4个工作线程
2018-11-25T19：31：57Z使用钱包目录/home/bitcoin/.bitcoin
2018-11-25T19：31：57Z初始消息：验证钱包......
2018-11-25T19：31：57Z使用BerkeleyDB版本Berkeley DB 4.8.30 :( 2010年4月9日）
2018-11-25T19：31：57Z使用钱包wallet.dat
2018-11-25T19：31：57Z BerkeleyEnvironment :: Open：LogDir = / home / bitcoin / .bitcoin / database ErrorFile = / home / bitcoin / .bitcoin / db.log
2018-11-25T19：31：57Z缓存配置：
2018-11-25T19：31：57Z *使用2.0MiB作为块索引数据库
2018-11-25T19：31：57Z *使用8.0MiB进行链状态数据库
2018-11-25T19：31：57Z *使用90.0MiB进行内存中的UTXO设置（加上最多47.7MiB的未使用的mempool空间）
2018-11-25T19：31：57Z init消息：加载块索引...
2018-11-25T19：31：57Z在/home/bitcoin/.bitcoin/blocks/index中打开LevelDB
2018-11-25T19：31：58Z成功打开LevelDB
2018-11-25T19：31：58Z使用/home/bitcoin/.bitcoin/blocks/index的混淆密钥：0000000000000000
2018-11-25T19：32：21Z LoadBlockIndexDB：最后一个块文件= 1445
2018-11-25T19：32：21Z LoadBlockIndexDB：最后一个块文件信息：CBlockFileInfo（blocks = 34，size = 38568339，high = 551687 ... 551720，time = 2018-11-25 ... 2018-11-25）
2018-11-25T19：32：21Z检查所有blk文件是否存在...
2018-11-25T19：32：22Z在/home/bitcoin/.bitcoin/chainstate中打开LevelDB
2018-11-25T19：32：26Z成功打开LevelDB
2018-11-25T19：32：26Z使用/ home / bitcoin / .bitcoin / chainstate的混淆密钥：379f438868caeb46
2018-11-25T19：32：27Z加载最佳链：hashBestChain = 00000000000000000019f4d6b0a3ac29f65789035e88ca279d2820a33405e056 height = 551720 date = 2018-11-25T19：22：18Z progress = 0.999996
2018-11-25T19：32：27Z初始消息：重绕块...
2018-11-25T19：32：34Z初始化消息：验证块...
2018-11-25T19：32：34Z在3级验证最后6个区块
2018-11-25T19：32：34Z [0％] ... [16％] ... [33％] ... [50％] ... [66％] ... [83％] .. [99％] ... [DONE]。
2018-11-25T19：32：57Z最后6个街区没有硬币数据库不一致（10080笔交易）
2018-11-25T19：32：57Z块指数60089ms
2018-11-25T19：32：57Z初始化消息：正在加载钱包......
2018-11-25T19：32：58Z [默认钱包] nFileVersion = 170001
2018-11-25T19：32：58Z [默认钱包]密钥：2001明文，0加密，2001 w /元数据，2001总计。未知的钱包记录：1
2018-11-25T19：32：58Z [默认钱包]钱包在432ms完成装载
2018-11-25T19：32：58Z [默认钱包] setKeyPool.size（）= 2000
2018-11-25T19：32：58Z [默认钱包] mapWallet.size（）= 0
2018-11-25T19：32：58Z [默认钱包] mapAddressBook.size（）= 0
2018-11-25T19：32：58Z mapBlockIndex.size（）= 551721
2018-11-25T19：32：58Z nBestHeight = 551720
2018-11-25T19：32：58Z绑定到[::]：8333
2018-11-25T19：32：58Z绑定到0.0.0.0:8333
2018-11-25T19：32：58Z init消息：加载P2P地址......
2018-11-25T19：32：58Z torcontrol螺纹启动
2018-11-25T19：32：58Z离开InitialBlockDownload（锁定为false）
2018-11-25T19：33：00Z从peers.dat 1856ms加载63097地址
2018-11-25T19：33：00Z初始消息：加载banlist ...
2018-11-25T19：33：00Z init消息：启动网络线程...
2018-11-25T19：33：00Z净线程启动
2018-11-25T19：33：00Z addcon线程启动
2018-11-25T19：33：00Z初始化消息：完成加载
2018-11-25T19：33：00Z msghand线程启动
2018-11-25T19：33：00Z dnsseed线程启动
2018-11-25T19：33：00Z opencon线程启动
2018-11-25T19：33：02Z新出站对等连接：版本：70015，块= 551720，对等= 1
2018-11-25T19：33：04Z套接字recv错误通过对等方重置连接（104）
2018-11-25T19：33：11Z从DNS种子加载地址（可能需要一段时间）
2018-11-25T19：33：14Z新出站对等连接：版本：70015，块= 551720，对等= 7
2018-11-25T19：33：14Z新出站对等连接：版本：70015，块= 551720，对等= 9
2018-11-25T19：33：15Z新出站对等连接：版本：70015，块= 551720，对等= 10
2018-11-25T19：33：26Z从DNS种子发现的255个地址
2018-11-25T19：33：26Z dnsseed螺纹出口
2018-11-25T19：33：28Z新出站对等连接：版本：70015，块= 551720，对等= 14
2018-11-25T19：33：29Z新出站对等连接：版本：70015，块= 551720，对等= 15
...
...

$ bitcoin-cli getblockchaininfo
$ bitcoin-cli停止
$退出
```



恢复正常操作：再次启用和启动服务。

```
$ sudo systemctl启用bitcoind
$ sudo systemctl start bitcoind
$ sudo systemctl enable lnd
$ sudo systemctl start lnd
```



### LND

我们来检查LND的配置和操作。

```
$ sudo systemctl stop lnd
$ sudo su  - 比特币

$ cat /home/bitcoin/.lnd/lnd.conf
#RaspiBolt：lnd配置
#home/bitcoin/.lnd/lnd.conf

[应用选项]
DEBUGLEVEL =信息
maxpendingchannels = 5
别名= YOUR_NAME [LND]
颜色=＃68F442
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

$ lnd
尝试自动RPC配置到bitcoind
自动获取bitcoind的RPC凭据
2018-11-25 19：40：03.072 [INF] LTND：版本0.5.0-beta commit = 3b2c807288b1b7f40d609533c1e96a510ac5fa6d
2018-11-25 19：40：03.072 [INF] LTND：主动链：比特币（网络=主网）
2018-11-25 19：40：03.073 [INF] CHDB：检查架构更新：latest_version = 6，db_version = 6
2018-11-25 19：40：03.107 [INF] RPCS：密码RPC服务器监听127.0.0.1:10009
2018-11-25 19：40：03.107 [INF] RPCS：密码gRPC代理起始于127.0.0.1:8080
2018-11-25 19：40：03.107 [INF] LTND：等待钱包加密密码。使用`lncli create`创建钱包，`lncli unlock`解锁现有钱包，或使用`lncli changepassword`更改现有钱包的密码并解锁。
```

由于此“比特币”用户会话现在由LND占用，请在您的节点上打开第二个SSH会话（此处显示前缀为“$ 2”）并解锁您的钱包。

```
$ 2 sudo su  - 比特币
$ 2 lncli解锁
```

回到第一个SSH会话，钱包显示为已解锁且LND开始连接到网络（请参阅下面的示例输出）。此处将显示任何潜在错误。

```-----
2018-11-25 19：42：57.143 [INF] LNWL：打开钱包
2018-11-25 19：42：57.689 [INF] LTND：主链设置为：比特币
2018-11-25 19：42：57.785 [INF] LNWL：在tcp：//127.0.0.1：28332上开始通过ZMQ监听bitcoind块通知
2018-11-25 19：42：57.785 [INF] LTND：初始化比特币支持费用估算器
2018-11-25 19：42：57.785 [INF] LNWL：在tcp：//127.0.0.1：28333上开始通过ZMQ监听bitcoind事务通知
2018-11-25 19：43：04.128 [INF] LNWL：钱包已经解锁，没有时间限制
2018-11-25 19：43：04.129 [INF] LTND：LightningWallet打开了
2018-11-25 19：43：04.178 [INF] LNWL：开始从块0000000000000000001166353e32a530830aba5ab1bb1201330ad61d6019be93（高度551831）重新扫描0地址
2018-11-25 19：43：04.178 [INF] LNWL：从块0000000000000000001166353e32a530830aba5ab1bb1201330ad61d6019be93开始重新扫描
2018-11-25 19：43：04.646 [INF] HSWC：从磁盘恢复内存中的电路状态
2018-11-25 19：43：04.701 [INF] HSWC：加载的支付电路：num_pending = 0，num_open = 0
2018-11-25 19：43：04.746 [INF] SRVR：扫描本地网络以获取启用UPnP的设备
2018-11-25 19：43：05.775 [INF] LNWL：重新扫描结束于551831（0000000000000000001166353e32a530830aba5ab1bb1201330ad61d6019be93）
2018-11-25 19：43：05.864 [INF] LNWL：赶上高度551831的块哈希，这可能需要一段时间
2018-11-25 19：43：05.898 [INF] LNWL：完成追赶块哈希
2018-11-25 19：43：05.898 [INF] LNWL：已完成重新扫描0地址（同步到块0000000000000000001166353e32a530830aba5ab1bb1201330ad61d6019be93，高度551831）
2018-11-25 19：43：09.030 [INF] SRVR：使用UPnP自动设置端口转发以通告外部IP
2018-11-25 19：43：09.456 [INF] RPCS：侦听127.0.0.1:10009的RPC服务器
2018-11-25 19：43：09.456 [INF] RPCS：gRPC代理起始于127.0.0.1:8080
2018-11-25 19：43：09.463 [INF] LTND：等待链后端完成同步，start_height = 551832
2018-11-25 19：43：09.481 [INF] LTND：链后端完全同步（end_height = 551832）！
2018-11-25 19：43：09.528 [INF] NTFN：新的块时期订阅
2018-11-25 19：43：09.528 [INF] HSWC：启动HTLC Switch
2018-11-25 19：43：09.528 [INF] NTFN：新的块时期订阅
2018-11-25 19：43：09.529 [INF] NTFN：新的块时期订阅
2018-11-25 19：43：09.542 [INF] UTXN：处理丢失块的输出。从blockHeight = 551831开始，到当前blockHeight = 551832
2018-11-25 19：43：09.542 [INF] UTXN：尝试毕业身高= 551832：num_kids = 0，num_babies = 0
2018-11-25 19：43：09.650 [INF] UTXN：UTXO Nursery现已完全同步
2018-11-25 19：43：09.650 [INF] DISC：经过身份验证的Gossiper正在启动
2018-11-25 19：43：09.651 [INF] BRAR：开始合同观察员，观察违规行为。
2018-11-25 19：43：09.651 [INF] NTFN：新的块时期订阅
2018-11-25 19：43：09.655 [INF] CRTR：FilteredChainView开始
2018-11-25 19：43：15.542 [INF] CRTR：使用12730个通道激活的过滤链
2018-11-25 19：43：15.554 [INF] CRTR：通道图的修剪提示：高度= 551831，哈希= 0000000000000000001166353e32a530830aba5ab1bb1201330ad61d6019be93
2018-11-25 19：43：15.559 [INF] CRTR：将高度= 551831（散列= 0000000000000000001166353e32a530830aba5ab1bb1201330ad61d6019be93）的通道图同步到高度= 551832（散列= 0000000000000000001ab31e179a5e77b16ee0ddb1c8b07ae2b2dc11d06d68fa）
2018-11-25 19：43：16.452 [INF] CRTR：Block 0000000000000000001ab31e179a5e77b16ee0ddb1c8b07ae2b2dc11d06d68fa（height = 551832）已关闭0个频道
2018-11-25 19：43：16.452 [INF] CRTR：图形修剪完成：自高度551831以来关闭了0个通道
2018-11-25 19：43：16.563 [INF] CMGR：服务器监听[::]：9735
2018-11-25 19：43：16.607 [INF] SRVR：初始化对等网络引导程序！
2018-11-25 19：43：16.607 [INF] SRVR：使用种子创建DNS对等引导程序：[[nodes.lightning.directory soa.nodes.lightning.directory]]
2018-11-25 19：43：16.608 [INF] DISC：尝试使用以下内容进行引导：经过身份验证的通道图
2018-11-25 19：43：16.630 [INF] DISC：用bootstrap网络获得3个addrs
2018-11-25 19：43：16.806 [INF] SRVR：建立连接：86.70.56.25：9735
...
```

要恢复正常操作，请使用“Ctrl-C”关闭LND，然后

```
...
2018-11-25 19：43：27.382 [INF] PEER：断开连接104.196.6.10:9735，原因：服务器：断开对等端104.196.6.10:9735
2018-11-25 19：43：27.494 [INF] LTND：关机完成

$退出
$ sudo systemctl start lnd
```



-----

我将不断扩展此故障排除指南，并在问题部分报告已经或将要报告的结果。



