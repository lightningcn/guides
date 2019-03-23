[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

---

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

---

## 奖励指南：Shango Mobile Lightning Wallet

*难度：中等*

移动应用程序Shango（http://shangoapp.com）允许在RaspiBolt上管理LND的基本功能。它提供状态概览，列出同行，允许打开和关闭渠道，当然，您可以进行付款和创建发票。

目前这个应用程序正在进行beta测试。如果您发现了错误，可以通过以下方式报告此项目：https：//github.com/neogeno/shango-lightning-wallet/issues。

！[Shango app overview]（images / 60_shango.png）

### Pi上的准备工作

本指南介绍如何在您自己的网络中使用Shango，同样也连接您的RaspiBolt。完全可以在旅途中使用Shango并在家中连接到您的节点，但这涉及创建新的TLS证书，并且可能与本指南的其他部分冲突。

* 将以下行添加到`[Application Options]`部分的lnd配置文件中

  `$ sudo nano / home / bitcoin / .lnd / lnd.conf`
   ```
   rpclisten=0.0.0.0:10009
   ```

* 打开端口10009，以便Shango钱包可以与您的Lightning节点通信，但只能从本地网络中进行通信（“sudo ufw允许来自192.168.0.0 / 24 ...```下面假设您的Pi的IP地址是类似于```192.168.0。???```，???是0到255之间的任何数字。例如，如果你的IP地址是```12.34.56.78```，你必须适应这行来自```sudo ufw allow from 12.34.56.0 / 24 ...```。，见[更多细节]（https://github.com/Stadicus/guides/blob/master/raspibolt/raspibolt_20_pi.md#hardening - 您-π））

  ```
  $ sudo ufw allow from 192.168.0.0/24 to any port 10009 comment 'allow LND grpc from local LAN'
  $ sudo ufw enable
  $ sudo ufw status
  ```

* 重启LND并解锁钱包
  ```
  $ sudo systemctl restart lnd
  $ lncli unlock
  ``` 

* 安装QR编码器将我们的超级秘密管理员信息传递给应用程序
  ```
  $ sudo apt install qrencode
  $ cd /home/admin/.lnd
  ```

* 设置比特币网络：
  * 在testnet上：`$ export NETWORK = testnet`
  * 在主网上：`$ export NETWORK = mainnet`

### 配置Shango应用程序

* 启动应用程序并转到“设置”/“连接到其他LND服务器”

* 在您的RaspiBolt上，输入以下命令并使用该应用程序“扫描QR”
  ```
  echo -e "$(curl -s ipinfo.io/ip),\n$(xxd -p -c2000 ~/.lnd/data/chain/bitcoin/$NETWORK/admin.macaroon)," > qr.txt && qrencode -t ANSIUTF8 < qr.txt
  ```

* 在应用程序中，输入字段“IP：Port”将填充您的外部IP地址。确保将其替换为您的内部IP地址（例如，`192.168.0.20：10009`）。
* 点击“连接”，应用程序应与您的RaspiBolt同步。

---

<<返回：[奖励指南]（raspibolt_60_bonus.md）
