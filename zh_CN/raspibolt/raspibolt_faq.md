[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [Lightning]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[Bonus]（raspibolt_60_bonus.md）]  -  [** [疑难解答]（raspibolt_70_troubleshooting.md）**]

-------
### Raspberry Pi上的️⚡Lightning️⚡初学者指南
--------

# 一般常见问题

### 我可以通过路由Lightning付款致富吗？
没人知道。可能不是。您将获得最低费用。我不在乎。享受车程！

### 我可以将Ext4格式的硬盘连接到我的Windows计算机吗？
Ext4文件系统与标准Windows不兼容，但与Paragon Software的[Linux文件系统]（https://www.paragon-software.com/home/linuxfs-windows/#faq）等其他软件兼容（它们提供了10天免费试用）这是可能的。

### 所有Linux命令都做了什么？
这是一个（非常）简短的常用Linux命令列表供您参考。对于特定命令，可以输入`man [command]`来显示手册页（输入`q`退出）。

| 命令 | 描述 | 例 |
| -- | -- | -- |
| `cd` | 切换到目录 | `cd / home / bitcoin` |
| `ls` | 列出目录内容 | `ls -la / mnt / hdd` |
| `cp` | 复制 | `cp file.txt newfile.txt` |
| `mv` | 移动 | `mv file.txt moved_file.txt`
| `rm` | 去掉 | `rm temporaryfile.txt`
| `mkdir` | 制作目录 | `mkdir / home / bitcoin / newdirectory`
| `ln` | 建立链接 | `ln -s / target_directory / link`
| `sudo` | 以超级用户身份运行命令 | `sudo nano textfile.txt`
| `su` | 切换到不同的用户帐户 | `sudo su bitcoin`
| `chown` | 更改文件所有者  | `chown比特币：比特币myfile.txt`
| `chmod` | 更改文件权限 | `chmod + x executable.script`
| `nano` | 文本文件编辑器 | `nano textfile.txt`
| `tar` | 存档工具 | `tar -cvf archive.tar file1.txt file2.txt`
| `exit` | 退出当前用户会话 | `exit`
| `systemctl` | 控制系统服务 | `sudo systemctl start bitcoind`
| `journalctl` | 查询systemd日志 | `sudo journalctl -u bitcoind`
| `htop` | 监控流程和资源使用情况 | `htop`
| `shutdown` | 关机或重启Pi | `sudo shutdown -r now`


### 我在哪里可以获得更多信息？
如果您想了解有关比特币的更多信息并对Lightning Network的内部运作感到好奇，比特币杂志中的以下文章提供了非常好的介绍：

