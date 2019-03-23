[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [[Raspberry Pi]（raspibolt_20_pi.md）]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [ [闪电]（raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[** Bonus **]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

------

### Raspberry Pi上的️⚡Lightning️⚡初学者指南

------

## 奖励指南：Pimp命令行
*难度：容易*

### 命令提示符
您可以通过启用颜色输出和设置自定义提示来为每个用户美化命令提示符。

* 打开并编辑`.bashrc`，如下所示，保存并退出
  `$ nano / home / admin / .bashrc`

```庆典
#enable color prompt（取消注释）
force_color_prompt = YES

#pimp prompt（替换PS1线）
PS1 =“$ {debian_chroot：+（$ debian_chroot）} \ [\ e [33m \] \ u \ [\ 033 [01; 34m \] \ w \ [\ e [33; 40m \]₿\ [\ e [m \]“

#set“ls”总是使用-la选项（在文件末尾插入）
alias ls ='ls -la --color = always'
```

！[皮条客提示]（images / 60_pimp_prompt.png）

* 重新加载配置
  `source / home / admin / .bashrc`

！[Pimped提示]（images / 60_pimp_prompt_result.png）

### Bash完成
作为用户“admin”，为比特币核心和所有Lightning项目安装bash完成脚本。然后按Tab键完成命令（例如bitcoin-cli getblockch [Tab]→bitcoin-cli getblockchaininfo）

```
$ cd / home / admin / download
$ wget https://raw.githubusercontent.com/bitcoin/bitcoin/master/contrib/bitcoin-cli.bash-completion
$ wget https://raw.githubusercontent.com/lightningnetwork/lnd/master/contrib/lncli.bash-completion
$ sudo cp * .bash-completion /etc/bash_completion.d/
```

下次登录后将启用Bash完成功能。

------

<<返回：[奖励指南]（raspibolt_60_bonus.md）
