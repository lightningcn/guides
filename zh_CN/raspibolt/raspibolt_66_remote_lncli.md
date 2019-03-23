[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 额外指南：在另一台计算机上使用`lncli`

*难度：容易*

可以在RaspiBolt上运行* lnd *，在另一台计算机上运行* lncli *。以下说明解释了如何在Windows PC上安装* lncli *并与RaspiBolt进行通信。不同计算机系统（MAC，Linux，...）的说明将非常相似。

在这些说明中，假设lncli计算机与RaspiBolt位于同一LAN上。 lncli计算机可能位于本地LAN之外，但这会带来额外的安全风险，并且不会包含在本指南中。

## RaspiBolt

- 以管理员身份登录

- 允许防火墙中的端口10009

```
admin~฿sudo su
root @ RaspiBolt：/ home / admin #ufw允许从192.168.0.0/24到任何端口10009评论'允许来自本地局域网的rpc'
root @ RaspiBolt：/ home / admin #ufw status
root @ RaspiBolt：/ home / admin＃exit
```
- 在lnd.conf的[Application Options]部分添加一个新行，以允许rpc不仅仅是默认的localhost
  `admin~฿sudo nano / home / bitcoin / .lnd / lnd.conf`

```
[应用选项]
rpclisten = 0.0.0.0：10009
```

- 暂时允许复制admin.macaroon
  `admin~฿sudo chmod 777 / home / bitcoin / .lnd / admin.macaroon`

## Windows PC

- 使用您的浏览器访问https://github.com/lightningnetwork/lnd/releases

- 下载适用于您操作系统的文件。对于Windows10，它通常是lnd-windows-amd64-vx.x.x.zip

- 打开压缩文件并将lncli应用程序（例如lncli.exe）解压缩到桌面。
  ！[Zip文件]（images / 60_remote_zip.png）

- 打开CMD窗口
  `按Win + R，输入cmd，然后按Enter键


- 切换到保存lncli.exe的目录，然后查看帮助信息

```
> cd％USERPROFILE％\ desktop
> lncli
...
全球选择：
   --rpcserver value host：ln守护进程的端口（默认值：“localhost：10009”）
   --lnd的基本目录的lnddir值路径（默认值：“C：\\ Users \\ xxxx \\ AppData \\ Local \\ Lnd”）
   --tlscertpath值到TLS证书的路径（默认值：“C：\\ Users \\ xxxx \\ AppData \\ Local \\ Lnd \\ tls.cert”）
   --no-macaroons禁用macaroon身份验证
    -  macaroon文件的macaroonpath值路径（默认值：“C：\\ Users \\ xxx \\ AppData \\ Local \\ Lnd \\ admin.macaroon”）
   --macaroontimeout值反重播蛋白杏仁饼干有效时间（秒）（默认值：60）
    - 如果设置了macaroonip值，则将macaroon锁定到特定的IP地址
   --help，-h显示帮助
   --version，-v打印版本
```
- 记下默认（基本）目录

- 制作必要的默认目录
  `> mkdir％LOCALAPPDATA％\ Lnd`
* 按照[[Mainnet]（raspibolt_50_mainnet.md）]中的说明使用WinSCP复制显示的文件
  * 本地：`\ Users \ xxxx \ AppData \ Local \ Lnd`
  * 远程：`/ home / bitcoin / .lnd /`
  * 档案：`见下文

 ！[要复制的文件]（images / 60_winLND.png）


 - 回到RaspiBolt：重置admin.macaroon权限
   `admin~฿sudo chmod 600 / home / bitcoin / .lnd / admin.macaroon`


- 在PC上运行lncli
```
> cd％USERPROFILE％\ desktop
> lncli --rpcserver ip.of.your.raspibolt：10009 getinfo
```

## 关于Permisson文件（Macaroons）的一个词

默认情况下，* lncli *将加载* admin.macaroon *，因此具有完全权限。要限制lncli计算机可以执行的操作，您可以删除不需要的蛋白杏仁饼干文件并启动* lncli *指定适当的蛋白杏仁饼干。

例

```
> lncli --macaroonpath％LOCALAPPDATA％\ Lnd \ readonly.macaroon --rpcserver ip.of.your.raspibolt：10009 addinvoice --amt = 100
[lncli] rpc错误：code = Unknown desc =权限被拒绝
```

下表显示了每个蛋白杏仁饼干允许的命令

* ？ =未选中
* n =未检查，推定为否
* 否=否（选中，v0.4.1）
* 是=是（已选中，v0.4.1）


|命令|管理|只读|发票|
|-------| :---: |:---: | :---: |
|创建|是|ñ|没有|
|开锁|是|是|是|
|新地址|是|没有|是|
|sendmany|是|ñ|ñ|
|sendcoins|是|ñ|ñ|
|连|是|ñ|没有|
|断开|是|ñ|没有|
|openchannel|是|ñ|没有|
|closechannel|是|ñ|没有|
|closeallchannels|是|ñ|没有|
|listpeers|是|是|没有|
|walletbalance|是|是|没有|
|channelbalance|是|是|没有|
|获取信息|是|是|没有|
|pendingchannels|是|是|没有|
|sendpayment|是|ñ|没有|
|payinvoice|是|ñ|没有|
|addinvoice|是|没有|是|
|lookupinvoice|是|是|是|
|listinvoices|是|是|是|
|listchannels|是|是|没有|
|listpayments|是|是|没有|
|describegraph|是|是|没有|
|getchaninfo|是|是|没有|
|getnodeinfo|是|是|没有|
|queryroutes|是|是|没有|
|getnetworkinfo|是|是|没有|
|DEBUGLEVEL|是|没有|没有|
|decodepayreq|是|是|没有|
|listchaintxns|是|是|没有|
|停|是|没有|没有|
|signmessage|是|ñ|ñ|
|verifymessage|是|？|ñ|
|feereport|是|是|没有|
|updatechanpolicy|是|没有|没有|
|fwdinghistory|是|是|没有|

* robclark56指南，谢谢！*

------

<<返回：[奖励指南]（raspibolt_60_bonus.md）