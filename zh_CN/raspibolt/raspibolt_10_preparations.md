[[简介]（README.md）]  -  [**准备**]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [[闪电] （raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[Bonus]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

-------
### Raspberry Pi上的️⚡Lightning️⚡初学者指南
--------

# 准备工作

## 硬件要求
本指南以易于使用且非常灵活的Raspberry Pi为基础。这个令人惊奇的硬件是一个很小的计算机芯片，成本约35美元，消耗的能源非常少。

！[Raspberry Pi]（图片/ 10_raspberrypi_hardware.png）
* Raspberry Pi 3：价格低于40美元的小型计算机*

建议使用最新的Raspberry Pi以获得良好的性能：
* Raspberry Pi 3型号B或更好
* Micro SD卡：8 GB或更多，包括适配器到您的普通电脑
* USB电源适配器：5V / 1.2A（更安培更好）+ Micro USB线
* 外部硬盘：500 GB或更多
* 可选：Raspberry Pi案例

我使用了Raspberry Pi 3 Model B并使用8 GB SD卡进行设置。要运行Lightning节点，必须在本地存储完整的比特币区块链，大约200 GB并且不断增长。我买了一个便宜的硬盘盒，重新使用了一个旧的500 GB硬盘。只要你使用一个像样的电源（2.5A +），一个现代的2.5英寸硬盘，通过USB连接到Pi也能正常工作。

## 下载比特币区块链
比特币区块链记录所有交易，基本上定义谁拥有多少比特币。这是所有信息中最重要的信息，我们不应该依赖其他人来提供这些数据。要在主网上设置我们的比特币全节点，我们需要

* 下载整个区块链（~200 GB），
* 验证曾经发生过的每个比特币交易以及每个曾经开采过的区块，
* 为所有事务创建索引数据库，以便我们稍后可以查询，
* 计算所有比特币地址余额（称为UTXO集）。

：point_right：有关其他信息，请参阅[运行完整节点]（https://bitcoin.org/en/full-node）。

虽然我们将首先为比特币测试网设置RaspiBolt，但比特币主网区块链的验证可能需要几天时间。这就是我们现在已经开始这项任务的原因。

### 使用普通电脑
你可以想象Raspberry Pi不能完成这项巨大的任务。下载不是问题，但由于其低计算能力和内存不足，最初处理整个区块链需要数周或数月。我们需要在普通计算机上下载并验证带有比特币核心的区块链，然后将数据传输到Pi。这只需要进行一次。之后，Pi可以轻松跟上新的块。

本指南假定您将使用Windows计算机执行此任务，但它适用于大多数操作系统。您需要在内部或外部硬盘上提供大约250 GB的可用磁盘空间（但不是为Pi保留的磁盘空间）。由于索引会产生大量的读/写流量，因此硬盘越快越好。内置驱动器或外部USB3硬盘将明显快于具有USB2连接的硬盘。

我们稍后将使用Ext4文件系统格式化Pi的外部硬盘，这最适合我们的用例。然后使用SCP，我们通过本地网络从Windows计算机复制区块链。

### 下载并验证比特币核心
从[bitcoincore.org/en/download](https://bitcoincore.org/en/download）下载比特币核心安装程序并将其存储在您要用于下载区块链的目录中。为了检查程序的真实性，我们计算其校验和并将其与提供的校验和进行比较。

在Windows中，我将使用`>`开始输入所需的所有命令，因此使用命令`> cd bitcoin`，只需输入`cd bitcoin`并按Enter键。

打开Windows命令提示符（`Win + R`，输入`cmd`，点击`Enter`），导航到比特币目录（对我来说，它在驱动器`D：`上，在Windows资源管理器中检查）并创建新目录`bitcoin_mainnet`。然后计算已下载程序的校验和。
```
> D：
> cd \ bitcoin
> mkdir bitcoin_mainnet
> dir
> certutil -hashfile bitcoin-0.17.0.1-win64-setup.exe sha256
a624de6c915871fed12cbe829d54474e3c8a1503b6d703ba168d32d3dd8ac0d3
```
！[Windows命令提示符：验证校验和]（images / 10_blockchain_wincheck.png）

将此值与[发布签名]（https://bitcoin.org/bin/bitcoin-core-0.17.0.1/SHA256SUMS.asc）进行比较。对于Windows v0.17.0.1安装程序二进制文件，它的
```
32位：400c88eae33df6a0754972294769741dce97a706dc22d1438f8091d7647d5506
64位：a624de6c915871fed12cbe829d54474e3c8a1503b6d703ba168d32d3dd8ac0d3
```
通常，您还需要检查此文件的签名，但这在Windows上很痛苦，因此我们稍后会在Pi上执行此操作。

### 安装比特币核心
执行比特币核心安装文件（您可能需要右键单击并选择“以管理员身份运行”）并使用默认设置进行安装。在目录“C：\ Program Files \ Bitcoin”中启动程序`bitcoin-qt.exe`。选择新的“bitcoin_mainnet”文件夹作为自定义数据目录。

！[比特币核心目录选择]（images / 10_bitcoinqt_directory.png）

比特币核心打开并立即开始同步区块链。如果您的计算机有大量内存，您可以通过添加以下行（使用兆字节内存，调整到您的计算机）来增加数据库内存缓存：

```
dbcache = 6000
```
保存并关闭文本文件，使用`File` /`Exit`退出Bitcoin Core并重新启动程序。该程序将再次开始同步。

现在让区块链同步，我们已经可以开始研究Pi了。

---
下一个：[Raspberry Pi >>]（raspibolt_20_pi.md）