* [什么是比特币？](https://bitcoinmagazine.com/guides/what-bitcoin)
* [了解闪电网络](https://bitcoinmagazine.com/articles/understanding-the-lightning-network-part-building-a-bidirectional-payment-channel-1464710791/)
* 比特币资源：http：//lopp.net/bitcoin.html
* 闪电网资源：[lnroute.com]（http://lnroute.com）


### 如何升级比特币核心？
最新版本可以在比特币核心项目的Github页面上找到。请务必阅读发行说明，因为这些内容可能包含重要的升级信息。
https://github.com/bitcoin/bitcoin/releases

* 您可能希望首先创建[系统备份]（raspibolt_65_system-recovery.md）。

* 作为“admin”用户，停止lnd和bitcoind系统单元
  `$ sudo systemctl stop lnd`
  `$ sudo systemctl stop bitcoind`

* 按照本指南的[比特币部分]（https://github.com/Stadicus/guides/blob/master/raspibolt/raspibolt_30_bitcoin.md）中的说明下载，验证，提取和安装比特币核心二进制文件。

* 启动bitcoind和lnd系统单元
  `$ sudo systemctl start bitcoind`
  `$ sudo systemctl start lnd`

：information_source：请注意比特币核心的内部数据结构从0.16变为0.17。如果您使用其他计算机下载区块链，请确保使用相同的版本。如果升级到0.17，则会自动转换数据结构（可能需要几个小时），并且不再可以将该数据用于旧版本。

### 如何升级LND？
升级可能会导致许多问题。请**始终**完全阅读[LND发行说明]（https://github.com/lightningnetwork/lnd/releases/tag/v0.5-beta）以了解更改。它们还涵盖了许多其他主题和许多此处未提及的新功能。

* 您可能希望首先创建[系统备份]（raspibolt_65_system-recovery.md）。

* 当**升级到LND 0.5 **时，我还建议首先关闭您的频道，因为存在许多需要非常技术性工作来解决它们的问题。

* 作为“admin”用户，停止lnd系统单元
  `$ sudo systemctl stop lnd`

* 如果从低于v0.5的版本升级，请删除macaroon文件。
  `$ sudo rm /home/bitcoin/.lnd / * .macaroon`

* 删除旧的东西，然后下载，验证并安装最新的LND二进制文件
  ```
  $ cd /home/admin/download
  $  rm -f lnd-linux* manifest* pgp_keys.asc
  $ wget https://github.com/lightningnetwork/lnd/releases/download/v0.5-beta/lnd-linux-armv7-v0.5-beta.tar.gz
  $ wget https://github.com/lightningnetwork/lnd/releases/download/v0.5-beta/manifest-v0.5-beta.txt
  $ wget https://github.com/lightningnetwork/lnd/releases/download/v0.5-beta/manifest-v0.5-beta.txt.sig
  $ wget https://keybase.io/roasbeef/pgp_keys.asc

  $ sha256sum --check manifest-v0.5-beta.txt --ignore-missing
  > lnd-linux-armv7-v0.5-beta.tar.gz: OK

  $ gpg ./pgp_keys.asc
  > BD599672C804AF2770869A048B80CD2BB8BD8132

  $ gpg --import ./pgp_keys.asc
  $ gpg --verify manifest-v0.5-beta.txt.sig
  > gpg: Good signature from "Olaoluwa Osuntokun <laolu32@gmail.com>" [unknown]
  > Primary key fingerprint: BD59 9672 C804 AF27 7086  9A04 8B80 CD2B B8BD 8132
  >      Subkey fingerprint: F803 7E70 C12C 7A26 3C03  2508 CE58 F7F8 E20F D9A2

  $ tar -xzf lnd-linux-armv7-v0.5-beta.tar.gz
  $ sudo install -m 0755 -o root -g root -t /usr/local/bin lnd-linux-armv7-v0.5-beta/*
  $ lnd --version
  > lnd version 0.5.0-beta commit=3b2c807288b1b7f40d609533c1e96a510ac5fa6d
  ```

* 从这个版本开始，LND期望两个不同的ZMQ套接字用于块和事务。编辑`bitcoin.conf`，保存并退出。
  ```
  $ sudo nano /home/bitcoin/.bitcoin/bitcoin.conf  
  zmqpubrawblock=tcp://127.0.0.1:28332
  zmqpubrawtx=tcp://127.0.0.1:28333
  ```
* 不再允许选项`debughtlc`，需要删除。编辑`lnd.conf`，保存并退出。
  ```
  $ sudo nano /home/bitcoin/.lnd/lnd.conf  
  #debughtlc=true
  ```
* 使用新配置重新启动服务，并使用“bitcoin”用户解锁钱包。这会创建一组新的蛋白杏仁饼干（如下所述）。
  ```
  $ sudo systemctl restart bitcoind
  $ sudo systemctl restart lnd
  $ sudo su - bitcoin
  $ lncli unlock
  $ exit
  ```

蛋白杏仁饼干现在位于每个支持的网络的链数据目录下。例如，比特币的mainnet admin macaroon现在位于：
  `/家庭/比特币/ .lnd /数据/连锁/比特币/ mainnet / admin.macaroon`

* 将新的蛋白杏仁饼干复制到您的管理员用户，否则该用户不能使用`lncli`。新的蛋白杏仁饼干位置也会影响您可能正在运行的[自动解锁脚本]（https://github.com/Stadicus/guides/blob/master/raspibolt/raspibolt_6A_auto-unlock.md）。
  * 对于** mainnet **，请使用以下命令：
    ```
    $ rm /home/admin/.lnd/admin.macaroon
    $ mkdir -p /home/admin/.lnd/data/chain/bitcoin/mainnet/
    $ sudo cp /home/bitcoin/.lnd/data/chain/bitcoin/mainnet/admin.macaroon /home/admin/.lnd/data/chain/bitcoin/mainnet/
    $ sudo cp /home/bitcoin/.lnd/tls.cert /home/admin/.lnd
    $ sudo chown -R admin:admin /home/admin/.lnd
    $ lncli getinfo
    ```

  * 如果您使用的是** testnet **，请使用上面的mainnet命令，但将目录“mainnet”替换为“testnet”。您还需要始终使用`lncli --network = testnet`，例如`lncli --network = testnet getinfo`。有关如何创建别名以避免每次都输入此别名，请参阅[发行说明]（https://github.com/lightningnetwork/lnd/releases）。

* 别忘了解锁钱包和查看日志
  `$ lncli unlock`
  `$ sudo journalctl -u lnd -f`

### 当我使用带有64位处理器的Raspberry Pi 3时，为什么需要32位版本的比特币？
在撰写本文时（2018年7月），尚未开发出用于Raspberry Pi的64位操作系统。 Raspberry 3版本的64位处理器以32位兼容模式运行，具有32位操作系统。



------

<<返回：[故障排除]（raspibolt_70_troubleshooting.md）