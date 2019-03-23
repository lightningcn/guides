[[Intro]（README.md）]  -  [[Preparations]（raspibolt_10_preparations.md）]  -  [** Raspberry Pi **]  -  [[比特币]（raspibolt_30_bitcoin.md）]  -  [[闪电] （raspibolt_40_lnd.md）]  -  [[Mainnet]（raspibolt_50_mainnet.md）]  -  [[Bonus]（raspibolt_60_bonus.md）]  -  [[故障排除]（raspibolt_70_troubleshooting.md）]

-------
### Raspberry Pi上的️⚡Lightning️⚡初学者指南
--------

# 覆盆子皮

## 记下你的密码
您将需要几个密码，我发现最简单的方法是在开始时将它们全部写下来，而不是在整个指南中碰到它们。它们应该是唯一且非常安全的，至少12个字符。不要使用不常见的特殊字符**，空格或引号（'或“）。
```
[A]主用户密码
[B]比特币RPC密码
[C] LND钱包密码
[D] LND种子密码（可选）
```
！[xkcd：密码强度]（images / 20_xkcd_password_strength.png）

如果您需要灵感来创建密码：[xkcd：密码强度]（https://xkcd.com/936/）漫画很有趣并且包含很多真相。将密码的副本存储在安全的地方（最好是密码管理器，如[KeePass]（https://keepass.info/）），并在系统启动并运行后保持原始注释不在视线范围内。

## 准备操作系统
该节点无头运行，这意味着没有键盘或显示器，因此使用操作系统Raspbian Stretch Lite。

1. 下载[Raspbian Stretch Lite]（https://www.raspberrypi.org/downloads/raspbian/）磁盘映像
2. 使用[本指南]将磁盘映像写入SD卡（https://www.raspberrypi.org/documentation/installation/installing-images/README.md）

### 启用Secure Shell
没有键盘或屏幕，在初始设置期间不能与Pi直接交互。将图像写入Micro SD卡后，在卡的主目录中创建一个名为“ssh”（无扩展名）的空文件。这会导致从一开始就启用Secure Shell（ssh），我们将能够远程登录。

* 在MicroSD卡的启动分区中创建一个文件`ssh`

### 准备Wifi
我不推荐它，但您可以通过无线网络连接运行RaspiBolt。为避免使用网络电缆进行初始设置，您可以预先配置无线设置：

* 在MicroSD卡上创建一个文件`wpa_supplicant.conf`，其中包含以下内容。请注意，网络名称（ssid）和密码必须是双引号（如`psk =“password”`）
```
country=[COUNTRY_CODE]
ctrl_interface=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="[WIFI_SSID]"
    psk="[WIFI_PASSWORD]"
}
```
* 将`[COUNTRY_CODE]`替换为您所在国家/地区的[ISO2代码]（https://www.iso.org/obp/ui/#search）（例如``US`）
* 用您自己的WiFi的凭据替换`[WIFI_SSID]`和`[WIFI_PASSWORD]`。


### 开始你的Pi
* 安全地从计算机中弹出SD卡
* 将SD卡插入Pi
* 如果您尚未设置Wifi：使用以太网电缆将Pi连接到您的网络
* 通过使用Micro USB线将其连接到手机充电器来启动Pi

## 连接到网络
Pi正在启动并从您的家庭网络获取新地址。此地址可能会随时间而变化。为了使Pi可以从互联网上访问，我们为它分配一个固定的地址。

### 访问您的路由器
固定地址在网络路由器中配置：可以是电缆调制解调器或Wifi接入点。所以我们首先需要访问路由器。要找出它的地址，

* 在连接到家庭网络的计算机上启动命令提示符（在Windows中，单击“开始”菜单并直接键入cmd或在搜索框中键入，然后按Enter键）
* 在Mac / Linux上输入命令`ipconfig`（或`ifconfig`）
* 寻找“默认网关”并记下地址（例如“192.168.0.1”）

：point_right：其他信息：[访问路由器]（http://www.noip.com/support/knowledgebase/finding-your-default-gateway/）。

现在打开您的Web浏览器并通过输入地址访问您的路由器，就像常规Web地址一样。您需要这样登录，现在您可以查找家庭网络中的所有网络客户端。其中一个应该被列为“raspberrypi”，以及它的地址（例如“192.168.0.240”）。

