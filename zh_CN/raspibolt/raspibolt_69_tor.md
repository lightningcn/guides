[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 额外指南：带Tor的匿名节点
*难度：中等*

### 什么是Tor？

Tor是一款免费软件，允许您通过网络节点路由来匿名互联网流量，以隐藏端点的位置和使用情况。

它被称为“洋葱路由器”的“Tor”：信息被其通过的节点的公钥多次加密。每个节点解密与其自己的私钥对应的信息层，就像剥洋葱一样，直到最后一个将显示清晰的消息。

：point_right：在[维基百科]上了解更多信息（https://en.wikipedia.org/wiki/Tor_%28anonymity_network%29）。

### 为什么要运行Tor？

Tor主要用作阻碍流量分析的方法，这意味着分析您的互联网活动（在您正在浏览的网站和您正在使用的服务上记录您的IP地址），以了解您和您的兴趣。流量分析对于广告非常有用，您可能只想隐藏此类信息，而不仅仅是出于隐私问题。但它也可能被彻头彻尾的恶意行为者，罪犯或政府用来以很多可能的方式伤害你。

Tor允许您在互联网上共享数据而不会泄露您的位置或身份，这在运行比特币节点时绝对有用。

在你应该运行Tor的所有原因中，以下是与比特币最相关的：
* 通过用您的节点公开您的家庭IP地址，您实际上是在说“整个星球”在这个家中我们运行一个节点“。这只是“在这个家里，我们确实有比特币”的一小步，这可能会让你和你的亲人成为盗贼的目标。
* 如果您所居住的国家/地区完全成熟禁令和打击比特币所有者，您将成为执法的明显目标。
* 与CoinJoin等其他隐私方法相结合，您可以获得更多的交易隐私，因为它消除了某人能够窥探您的节点流量，分析您转发的交易并尝试确定哪些UTXO属于您的交易的风险。

所有上述参数在使用Lightning时也是相关的，因为看到在您的家庭IP地址上运行的Lightning节点的人可以很容易地推断出在同一位置存在比特币节点。

### 安装Tor

本指南假设您正在运行** Raspberry Pi 3 **或更高版本。如果您的RaspiBolt是基于早期版本构建的，它将无法如下所述工作，您可能需要[查看这些说明]（https://tor.stackexchange.com/questions/242/how-to-run-反而是在树上覆盆子。

此外，本指南建立在使用** Raspbian Stretch Lite **运行的RaspiBolt指南之上。如果您运行不同的操作系统，则可能需要从源构建Tor，并且路径可能会有所不同。

如需其他参考，可在[Tor项目网站]（https://www.torproject.org/docs/debian.html.en#ubuntu）上获取原始说明。

* 通过SSH以用户“admin”连接到RaspiBolt，如[主要指南中所述]（raspibolt_20_pi.md #connection-to-the-pi）。

* 将以下两行添加到`sources.list`以添加torproject存储库。
  ```
  $ sudo nano /etc/apt/sources.list
  ```
  ```
  deb https://deb.torproject.org/torproject.org stretch main
  deb-src https://deb.torproject.org/torproject.org stretch main
  ```
* 为了验证Tor文件的完整性，请使用网络证书管理服务（dirmngr）下载并添加torproject的签名密钥。
  ```
  $ sudo apt install dirmngr
  $ gpg --keyserver keys.gnupg.net --recv A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89
  $ gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -
  ```

* 现在可以安装最新版本的Tor。虽然不是必需的，但是`tor-arm`提供了一个您可能觉得有用的仪表板。
  ```
  $ sudo apt update
  $ sudo apt install tor tor-arm
  ```

* 检查Tor的版本（它应该是0.3.3.6或更新版本）并且服务已启动并正在运行。
  ```
  $ tor --version
  Tor version 0.3.4.9 (git-074ca2e0054fded1).
  $ systemctl status tor
  ```
* 检查“tor-service-defaults-torrc”文件中的“用户”名称是“debian-tor”。
  ```
  $ cat /usr/share/tor/tor-service-defaults-torrc
  User debian-tor
  ```

* 检查哪些用户属于debian-tor组。如果不存在“比特币”，最有可能出现这种情况，您需要添加它并再次检查。
  ```
  $ cat /etc/group | grep debian-tor
  debian-tor:x:113:
  $ sudo usermod -a -G debian-tor bitcoin
  $ cat /etc/group | grep debian-tor
  debian-tor:x:123:bitcoin
  ```
* 通过取消注释（删除＃）或添加以下行来修改Tor配置。
  ```
  $ sudo nano /etc/tor/torrc
  ```
  ```
  # uncomment:
  ControlPort 9051
  CookieAuthentication 1

  # add:
  CookieAuthFileGroupReadable 1
  ```

* 重新启动Tor以激活修改
  ```
  $ sudo systemctl restart tor
  ```

### 为比特币核心设置Tor

#### 组态

* 在“admin”用户会话中，停止比特币和LND。
  ``` 
  $ sudo systemctl stop lnd
  $ sudo systemctl stop bitcoind
  ```

* 打开比特币配置并添加以下行。不应指定参数“onlynet”（如果存在，则删除此行）。
  ```
  $ sudo nano /home/bitcoin/.bitcoin/bitcoin.conf
  ```
  ```
  # add / change:
  proxy=127.0.0.1:9050
  bind=127.0.0.1
  listenonion=1
  ```

* 存档当前日志文件并重新启动Bitcoin Core以使用调整后的配置。
  ```
  $ sudo mv /home/bitcoin/.bitcoin/debug.log /home/bitcoin/.bitcoin/debug.log.old
  $ sudo systemctl start bitcoind
  ```

#### 验证
比特币核心正在启动，我们现在需要检查所有连接是否真正通过Tor路由。

* 验证`debug.log`文件中的操作。大约一分钟后你应该看到你的洋葱地址。

  ```
  $ sudo tail /home/bitcoin/.bitcoin/debug.log -f -n 200
  InitParameterInteraction: parameter interaction: -proxy set -> setting -upnp=0
  InitParameterInteraction: parameter interaction: -proxy set -> setting -discover=0
  ...
  torcontrol thread start
  ...
  tor: Got service ID [YOUR_ID] advertising service [YOUR_ID].onion:8333
  addlocal([YOUR_ID].onion:8333,4)
  ```
  ！[Tor安装的debug.log输出]（./ images / 69_startup.png）
  ！[Tor安装2的debug.log输出]（./ images / 69_startup2.png）

* 显示比特币网络信息，以验证不同的网络协议是否绑定到代理“127.0.0.1：9050”，即本地主机上的Tor。注意`onion`网络现在是`可达的：true`。
  ```
  $ bitcoin-cli getnetworkinfo
  ```
  ！[bitcoin-cli getnetworkinfo列出网络协议绑定]（./ images / 69_networkinfo.png）

* 验证您的节点是否可以在比特币网络中访问。转到[bitnodes.earn.com]（https://bitnodes.earn.com/）并在此处复制/粘贴您的`.onion`地址：
  ！[bitnodes]（./图像/ 69_bitnodes.png）

  如果结果是否定的，请稍后再尝试，因为有时查找将无法联系您的节点。如果您的节点持续无法访问，请验证您的[端口转发]（raspibolt_20_pi.md＃port-forwarding  -  upnp）和[防火墙]（raspibolt_20_pi.md #enable-the-uncomplicated-firewall）设置。

* 检查其他对等方用于连接到节点的IP地址。首先，获取真实的公共IP地址，然后验证它是否被比特币用作“localaddr”。
  ```
  $ curl icanhazip.com
  $ bitcoin-cli getpeerinfo | grep  local
  ```

  ：警告：如果您仍然看到列出了您的真实公共IP地址，那么由于您的一个或多个同行目前已在clearnet上与您联系，因此出现了问题。这意味着您实际上是匿名化的。

* 如果您此刻不打算配置LND，也请启动该服务。否则，您可以跳过此步骤。
  ```
  $ sudo systemctl start lnd
  ```

**其他视频指南**
：point_right：如果你有点迷失，你可以观看[这个视频]（https://youtu.be/57GW5Q2jdvw）非常清楚，并显示几乎相同的过程（还有一些额外的可选步骤，我描述如下）。

### 为LND设置Tor

两个要点：
* LND需要Tor版本0.3.3.6或更新版本。如果您按照本教程安装Tor，这应该不是问题。
* 如果您以前在clearnet上运行过节点，建议关闭所有Lightning频道并在Tor上启动一个全新的节点。您现有的公钥已经与您的真实IP地址相关联，并且已为您的同行所知，因此使用此数据您很容易进行去匿名化。

好的，让我们开始工作吧。

* 使用“admin”用户，停止LND：
  ```
  $ sudo systemctl stop lnd
  ```

* 打开LND配置文件并添加/更改以下行。
  ```
  $ sudo nano /home/bitcoin/.lnd/lnd.conf
  ```
  ```
  # add / change the following options within [Application Options]:
  listen=localhost
  nat=false

  # add:
  [Tor]
  tor.active=true
  tor.v3=true
  ```

* 像往常一样重新启动LND，给它一些时间并解锁钱包：
  ```
  $ sudo systemctl start lnd
  $ lncli unlock
  ```

* `lncli getinfo`或`lncli getnodeinfo [YOUR_PUBKEY]`命令的输出不再显示您的IP地址。

：point_right：[LND项目Github存储库]提供了更多信息（https://github.com/lightningnetwork/lnd/blob/master/docs/configuring_tor.md）。

### 再往前走一点

您的比特币和闪电节点现在通过Tor网络连接到世界，并且更难以隔离并与地理位置相关联。但你应该知道Tor不是[银弹]（https://security.stackexchange.com/questions/147402/how-do-traffic-correlation-attacks-against-tor-users-work）并且你是仍然容易受到从“简单”DoS到用户完全去匿名化的一系列攻击。此配置旨在成为暴露于平均风险的普通用户的性能和安全性之间的良好折衷。

可以进一步减少攻击面，但这可能会妨碍您与其他对等方连接的能力。但是，这可能会使您与网络的其余部分失去同步。

例如，您可以：

* **仅连接到Tor节点**
  接受仅与具有`.onion`地址的对等体连接。在比特币配置文件中，添加以下行：

  ```
  $ sudo nano /home/bitcoin/.bitcoin/bitcoin.conf
  ```
  ```
  onlynet=onion
  ```

  如果你查看`bitcoin-cli getnetworkinfo`，你会注意到'ipv4`和`ipv6`网络不再可达。您现在只能连接到Tor网络上的对等端。

* **停用DNS查找**
  其他对等体的DNS查找可能会用来对您进行去匿名化，至少它是在过去发生过的。有些人可能想要停用通常用于查找网络上其他节点的DNS请求。

  使用此配置，您的节点无法自行查找对等项。这就是为什么有必要使用硬编码的几个节点列表来引导它，以便在启动时通过指定`addnode`来联系。您需要为每个地址添加一行。您可以在线查找地址列表，例如[此处]（https://bitcoin.stackexchange.com/questions/70069/how-can-i-setup-bitcoin-to-be-anonymous-with-tor），但它提出其他风险所以要小心......

  在比特币配置文件中，添加以下行：
  ```
  $ sudo nano /home/bitcoin/.bitcoin/bitcoin.conf
  ```
  ```
  dnsseed=0
  dns=0
  addnode=[ADDRESS].onion(:port)
  addnode=[ADDRESS].onion(:port)
  ...
  ```

  如果您想了解更多有关DNS的信息，可以查看[Wikipedia]（https://fr.wikipedia.org/wiki/Domain_Name_System）或令人难以置信的（漫画小册子）Zine [Networking]（https：// wizardzines .com / zines / networking /）作者：Julia Evans。

------

每次更改内容时，不要忘记使用`sudo systemctl restart bitcoind`重新启动bitcoind。

------

<<返回：[奖励指南]（raspibolt_60_bonus.md）
