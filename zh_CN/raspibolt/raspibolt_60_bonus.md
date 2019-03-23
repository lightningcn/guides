[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [Lightning]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [** Bonus **]  -  [[Troubleshooting]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

# 奖金（可选）

在本节中，您可以找到各种可选主题，使您的RaspiBolt运行更加平滑。我将其拆分为各个小节，因为单个任务可能会很长。

## [**系统概述**]（raspibolt_61_system-overview.md）

*难度：容易*

您的RaspiBolt将在登录时使用快速系统摘要向您致意：

[！[MotD系统概述]（images / 60_status_overview.png）](raspibolt_61_system-overview.md)

## [启动时自动解锁LND]（raspibolt_6A_auto-unlock.md）

*难度：中等*

如果你的RaspiBolt可以在一个壁橱里的某个地方可靠地运行，那么每次系统启动时手动解锁LND钱包是不可行的。此脚本会在启动或服务重启时自动解锁钱包。然而，这需要最小的安全成本，因为密码需要存储在设备上。

## [** Tor **的匿名节点]（raspibolt_69_tor.md）

*难度：中等*

通过Tor网络路由所有比特币流量以保持匿名，避免泄露公共IP地址等私人信息。

[！[托尔]（图像/ 69_tor.png）](raspibolt_69_tor.md)

## [** Electrum Personal Server **]（raspibolt_64_electrum.md）

*难度：中级*

RaspiBolt是您定期进行链接交易的完美无信任比特币后端。与Electrum钱包一起，它甚至可以与您的Ledger或Trezor硬件钱包配合使用。

[！[琥珀金]（图像/ 60_eps_electrumwallet.png）](raspibolt_64_electrum.md)

## [** Zap桌面闪电钱包**]（raspibolt_71_zap.md）

*难度：容易*

Zap桌面应用程序（https://github.com/LN-Zap/zap-desktop）是一个跨平台的Lightning Network钱包，专注于用户体验和易用性。

！[Zap Desktop]（images / 71_zap1_cropped.png）

## [** Shango手机钱包**]（raspibolt_68_shango.md）

*难度：中级*

iOS和Android应用程序Shango为RaspiBolt提供了一个简洁的界面，管理同行和渠道，付款和创建发票。

[！[琥珀金]（图像/ 60_shango.png）](raspibolt_68_shango.md)

## [** Pimp命令行**]（raspibolt_62_commandline.md）

*难度：容易*

使用金฿进行命令行提示并使用更多颜色：

[！[Pimped提示]（images / 60_pimp_prompt_result.png）](raspibolt_62_commandline.md)

## [**在另一台计算机上使用`lncli` **]（raspibolt_66_remote_lncli.md）

*难度：容易*

从网络中的其他计算机控制Lightning节点，例如。来自Windows机器。

## [**系统恢复**]（raspibolt_65_system-recovery.md）

难度：容易

如果您的SD卡损坏或者您的节点变砖，那么手头有一个快速恢复图像很方便。它不是完整的备份解决方案，但允许系统恢复。

## [附加脚本：显示余额和渠道]（raspibolt_67_additional-scripts.md）

难度：容易

这些额外的bash脚本显示平衡概览（链上和通道，活动和非活动）以及格式良好的通道概述。

## 更多额外的东西

** [RaspiBolt-Extras]（https://github.com/robclark56/RaspiBolt-Extras/blob/master/README.md）**作者Rob Clark
* Lights-Out：自动解锁钱包和动态IP
* RaspiBoltDuo：同时运行testnet和mainnet
* 使用REST访问
* 接收Lightning付款：自动创建发票/ qr代码

------

下一页：[故障排除]（raspibolt_70_troubleshooting.md）>>