！[路由器客户端列表]（images / 20_net1_clientlist.png）

：point_right：不知道你的路由器密码？试试[routerpasswords.com]（http://www.routerpasswords.com/）。
：警告：如果您的路由器仍然使用初始密码：更改它！

### 设置固定地址

我们现在需要为Pi设置固定（静态）IP地址。通常，您可以在“DHCP服务器”下找到此设置。手动地址应与当前地址相同，只需将最后一部分更改为较低的数字（例如192.168.0.240→192.168.0.20）。

：point_right：需要其他信息吗？谷歌“[你的路由器品牌]配置静态dhcp ip地址”

### 端口转发/ UPnP
接下来，需要配置“端口转发”。不同的应用程序使用不同的网络端口，并且路由器需要知道特定端口的流量必须指向哪个内部网络设备。端口转发需要设置如下：

| 应用名称 | 外部端口 | 内部端口 | 内部IP地址 | 协议（TCP或UDP） |
| ---------------- | ------------- | ------------- | ------------------- | --------------------- |
| 比特币          | 8333          | 8333          | 192.168.0.20        | 都                  |
| 比特币测试     | 18333         | 18333         | 192.168.0.20        | 都                  |

：point_right：其他信息：[设置端口转发]（https://www.noip.com/support/knowledgebase/general-port-forwarding-guide/）。

Lightning网络守护进程（LND）支持** UPnP **自动配置端口转发，并将自己的外部IP地址通告给网络。

* 为您的路由器启用UPnP。

：point_right：如果您不确定如何，请搜索[“启用upnp路由器MY-ROUTER-MODEL”]（https://duckduckgo.com/?q=enable+upnp+router+MY-ROUTER-MODEL）拥有路由器模型。

保存并应用这些路由器设置，我们稍后会检查它们。断开Pi与电源的连接，等待几秒钟，然后重新插入。该节点现在应该获得新的固定IP地址。

！[固定网络地址]（images / 20_net2_fixedip.png）

## 在Raspberry Pi上工作
### 命令行简介
我们将在Pi的命令行上工作，这对您来说可能是新手。在下面找到一些基本信息，它将帮助您导航并与您的Pi进行交互。

#### 输入命令
您可以通过在命令下方打印结果来输入命令和Pi答案。为了清楚命令开始的位置，本指南中的每个命令都以`$`符号开头。系统响应标有`>`字符。

在以下示例中，只需输入`ls -la`并按Enter / return键：
```
$ ls -la
>示例系统响应
```
！[command ls -la]（images / 20_command_ls-la.png）

* **自动完成命令**：输入命令时，可以使用“Tab”键进行自动完成，例如。用于命令，目录或文件名。

* **命令历史记录**：通过按：arrow_up：和：arrow_down：在键盘上，您可以调用先前输入的命令。

* **常见Linux命令**：有关Linux命令的非常有选择性的参考列表，请参阅[FAQ]（raspibolt_faq.md）页面。

* **使用管理员权限**：我们的普通用户没有管理员权限。如果命令需要编辑系统配置，我们需要使用`sudo`（“superuser do”）命令作为前缀。我们不使用`nano / etc / fstab`编辑系统文件，而是使用`sudo nano / etc / fstab`。
  出于安全原因，用户“比特币”不能使用`sudo`命令。

* **使用Nano文本编辑器**：我们使用Nano编辑器创建新文本文件或编辑现有文本文件。它并不复杂，但保存和退出并不直观。
  * 保存：按“Ctrl-O”（输出），确认文件名，然后按“Enter”键
  * 退出：点击“Ctrl-X”

* **复制/粘贴**：如果您使用的是Windows和PuTTY SSH客户端，您可以通过鼠标选择文本（无需点击任何内容）从shell中复制文本，并用右侧粘贴光标位置 - 点击ssh窗口中的任意位置。

### 连接到Pi
现在是时候通过SSH连接到Pi并开始工作了。为此，需要安全Shell（SSH）客户端。安装，启动和连接：

