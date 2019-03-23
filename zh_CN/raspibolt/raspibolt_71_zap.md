[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 额外指南：Zap Desktop Lightning Wallet
*难度：容易*

桌面应用程序Zap（https://github.com/LN-Zap/zap-desktop）是一个跨平台的Lightning Network钱包，专注于用户体验和易用性。

下载Zap为您的操作系统：
https://github.com/LN-Zap/zap-desktop/releases
安装说明：https：//github.com/LN-Zap/zap-desktop#install

### Pi上的准备工作

* 允许从您自己的网络连接到RaspiBolt。检查你的Pi的IP地址是如何开始的，例如。 192.168.0或192.168.1，并相应地使用地址。以.0 / 24结尾将允许来自该网络的所有IP地址。
    `$ sudo nano / home / bitcoin / .lnd / lnd.conf`

    将以下行添加到“[Application Options]”部分：
   ```
   tlsextraip=192.168.0.0/24
   rpclisten=0.0.0.0:10009
   ```

* 删除tls.cert（重启LND会重新创建它）：
    `$ sudo rm / home / bitcoin / .lnd / tls。*`

* 重启LND：
  `$ sudo systemctl restart lnd`

* 将新的tls.cert复制到用户“admin”，因为它是lncli所需的：
    `$ sudo cp /home/bitcoin/.lnd/tls.cert / home / admin / .lnd`

* 解锁钱包
  `$ lncli unlock`

* 允许ufw防火墙从LAN侦听10009：
  `$ sudo ufw允许从192.168.0.0/24到任何端口10009评论'允许来自本地LAN的LND grpc'

 * 重启并检查防火墙：
  `$ sudo ufw enable`
  `$ sudo ufw status`

### 复制身份验证文件
要让Zap访问您的Lightning节点，它需要进行身份验证。为此，必须使用两个文件`tls.cert`（用于传输层安全性）和`admin.macaroon`（用于访问LND）。

#### 在Windows上
我们正在使用“安全复制”（SCP），因此[下载并安装WinSCP]（https://winscp.net）是一个免费的开源程序。需要配置为使用SSH私钥登录RaspiBolt。

* 启动WinSCP，使用以下详细信息配置新站点：
  * 文件协议：`SCP`
  * 主机名：`192.168.0.20`
  * 端口号：`22`
  * 用户名：`root`

  ！[WinSCP配置]（images / 71_zap_WinSCP.png）

* 单击Advanced / SSH / Authentication并从文件系统中选择SSH私钥（有关其他信息，请参阅[base guide]（raspibolt_20_pi.md＃login-with-ssh-keys））。单击确定确认。

  ！[WinSCP ssh键配置]（images / 71_zap_WinSCP2.png）

* 单击“保存”以保留配置，为其命名，然后单击“登录”。您可能需要输入SSH私钥密码。

  ！[WinSCP登录]（images / 71_zap_WinSCP3.png）

  ：警告：以root身份登录是访问这些文件所必需的。这只有在您将`authorized_keys`复制到根主目录时才会起作用（参见[base guide]（raspibolt_20_pi.md＃login-with-ssh-keys）。

* 导航到要在左窗格中存储身份验证文件的本地文件夹，并导航到WinSCP右窗格中的`/ mnt / hdd / lnd`。

* 下载文件`/ mnt / hdd / lnd / tls.cert`
  ！[WinSCP文件夹导航]（images / 71_zap_WinSCP4.png）

* 下载文件`/ mnt / hdd / lnd / data / chain / bitcoin / mainnet / admin.macaroon`
  ！[WinSCP文件夹导航]（images / 71_zap_WinSCP5.png）

* 退出WinSCP

#### On your Linux desktop terminal:  

* Copy the tls.cert to your home directory:  
  `$ scp admin@your.RaspiBolt.LAN.IP:/home/admin/.lnd/tls.cert ~/`

* Copy the admin.macaroon to your home directory:  
  `$ scp root@your.RaspiBolt.LAN.IP:/home/bitcoin/.lnd/data/chain/bitcoin/mainnet/admin.macaroon ~/`

### Configure Zap

* Start the app and select:  
  ```Connect your own node```
  ![Zap welcome screen](images/71_zap_desktop1.png)


* Fill in the next screen:  
  Host: `192.168.0.20:10009` (use your own RaspiBolt ip address)  
  TLS Certificate: `C:\path\to\your\tls.cert`  
  Macaroon: `C:\path\to\your\admin.macaroon`  
  ![Zap connection configuration](images/71_zap_desktop2.png)

* Confirm the settings on the following screen and you are done!

![Zap Desktop wallet](images/71_zap_desktop4.png)

------

<<返回：[奖励指南]（raspibolt_60_bonus.md）
