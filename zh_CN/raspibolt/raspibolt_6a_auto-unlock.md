[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 额外指南：启动时自动解锁LND
*难度：中等*

>请注意：本指南尚未更新至LND 0.5，可能无法按预期工作。

每次重新启动LND守护程序时，需要手动解锁LND钱包这一事实需要一个小小的习惯。从安全角度来看，这是有道理的，因为钱包是加密的，并且密钥不存储在同一台机器上。然而，对于可靠的操作，这不是最佳的，因为您可以在由于某种原因（崩溃或停电）重新启动后轻松恢复LND，但随后它被锁定的钱包卡住并且根本无法操作。

这就是自动解锁钱包的脚本有用的原因。密码以纯文本形式存储在root用户目录中，所以显然不那么安全，但对于合理的数量，这在我看来是一个很好的中间立场。您始终可以决定坚持手动解锁，或实施从远程计算机解锁钱包的解决方案。

：警告：重要说明：这仅适用于“systemd”版本230+。您可以按如下方式检查：
`$ systemd --version`


* 作为用户“admin”，创建一个新目录并将LND钱包密码[C]保存到文本文件中
  `$ sudo mkdir / etc / lnd`
  `$ sudo nano / etc / lnd / pwd`

* 以下脚本通过其Web服务（REST接口）解锁LND钱包。一些其他信息：
  * 该脚本检查Pi是刚刚启动还是仅重新启动服务。根据这一点，它等待3分钟（180秒）或10秒钟，以便`lnd`准备就绪。这似乎工作正常，但如果遇到超时问题，您可以调整睡眠时间。
  * 此脚本会自动检测您是使用比特币测试网还是主网。
  * API调用的输出附加到文件`debug.log`。如果你的钱包没有解锁（你不会看到另一个错误），请先检查这里。
  * 所有自动解锁都记录在`audit.log`中。

* 将以下脚本复制到新文件中。
 `$ sudo nano / etc / lnd / unlock`

  ```bash
  #!/bin/sh
  # LND wallet auto-unlock script (Updated for LND 0.5 and above)
  # 2018 by meeDamian, robclark56 (Updated by zwarbo, martinatime, CodingMuziekwijk)

  LN_ROOT="/home/bitcoin/.lnd"
  BITCOIN_DIR="/home/bitcoin/.bitcoin"

  upSeconds="$(cat /proc/uptime | grep -o '^[0-9]\+')"
  upMins=$((${upSeconds} / 60))

  if [ "${upMins}" -lt "5" ]
  then
    /bin/sleep 180s
  else
    /bin/sleep 10s
  fi

  chain="$(bitcoin-cli -datadir=${BITCOIN_DIR} getblockchaininfo | jq -r '.chain')"

  curl -s \
          -H "Grpc-Metadata-macaroon: $(xxd -ps -u -c 1000 ${LN_ROOT}/data/chain/bitcoin/${chain}net/admin.macaroon))" \
          --cacert ${LN_ROOT}/tls.cert \
          -X POST -d "{\"wallet_password\": \"$(cat /etc/lnd/pwd | tr -d '\n' | base64 -w0)\"}" \
          https://localhost:8080/v1/unlockwallet >> /etc/lnd/debug.log 2>&1

  echo "$? $(date)" >> /etc/lnd/audit.log
  exit 0
  ```

* 使目录和所有内容仅对“root”可访问

  ```bash
  $ sudo chmod 400 /etc/lnd/pwd
  $ sudo chmod 100 /etc/lnd/unlock
  $ sudo chown root:root /etc/lnd/*
  ```

* 编辑LND系统单元。这在LND运行后直接启动脚本。
  `$ sudo nano /etc/systemd/system/lnd.service`

  ```bash
  # remove this line (if present):
  # PIDFile=/home/bitcoin/.lnd/lnd.pid

  # add this line directly below ExecStart:
  ExecStartPost=+/etc/lnd/unlock

  # make sure that the overall timeout is longer than the script wait time, eg. 240s
  TimeoutSec=240
  ```

* 编辑LND配置文件以在端口8080上启用REST接口
  `$ sudo nano / home / bitcoin / .lnd / lnd.conf`

  ```bash
  # add the following line in the [Application Options] section
  restlisten=localhost:8080
  ```

* 重新加载systemd设备，重新启动LND并观察启动过程以查看钱包是否自动解锁

  ```bash
  $ sudo systemctl daemon-reload
  $ sudo systemctl restart lnd
  ```

* 您可以通过登录到第二个会话并观察日志文件来观察LND如何启动并将钱包解锁：
  `$ sudo journalctl -u lnd -f`

------

<<返回：[奖励指南]（raspibolt_60_bonus.md）