- Windows：PuTTY（[网站]（https://www.putty.org））
- Mac OS：内置SSH客户端（参见[本文]（http://osxdaily.com/2017/04/28/howto-ssh-client-mac/））
- Linux：只使用本机命令，例如。 `ssh pi @ 192.168.0.20`
- 使用以下SSH连接设置：
  - 主机名：您在路由器中设置的静态地址，例如。 `192.168.0.20`
  - 港口：`22`
  - 用户名：`pi`
  - 密码：`raspberry`。

！[登录]（图像/ 20_login.png）

：point_right：其他信息：[使用SSH与Raspberry Pi]（https://www.raspberrypi.org/documentation/remote-access/ssh/README.md）

### Raspi-配置
您现在位于自己的比特币节点的命令行上。首先我们完成Pi配置。输入以下命令：
`$ sudo raspi-config`

！[raspi-CONFIG]（图像/ 20_raspi-config.png）

* 首先，在`1`上将密码更改为您的`密码[A]`。
* 接下来，选择Update`8`以获取最新的配置工具
* 网络选项`2`：
  * 你可以给你的节点一个可爱的名字（如“RaspiBolt”）和
  * 配置您的Wifi连接（仅限Pi 3）
* 引导选项`3`：
  * 选择`Desktop / CLI`→`Console`和
  * `在启动时等待网络
* 本地化`4`：设置你的时区
* 高级`7`：运行`Expand Filesystem`并将`Memory Split`设置为16
* 选择“<Finish>”和“<No>”退出，因为不需要重启

### 软件更新
使用安全补丁和应用程序更新使系统保持最新非常重要。 “高级包装工具”（apt）使这一切变得简单：
`$ sudo apt-get update`
`$ sudo apt-get upgrade`

：point_right：每隔几个月定期执行此操作以获取与安全相关的更新。

确保安装了所有必需的软件包：
  `$ sudo apt-get install htop git curl bash-completion jq dphys-swapfile`

### 添加主用户“admin”
本指南使用主用户“admin”而不是“pi”使其更易于与其他平台重复使用。

* 创建新用户，设置密码[A]并将其添加到组“sudo”
  `$ sudo adduser admin`
  `$ sudo adduser admin sudo`
* 在您使用它时，将“root”管理员用户的密码更改为您的密码[A]。
  `$ sudo passwd root`
* 重新启动并使用新用户“admin”登录
  `$ sudo shutdown -r now`

### 添加服务用户“比特币”
比特币和闪电进程将在后台运行（作为“守护进程”）并出于安全原因使用单独的用户“比特币”。此用户没有管理员权限，无法更改系统配置。

* 使用命令`sudo`时，系统会提示您不时输入管理员密码以提高安全性。
* 输入以下命令，设置`password [A]`并使用enter / return键确认所有问题。
  `$ sudo adduser bitcoin`

### 安装外部硬盘
要存储区块链，我们需要大量空间。作为服务器安装，Linux本机文件系统Ext4是外部硬盘的最佳选择，因此我们将格式化硬盘，删除所有以前的数据。然后将外部硬盘连接到文件系统，并可以作为常规文件夹进行访问（这称为“安装”）。

：警告：**此硬盘上的先前数据将被删除！**

* 将硬盘插入正在运行的Pi中并为驱动器供电。

* 获取外部硬盘上主分区的NAME
  `$ lsblk -o UUID，NAME，FSTYPE，SIZE，LABEL，MODEL`

* 使用Ext4格式化外部硬盘（使用上面的[NAME]，例如`/ dev / sda1`）
  `$ sudo mkfs.ext4 / dev / [NAME]`

* 将由此format命令提供的UUID复制到本地（Windows）记事本。

* 在最后编辑fstab文件和以下作为新行（替换`UUID = 123456`）
  `$ sudo nano / etc / fstab`
  `UUID = 123456 / mnt / hdd ext4 noexec，默认为0 0`

* 创建目录以添加硬盘并设置正确的所有者
  `$ sudo mkdir / mnt / hdd`

* 挂载所有驱动器并检查文件系统。是“/ mnt / hdd”列出的吗？
  `$ sudo mount -a`
  `$ df / mnt / hdd`
```
文件系统1K块使用可用使用％挂载
/ dev / sda1 479667880 73756 455158568 1％/ mnt / hdd
```
*  设置所有者
  `$ sudo chown -R bitcoin：bitcoin / mnt / hdd /`

* 切换到用户“比特币”，导航到硬盘并创建比特币目录。
  `$ sudo su  - 比特币`
  `$ cd / mnt / hdd`
  `$ mkdir比特币
  `$ ls -la`

