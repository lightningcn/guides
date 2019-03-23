[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 奖励指南：电子邮件提醒

*难度：容易*

如果LND重新启动，那么获取电子邮件警报来解锁钱包会很不错。关键是将`sendmail`与简单的免费SMTP服务结合使用。

我使用https://www.easy-smtp.com，他们提供免费计划，您可以使用别名注册。这适用于任何SMTP服务，但有些配置更难（例如，Gmail需要特殊证书）。我们将按照[本指南]（https://www.easy-smtp.com/smtp-sendmail）进行操作。

* 安装`sendmail`
  `$ sudo apt-get install sendmail`
* 

---

<<返回：[奖励指南]（raspibolt_60_bonus.md）