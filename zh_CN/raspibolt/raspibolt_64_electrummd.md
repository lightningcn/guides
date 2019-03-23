[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 额外指南：Electrum Personal Server

*难度：中级*

### 介绍

保护比特币（意味着安全性和可用性的最佳组合）的最佳方法是使用硬件钱包（如[Ledger]（https://www.ledgerwallet.com/）或[Trezor]（https：// trezor） .io /））与您自己的比特币节点结合使用。这为您提供了安全性，隐私性，并且无需信任第三方来验证交易。

使用RaspiBolt设置，节点上的比特币核心钱包只能在命令行中使用，因为没有安装图形用户界面。由于比特币核心不支持硬件钱包，因此只能实现“热钱包”（暴露于互联网）。

使用比特币核心更多功能的一种可能性是设置一个额外的[ElectrumX]（https://github.com/kyuupichan/electrumx）服务器，然后使用伟大的[电子钱包]（https://electrum.org/ ）（在您的常规计算机上）与硬件钱包集成。但这种设置并不容易，而且开销不仅仅是Raspberry Pi可以处理的。

新的[Electrum Personal Server]（https://github.com/chris-belcher/electrum-personal-server）可以将Electrum（使用硬件钱包）直接连接到RaspiBolt。与ElectrumX相比，这不是一个为多个用户服务的完整服务器，而是您自己的专用后端。

在使用此设置之前，请通过设置自己的Electrum钱包，访问链接的项目网站并阅读[Electrum Personal Server将为用户提供他们需要的完整节点安全性]来熟悉所有组件（https://bitcoinmagazine.com/比特币杂志中的文章/ electrum-personal-server-will-give-users-full-node-security-they-need /）。

### 准备工作

* 使用用户'admin'，确保安装了Python3和PIP
  `$ sudo apt install -y python3 python3-pip`

* 配置防火墙以允许传入请求（请检查是否需要调整子网掩码，如[原始设置中所述]（raspibolt_20_pi.md #enable-the-uncomplicated-firewall））
  ```
  $ sudo ufw allow from 192.168.0.0/24 to any port 50002 comment 'allow EPS from local network'
  $ sudo ufw enable
  $ sudo ufw status
  ```

Electrum Personal Server使用带有“仅限观看”地址的比特币核心钱包来监控区块链。

* 确保在“bitcoin.conf”中没有设置`disablewallet = 1`（它可以丢失，或设置为'0`）。保存并退出。
  `$ sudo nano / home / bitcoin / .bitcoin / bitcoin.conf`

* 如果您更改了`bitcoin.conf`，请重新启动bitcoind
  `$ sudo systemctl restart bitcoind`

### 安装Electrum Personal Server

* 打开“比特币”用户会话并切换到主目录
  `$ sudo su  - 比特币`

* 下载，验证并提取最新版本（查看Github上的[版本页面]（https://github.com/chris-belcher/electrum-personal-server/releases）以获取正确的链接）

  ```
  # create new directory on external hdd
  $ mkdir /mnt/hdd/electrum-personal-server
  $ ln -s /mnt/hdd/electrum-personal-server /home/bitcoin/electrum-personal-server
  $ cd electrum-personal-server

  # download release
  $ wget https://github.com/chris-belcher/electrum-personal-server/archive/eps-v0.1.6.tar.gz
  $ wget https://github.com/chris-belcher/electrum-personal-server/releases/download/eps-v0.1.6/eps-v0.1.6.tar.gz.asc
  $ wget https://raw.githubusercontent.com/chris-belcher/electrum-personal-server/master/pgp/pubkeys/belcher.asc

  # verify that the release is signed by Chris Belcher (check the fingerprint)
  $ gpg --import belcher.asc
  $ gpg --verify eps-v0.1.6.tar.gz.asc
  > gpg: Good signature from "Chris Belcher <false@email.com>" [unknown]
  > Primary key fingerprint: 0A8B 038F 5E10 CC27 89BF  CFFF EF73 4EA6 77F3 1129

  $ tar -xvf eps-v0.1.6.tar.gz  
  $ rm *.gz*
  ```

* 复制和编辑配置模板（更新时跳过此步骤）
  ``` 
  $ cp electrum-personal-server-eps-v0.1.6/config.cfg_sample config.cfg
  $ nano config.cfg
  ```

  * 将钱包主公钥或仅限监视地址添加到`[master-public-keys]“和`[watch-only-addresses]`部分。可以在Electrum客户端菜单“钱包” - >“信息”中找到Electrum钱包的主公钥。

  * 在`[bitcoin-rpc]`中，取消注释并完成这些行。
    ```
    rpc_user = raspibolt
    rpc_password = [PASSWORD_B]
    ```

  * 在`[electrum-server]`中，将监听`host`改为`0.0.0.0`，这样你就可以从远程计算机到达它。防火墙只接受来自家庭网络的连接，而不接受来自互联网的连接。
    ```
    host = 0.0.0.0
    ```

* 保存并退出

* 安装Electrum Personal Server
  ```
  $ cd electrum-personal-server-eps-v0.1.6/
  $ pip3 install --user .
  ```

  ！[使用Python Pip安装Electrum Personal Server]（./ images / 60_eps_pip_install.png）

### 首先开始
Electrum Personal Server脚本安装在`/ home / bitcoin / .local / bin /`目录中。不幸的是，在Raspbian中，此目录不在系统路径中，因此在调用这些脚本时需要指定完整路径。或者，只需[将此目录添加到$ PATH环境变量]（https://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path），但这不是必需的在本指南中。

  * 第一次运行服务器时，它会将所有已配置的地址作为仅限监视导入比特币节点。这可能需要10分钟，之后程序将退出。
    ```
    $ /home/bitcoin/.local/bin/electrum-personal-server /home/bitcoin/electrum-personal-server/config.cfg
    ```

  * 如果您的钱包有先前的交易，Electrum Personal Server需要重新扫描比特币区块链以获取历史信息。整个区块链可能需要很长时间，因此您可以设置扫描的开始日期（每年的历史记录仍然需要超过1小时）。
    ```
    $ /home/bitcoin/.local/bin/electrum-personal-server-rescan /home/bitcoin/electrum-personal-server/config.cfg
    ```  

  * 您可以从第二个SSH会话监控比特币核心日志文件中的重新扫描进度：
    ```
    $ sudo tail -f /home/bitcoin/.bitcoin/debug.log
    ```

  * 再次运行Electrum Personal Server并从常规计算机连接您的Electrum钱包。
    ```
    $ /home/bitcoin/.local/bin/electrum-personal-server /home/bitcoin/electrum-personal-server/config.cfg
    ``` 

  [！[手动运行Electrum Personal Server]（images / 60_eps_first-start.png）](https://github.com/Stadicus/guides/blob/master/raspibolt/images/60_eps_first-start.png)

### 连接Electrum

在常规计算机上，配置Electrum以使用RaspiBolt：

* 在菜单中：`工具>网络>服务器`
* 取消选中“自动选择服务器”
* 在地址字段中输入RaspiBolt的IP（例如192.168.0.20）

  [！[将电子连接到RaspiBolt]（images / 60_eps_electrum-connect.png）](https://github.com/Stadicus/guides/blob/master/raspibolt/images/60_eps_electrum-connect.png)

* “关闭”并在“控制台”选项卡中检查连接

  [！[检查电子控制台]（images / 60_eps_electrumwallet.png）](https://github.com/Stadicus/guides/blob/master/raspibolt/images/60_eps_electrumwallet.png)

* 这也可以通过使用以下命令行参数启动Electrum钱包来实现：
  `--oneserver --server 192.168.0.20：50002：s`

### 自动启动
如果一切按预期工作，我们现在将自动启动RaspiBolt上的Electrum Personal Server。

* 在Pi上，按“Ctrl-C”退出Electrum Personal Server

* 退出“比特币”用户会话回到用户“admin”
  `exit`

* 作为“admin”，设置systemd单元以在启动时自动启动，保存并退出
  `$ sudo nano / etc / systemd / system / eps.service`

```
[单元]
说明= Electrum Personal Server
之后= bitcoind.service

[服务]
ExecStart = / usr / bin / python3 /home/bitcoin/.local/bin/electrum-personal-server /home/bitcoin/electrum-personal-server/config.cfg
用户=比特币
组=比特币
类型=简单
KillMode =过程
TimeoutSec = 60
重启=始终
RestartSec = 60

[安装]
WantedBy = multi-user.target
```

* 启用并启动eps.service单元
  `$ sudo systemctl enable eps.service`
  `$ sudo systemctl start eps.service`

* 检查Electrum Personal Server的启动过程
  `$ tail -f / tmp / electrumpersonalserver.log`

---

### 不要相信，验证。

恭喜，您现在拥有最好的比特币桌面钱包之一，能够通过支持硬件钱包来保护您的比特币，并使用您自己的无信任比特币全节点运行！

---

<<返回：[奖励指南]（raspibolt_60_bonus.md）