* 在新目录中创建一个testfile并将其删除。
  `$ touch bitcoin / test.file`
  `$ rm bitcoin / test.file`

* 退出“比特币”用户会话
  `$ exit`

如果此命令出错，则可能是外部硬盘安装为“只读”。这必须在继续之前修复。如果无法修复，请考虑重新格式化外部硬盘。

👉其他信息：[外部存储配置]（https://www.raspberrypi.org/documentation/configuration/external-storage.md）

### 移动交换文件

使用交换文件会很快降低SD卡的性能。因此，我们将它移动到外部硬盘。

* 作为用户“admin”，删除旧的交换文件
  `$ sudo dphys-swapfile swapoff`
  `$ sudo dphys-swapfile uninstall`

* 编辑配置文件并使用以下条目替换现有条目。保存并退出。
  `$ sudo nano / etc / dphys-swapfile`

```
CONF_SWAPFILE = / MNT / HDD /交换文件

#comment或删除CONF_SWAPSIZE行。然后它将动态创建
＃CONF_SWAPSIZE =
```

* 手动创建新的交换文件
  `$ sudo dd if = / dev / zero of = / mnt / hdd / swapfile count = 1000 bs = 1MiB`
  `$ sudo chmod 600 / mnt / hdd / swapfile`
  `$ sudo mkswap / mnt / hdd / swapfile`

* 启用新的交换配置
  `$ sudo dphys-swapfile setup`
  `$ sudo dphys-swapfile swapon`

## 强化你的Pi

以下步骤需要管理员权限，必须与用户“admin”一起执行。

### 启用简单的防火墙
Pi将从互联网上可见，因此需要抵御攻击。防火墙控制允许的流量并关闭可能的安全漏洞。

下面的'ufw允许来自192.168.0.0 / 24 ......`假设您的Pi的IP地址类似于`192.168.0。???`，???如果您的IP地址是“12.34.56.78”，则必须将此行调整为“ufw allow from 12.34.56.0 / 24 ...”。
```
$ sudo apt-get install ufw
$ sudo su
$ ufw默认否认传入
$ ufw默认允许传出
$ ufw允许从192.168.0.0/24到任何端口22评论'允许来自本地LAN的SSH'
$ ufw允许proto udp从192.168.0.0/24端口1900到任何评论'允许本地LAN SSDP用于UPnP发现'
$ ufw允许9735评论'允许闪电'
$ ufw允许8333评论'允许比特币主网'
$ ufw允许18333评论'允许比特币测试网'
$ ufw启用
$ systemctl enable ufw
$ ufw状态
$退出
```
！[UFW状态]（images / 20_ufw_status.png）

：point_right：其他信息：[UFW Essentials]（https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands）

：point_right：如果您发现自己被错误锁定，可以将键盘和屏幕连接到Pi以在本地登录并修复这些设置（尤其是SSH端口22）。

### 的fail2ban
必须特别保护对Pi的SSH登录。防火墙阻止来自网络外部的所有登录尝试，但应采取其他措施来防止攻击者（可能来自网络内部）尝试所有可能的密码。

第一个措施是安装“fail2ban”，这项服务可以在10分钟内完成5次失败登录尝试，从而切断任何系统。这使得蛮力攻击变得不可行，因为它只需要太长时间。

！[的fail2ban]（图像/ 20_fail2ban.png）
*我通过输入错误的密码锁定自己*：wink：

`$ sudo apt-get install fail2ban`

初始配置应该没问题，因为默认情况下启用SSH。如果你想深入了解，你可以
：point_right：[自定义配置]（https://linode.com/docs/security/using-fail2ban-for-security/）。

### 使用SSH密钥登录
保护SSH登录的最佳选择之一是完全禁用密码登录并需要SSH密钥证书。只有拥有私钥的人才能登录。

** _为“admin”用户设置SSH密钥：_ **

