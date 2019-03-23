[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [**主网**]  -  [[奖金]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

-------
### Raspberry Pi上的️⚡Lightning️⚡初学者指南
--------

# Mainnet
把真正的比特币放在线上你觉得舒服吗？这是怎么做的。

⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️

但首先：如果你不熟悉自己更深入地学习Linux，或者甚至从源代码编译程序，那么你很快就会失去一些资金。闪电网尚未投入生产，LND仍处于测试阶段。

```
个人免责声明：本指南按原样提供，不作任何保证。大多数组件都是
正在开发中，本指南可能包含导致比特币丢失的事实错误。
使用本指南需要您自担风险。
```
```
Lightning Labs免责声明：由于这是lnd的第一个mainnet版本，我们建议用户使用
只进行少量试验（#craefulgang #craefulgang #craefulgang）。
```

⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️

还想试试吗？继续需要您自担风险。

## 复制mainnet区块链

当前设置在比特币测试网上运行。然而，在开始时，我们开始在您的普通计算机上下载比特币主网络区块链。在此计算机上直接在比特币核心中检查验证进度。要继续，应该完全同步（请参阅状态栏）。

验证完成后，立即关闭Windows上的Bitcoin Core。我们现在将整个数据结构复制到RaspiBolt。这大约需要6个小时。

### 暂时启用密码登录
为了使用用户“比特币”复制数据，我们需要临时启用密码登录。

* 作为用户“admin”，编辑SSH配置文件并在“PasswordAuthentication no”前面放置一个“＃”以禁用整行。保存并退出。
  `$ sudo nano / etc / ssh / sshd_config`
  `＃PasswordAuthentication no`

* 重新启动SSH守护程序。
  `$ sudo systemctl restart ssh`

### 使用WinSCP复制
我们正在使用“安全复制”（SCP），因此[下载并安装WinSCP]（https://winscp.net）是一个免费的开源程序。

* 使用WinSCP，您现在可以使用用户“比特币”连接到您的Pi。
！[WinSCP连接设置]（images / 50_WinSCP_connection.png）

* 接受服务器证书并导航到本地和远程比特币目录：
  * 本地：`d：\ bitcoin \ bitcoin_mainnet \`
  * 远程：`\ mnt \ hdd \ bitcoin \`

* 您现在可以将两个子目录`blocks`和`chainstate`从Local复制到Remote。这将需要大约6个小时。
！[WinSCP副本]（images / 50_WinSCP_copy.png）

：警告：转移不得中断。确保您的计算机没有进入睡眠状态。

：point_right：_附加信息：[比特币核心数据目录结构]（https://en.bitcoin.it/wiki/Data_directory）

### 再次禁用密码登录
* 作为用户“admin”，删除“PasswordAuthentication no”前面的`#`以启用该行。保存并退出。
  `$ sudo nano / etc / ssh / sshd_config`
  `PasswordAuthentication no`

* 重新启动SSH守护程序。
  `$ sudo systemctl restart ssh`

## 发回你的testnet比特币

为了避免烧毁我们的测试网比特币，并且为了对下一个测试人员的礼貌，我们关闭了所有渠道，并将资金提取到[比特币测试网龙头]网站上所述的地址（https：//testnet.manu.backend。汉堡/水龙头）。

* `$ lncli --network = testnet closeallchannels`

* 等待单位信道余额为零，资金将返回我们的链上钱包。
  `$ lncli --network = testnet channelbalance`
  `$ lncli --network = testnet walletbalance`

- 发送'walletbalance`提供的金额减去500 satoshis以计算费用。如果您收到“资金不足”错误，请扣除更多信息，直到广告获得广播为止。
  `$ lncli --network = testnet sendcoins 2N8hwP1WmJrFF5QWABn38y63uYLhnJYJYTF [金额]`

## 调整配置

* 停止比特币和闪电服务。
  `$ sudo systemctl stop lnd`
  `$ sudo systemctl stop bitcoind`

* 通过注释`testnet = 1`来编辑“bitcoin.conf”文件。保存并退出。
  `$ sudo nano / home / bitcoin / .bitcoin / bitcoin.conf`
```
＃删除以下行以启用比特币主网
＃testnet = 1
```

* 将更新的“bitcoin.conf”复制到用户“admin”以获取凭据
  `$ sudo cp /home/bitcoin/.bitcoin/bitcoin.conf / home / admin / .bitcoin /`


* 通过从`bitcoin.testnet = 1`切换到`bitcoin.mainnet = 1`来编辑“lnd.conf”文件。保存并退出。
  `$ sudo nano / home / bitcoin / .lnd / lnd.conf`
```
＃启用testnet或mainnet
＃bitcoin.testnet = 1
bitcoin.mainnet = 1
```
## 重启bitcoind＆lnd for mainnet

：警告：**在完成主网区块链的复制任务之前，请勿继续执行**。

* 启动Bitcoind并检查它是否在mainnet上运行（您可以使用`Ctrl-C`退出debug.log）

  ```
  $ sudo systemctl start bitcoind  
  $ systemctl status bitcoind.service  
  $ sudo tail -f /home/bitcoin/.bitcoin/debug.log   
  $ bitcoin-cli getblockchaininfo 
  ```

* **等到区块链完全同步**：“blocks”=“headers”，否则在创建新的lnd主网钱包时可能会遇到性能/内存问题。

* 启动LND并检查其操作

  ```
  $ sudo systemctl start lnd
  $ systemctl status lnd
  $ sudo journalctl -f -u lnd
  ```

  如果一切正常，请重新启动RaspiBolt并再次检查操作。
  `$ sudo shutdown -r now`

* 监视第一个`bitcoind`然后`lnd`的启动过程

  ```
  $ sudo tail -f /home/bitcoin/.bitcoin/debug.log  
  $ sudo journalctl -f -u lnd
  ```

* 使用与testnet完全相同的**`密码[C]`创建主网钱包。如果您使用其他密码，则需要重新创建访问凭据。
  `$ lncli创建`

* 将权限文件和TLS证书复制到用户“admin”以使用`lncli`

  ```
  $ sudo cp /home/bitcoin/.lnd/tls.cert /home/admin/.lnd  
  $ cd /home/bitcoin/  
  $ sudo cp --parents .lnd/data/chain/bitcoin/mainnet/admin.macaroon /home/admin/
  $ sudo chown admin:admin /home/admin/.lnd/ -R  
  ```

* 重新启动`lnd`并解锁你的钱包（输入`password [C]`）

  ``` 
  $ sudo systemctl restart lnd
  $ lncli unlock
  ```

* 监控LND启动进度，直到它赶上主网区块链（此刻约为515k块）。这可能需要长达2个小时，然后你会看到很多非常快速的聊天（用`Ctrl-C`退出）。
  `$ sudo journalctl -f -u lnd`

* 通过获取一些节点信息确保`lncli`正常工作
  `$ lncli getinfo`

：point_right：**重要**：你需要在每次重启lnd服务后手动解锁lnd钱包！

## 开始使用闪电网络

### 资助您的节点

恭喜，你的RaspiBolt在比特币主网上运行！要打开频道并开始使用它，你需要用一些比特币来资助它。对于初学者，只在你的节点上放置你愿意失去的东西。垄断资金。

* 生成新的比特币地址以接收链上资金
  `$ lncli newaddress np2wkh`
  “>”地址“：”3 ..........................“`

* 从您的普通比特币钱包中，将少量比特币发送到此地址

* 检查您的LND钱包余额
  `$ lncli walletbalance`

* 在区块链资源管理器上监控您的交易：
  https://smartbit.com.au

### LND在行动
一旦您的资金交易被开采和确认，LND将开始开放和维护渠道。此功能称为“自动驾驶”，并在“lnd.conf”文件中配置。如果您想手动维护频道，可以禁用自动驾驶仪。

一些命令尝试：

* 列出命令行界面的所有参数（cli）
   `$ lncli`

* 获得特定参数的帮助
   `$ lncli help [ARGUMENT]`

* 找出有关您的节点的一些常规统计信息：
   `$ lncli getinfo`

* 连接到对等端（您可以在此处找到一些要连接的节点：https：//1ml.com/）：
   `$ lncli connect [NODE_URI]`

* 检查您当前连接的同伴：
   `$ lncli listpeers`

* 与节点开一个通道：
   `$ lncli openchannel [NODE_PUBKEY] [AMOUNT_IN_SATOSHIS] 0
    *请记住，[NODE_URI]最后包含@IP：PORT，而[NODE_PUBKEY]不包含*

* 检查待处理渠道的状态：
   `$ lncli pendingchannels`

* 检查您的有效频道的状态：
   `$ lncli listchannels`

* 在支付发票之前，您应解码它以检查金额和其他信息是否正确：
   `$ lncli decodepayreq [INVOICE]`

* 支付发票：
   `$ lncli payinvoice [INVOICE]`

* 检查您发送的付款：
   `$ lncli listpayments`

* 创建发票：
   `$ lncli addinvoice [AMOUNT_IN_SATOSHIS]`

* 列出所有发票：
  `$ lncli listinvoices`

* 要关闭一个通道，你需要以下两个参数，这些参数可以用`listchannels`确定，并列为“channelpoint”：`FUNDING_TXID`：`OUTPUT_INDEX`。
   `$ lncli listchannels`
   `$ lncli closechannel [FUNDING_TXID] [OUTPUT_INDEX]`

* 要强制关闭某个频道（如果您的同伴离线或不合作），请使用
   `$ lncli closechannel --force [FUNDING_TXID] [OUTPUT_INDEX]`

👉有关其他信息，请参阅[LND API参考]（http://api.lightning.community/）

### 试试看
要试用您的新Lightning节点，您可以向我发送一个微提示：
[文章'初学者指南️⚡Lightning️⚡在树莓派上']（https://mainnet.yalls.org/articles/97d67df1-d721-417d-a6c0-11d793739be9:0965AC5E-56CD-4870-9041-E69616660E6F/70858a49 -d91c-40fb-ae34-bddc2e938704）你们都有（0.01美元）

### 探索闪电主网
关于您自己的节点，有很多很好的资源可以探索Lightning主网。

* [Recksplorer]（https://rompert.com/recksplorer/）：闪电网络地图
* [1ML]（https://1ml.com）：Lightning网络搜索和分析引擎
* [lnroute.com]（http://lnroute.com）：全面的Lightning网络资源列表

### （可选）添加别名以获得更简单的命令
如果您不想每次都输入完整的命令，别名将有所帮助。有关别名设置，请参见[Additional scripts]（raspibolt_67_additional-scripts.md）部分。

---
下一篇：[奖金>>]（raspibolt_60_bonus.md）