**对于Windows用户**：[在Linux服务器上使用PuTTY配置“无密码SSH密钥验证”]（https://www.tecmint.com/ssh-passwordless-login-with-putty）

* 您应该已经生成了三个新文件。保证他们安全！

！[SSH密钥文件]（images / 20_ssh_keys_filelist.png）

**对于Mac / Linux用户：**

* 回到您正在处理的机器上（确保这种情况的一种简单方法是在终端窗口中打开一个新选项卡）我们首先需要检查现有的私钥/公钥对：

   `$ ls -la~ / .ssh / * .pub`

* 如果列出了文件，则您的公钥应为以下文件之一（默认情况下）：

   ```
   id_dsa.pub
   id_ecdsa.pub
   id_ed25519.pub
   id_rsa.pub
   ```
* 如果存在其中一个文件，请跳到下面的“让我们确定......”项目符号。如果这些文件都不存在，或者您得到“没有这样的文件或目录”，请执行以下操作以创建新对：

   `$ ssh-keygen -t rsa -b 4096`
   * 当系统提示您“输入要保存密钥的文件”时，按Enter键。这接受默认文件位置。

   * 接下来，要强制执行密钥安全性，请使用密码[A]来保护密钥。再次输入以确认。

* 让我们确保Raspberry pi上存在`〜/ .ssh`目录（确保将Raspberry Pi的IP替换为下面的RASPBERRY_PI_IP）：

   `ssh admin @ RASPBERRY_PI_IP'mkdir -p~ / .ssh'`

* 将您的公钥复制到Raspberry Pi并设置.ssh目录的文件模式（同样，在下面替换您的Pi的IP for RASPBERRY_PI_IP）。如果您的公钥文件不是`id_rsa.pub`，请替换下面的文件名：

   `$ cat~ / .ssh / id_rsa.pub | ssh admin @ RASPBERRY_PI_IP'cat >>〜/ .ssh / authorized_keys && chmod -R 700~ / .ssh /'`

**一旦Raspberry Pi拥有您的公钥副本，我们现在将禁用密码登录：**

* 使用SSH密钥以“admin”身份登录Raspberry Pi（不应再提示您输入管理员密码）。

* 编辑ssh配置文件
`$ sudo nano / etc / ssh / sshd_config`

* 将设置“ChallengeResponseAuthentication”和“PasswordAuthentication”更改为“no”（如果需要，通过删除＃取消注释该行）
  ！[SSH配置]（images / 20_ssh_config.png）

* 保存配置文件并退出

* 复制用户“root”的SSH公钥，以防万一
  `$ sudo mkdir / root / .ssh`
  `$ sudo cp /home/admin/.ssh/authorized_keys / root / .ssh /`
  `$ sudo chown -R root：root / root / .ssh /`
  `$ sudo chmod -R 700 / root / .ssh /`
  `$ sudo systemctl restart ssh`

* 退出并重新登录。您无法再使用“pi”或“bitcoin”登录，只有“admin”和“root”具有必要的SSH密钥。
  `$ exit`

：警告：**备份你的SSH密钥！**如果丢失了，你需要在你的Pi上附加一个屏幕和键盘。

### 增加打开文件限制

如果您的RaspiBolt被互联网请求（由于DDoS攻击而诚实或恶意）淹没，您将很快遇到“无法接受连接：太多打开文件”错误。这是由于设置得太低的打开文件（表示单个tcp连接）的限制。

编辑以下三个文件，在结束注释之前添加其他行，保存并退出。

```
$ sudo nano /etc/security/limits.conf
* soft nofile 128000
* hard nofile 128000
root soft nofile 128000
root hard nofile 128000


```

！[编辑pam.d / limits.conf]（images / 20_nofile_limits.png）



```
$ sudo nano /etc/pam.d/common-session
会话需要pam_limits.so
```

！[编辑pam.d / common-session]（images / 20_nofile_common-session.png）



```
$ sudo nano /etc/pam.d/common-session-noninteractive
会话需要pam_limits.so
```

！[编辑pam.d / common-session-noninteractive]（images / 20_nofile_common-session-noninteractive.png）



---

下一篇：[比特币>>]（raspibolt_30_bitcoin.md）
