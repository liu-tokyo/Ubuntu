# Ubuntu如何配置SSH服务端

## 1. 目标分析

　`SSH` 为`Secure Shell`的缩写，由 `IETF` 的网络小组（`Network Working Group`）所制定；`SSH` 为建立在应用层基础上的安全协议。`SSH` 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 `SSH` 协议可以有效防止远程管理过程中的信息泄露问题。`SSH`最初是`UNIX`系统上的一个程序，后来又迅速扩展到其他操作平台。`SSH`在正确使用时可弥补网络中的漏洞。`SSH`客户端适用于多种平台。几乎所有`UNIX`平台—包括`HP-UX`、`Linux`、`AIX`、`Solaris`、`DigitalUNIX`、`Irix`...以及其他平台，都可运行`SSH`。  
　现在我们知道`SSH`其实最早是一个安全协议，这个协议就是为远程登录会话而生。因为这个协议，衍生的的SSH程序是我们要安装的软件。要利用`SSH`连接两台机器，可想可知一个要作为客户端访问，另外一台要作为服务端提供被访问的服务。我们也知道很多`Linux`、`OSX`等类`Unix`系统都已自带了`SSH`的客户端，`Windows`也有不少类似`PuTTY`这样优秀的软件。
So，我们接下来要做的其实是如何安装并开启`SSH Server`。

- 服务器端：`Ubuntu 22.04`
- 客户端：`Debian 11.4`

## 2. 安装步骤

- 确定`Ubuntu`是否安装`SSH`服务

  ```bash
  systemctl status ssh
  ```

  如果尚未安装，显示如下信息：

  ```bash
  test@test-virtual-machine:~$ systemctl status ssh
  Unit ssh.service could not be found.
  ```

- 安装`SSH Server`

  ```bash
  sudo apt install openssh-server
  ```

  确定`Ubuntu SSH`服务状态：

  ```bash
  systemctl status ssh
  ```

  执行结果如下：

  ```bash
  test@test-virtual-machine:~$ systemctl status ssh
  ● ssh.service - OpenBSD Secure Shell server
       Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
       Active: active (running) since Mon 2022-09-05 20:28:51 JST; 46s ago
         Docs: man:sshd(8)
               man:sshd_config(5)
     Main PID: 2350 (sshd)
        Tasks: 1 (limit: 9416)
       Memory: 1.7M
          CPU: 20ms
       CGroup: /system.slice/ssh.service
               └─2350 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
  
   9月 05 20:28:51 test-virtual-machine systemd[1]: Starting OpenBSD Secure Shell server...
   9月 05 20:28:51 test-virtual-machine sshd[2350]: Server listening on 0.0.0.0 port 22.
   9月 05 20:28:51 test-virtual-machine sshd[2350]: Server listening on :: port 22.
   9月 05 20:28:51 test-virtual-machine systemd[1]: Started OpenBSD Secure Shell server.
  ```

  现在能看到：

  * 第一行加载状态，已加载`ssh.service`文件；
  * 第二行是否活动，正在运行；并且留意到一个守护进程`sshd`；
  * 再往下看到监听的端口是`22`。

  至此，我们的`SSH`服务端已经安装启动OK。

- 确定`Ubuntu`机器的`IP`：

  ```bash
  ifcofnig 
  ```

  执行不正常的话，需要安装相关组件：

  ```bash
  sudo apt install net-tools
  ```

  其实输入下面的指令更加直接（以下所有指令都可以）：

  ```bash
  ip a
  ip addr
  ip address
  ip address show
  ```

  IP地址信息如下：

  ```bash
  test@test-virtual-machine:~$ ifconfig
  ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 192.168.114.155  netmask 255.255.255.0  broadcast 192.168.114.255
          inet6 fe80::5691:40fe:6841:5989  prefixlen 64  scopeid 0x20<link>
          ether 00:0c:29:dc:fd:ef  txqueuelen 1000  (以太网)
          RX packets 52  bytes 14266 (14.2 KB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 104  bytes 12853 (12.8 KB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
          inet 127.0.0.1  netmask 255.0.0.0
          inet6 ::1  prefixlen 128  scopeid 0x10<host>
          loop  txqueuelen 1000  (本地环回)
          RX packets 137  bytes 12493 (12.4 KB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 137  bytes 12493 (12.4 KB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  ```

- 确定是否可访问Ubuntu

  ```bash
  ping 192.168.114.155
  ```

## 3. SSH登录

### 3.1 Linux-Ubuntu

- 使用账号密码登录`Ubuntu`

  `Debian11.4`自带客户端，直接输入如下指令即可：

  ```bash
  ssh test@192.168.114.155 -p 22
  ```

  对于第一次登陆的会提示主机不被认可，加密签名的指纹。输入 `yes` 确认，继续...  
  需要验证 `Ubuntu` 当前用户 `test` 的登录密码，输入确认即可登录成功。

  具体信息如下：

  ```bash
  liu@debian:~$ ssh test@192.168.114.155
  The authenticity of host '192.168.114.155 (192.168.114.155)' can't be established.
  ECDSA key fingerprint is SHA256:1cLNv//gj+hd1uGvw5qtuETEA8mOZYI94FsiiYDM+4s.
  Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
  Warning: Permanently added '192.168.114.155' (ECDSA) to the list of known hosts.
  test@192.168.114.155's password: 
  Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-47-generic x86_64)
   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  0 更新可以立即应用。
  The programs included with the Ubuntu system are free software;
  the exact distribution terms for each program are described in the
  individual files in /usr/share/doc/*/copyright.
  Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
  applicable law.
  test@test-virtual-machine:~$ 
  ```

- 简单测试

  正常登录之后，可以输入指令进行测试：

  ```bash
  test@test-virtual-machine:~$ sudo apt update
  [sudo] test 的密码： 
  命中:1 http://jp.archive.ubuntu.com/ubuntu jammy InRelease
  命中:2 http://jp.archive.ubuntu.com/ubuntu jammy-updates InRelease
  命中:3 http://jp.archive.ubuntu.com/ubuntu jammy-backports InRelease
  获取:4 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
  已下载 110 kB，耗时 2秒 (68.0 kB/s)
  正在读取软件包列表... 完成
  正在分析软件包的依赖关系树... 完成
  正在读取状态信息... 完成                 
  有 3 个软件包可以升级。请执行 ‘apt list --upgradable’ 来查看它们。
  ```

- 退出登录

  输入 `exit` 指令，退出登录：

  ```bash
  test@test-virtual-machine:~$ exit
  注销
  Connection to 192.168.114.155 closed.
  liu@debian:~$ 
  ```



### 3.2 Windows

本人电脑安装了 Git 客户端，所以带有 SSH 客户端应用。

- 打开 命令提示符，输入如下指令：

  ```bash
  C:\Users\liu>ssh test@192.168.114.155 -p 22
  The authenticity of host '192.168.114.155 (192.168.114.155)' can't be established.
  ECDSA key fingerprint is SHA256:1cLNv//gj+hd1uGvw5qtuETEA8mOZYI94FsiiYDM+4s.
  Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
  Warning: Permanently added '192.168.114.155' (ECDSA) to the list of known hosts.
  test@192.168.114.155's password:
  Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-47-generic x86_64)
   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  0 更新可以立即应用。
  Last login: Mon Sep  5 20:51:00 2022 from 192.168.114.1
  test@test-virtual-machine:~$
  ```

  和 Debian 一样，很顺利就能连接到 SSH 服务器，后续步骤不在赘述！

### 3.3 VirtualBox的端口映射

VirtualBox 虚拟机的话，端口不能直接从宿主机访问，需要映射端口到宿主机。

- `设置` → `网络` → `网卡1` ，展开 `高级` 项目，点击 `转发端口` 按钮进行设置。

  本人测试，是把 22 端口映射为 宿主机的 9022 端口，所以从宿主机的命令提示符链接的话，指令如下：

  ```bash
  ssh vip@localhost -p 9022
  ```

  ※ 用户名为 `vip`

## 4. 排除故障 - 网络

- 若不能访问，检查Ubuntu防火墙：

  ```bash
  # 检查防火墙
  sudo ufw status verbose
  
  # 开启防火墙
  sudo ufw enable
  
  # 开放端口
  sudo ufw allow 22
  ```

  如下命令，可以设置开放来自某IP访问某端口的权限：

  ```bash
  sudo ufw allow from your_ip to any port 22
  ```


## 5. 免密登录

　SSH 为 Secure Shell 的缩写，由 IETF 的网络工作小组（Network Working Group）所制定；SSH 为建立在应用层和传输层基础上的安全协议。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。SSH最初是UNIX系统上的一个程序，后来又迅速扩展到其他操作平台。SSH在正确使用时可弥补网络中 的漏洞。SSH客户端适用于多种平台。  
从客户端来看，SSH提供两种级别的安全验证:

- 基于口令的安全验证
- 基于密匙的安全验证

### 5.1 基于口令的安全验证

只要你知道自己帐号和口令，就可以登录到远程主机。所有传输的数据都会被加密， 但是不能保证你正在连接的服务器就是你想连接的服务器。

这个过程如下：

1. 远程主机收到用户的登录请求，把自己的公钥发给用户。

2. 用户使用这个公 钥，将登录密码加密后，发送回来。

3. 远程主机用自己的私钥，解密登录密码，如果密码正确，就同意用户登录。这种方式可能会有别的服务器在冒充真正的服 务器，将公钥发送给客户端，客户端就会将密码加密后发送给冒充的服务器，冒充的服务器就可以拿自己的私钥获取到密码，也就是受到“中间人”这种方式的攻 击。 

值得一说的是当第一次链接远程主机时，会提示您当前主机的”公钥指纹”，询问您是否继续，如果选择继续后就可以输入密码进行登录了，当远程的主机接受以后，该台服务器的公钥就会保存到`~/.ssh/known_hosts`文件中。

### 5.2 基于密匙的安全验证

需要依靠密匙，也就是你必须为自己创建一对密匙，并把公用密匙放在需要访问的服务器上。如果你要连接到SSH服务器上，客户端软件就会向服务器发出请求，请求用你的密匙进行安全验证。服务器收到请求之后，先在该服务器上你的主目录下寻找你的公用密匙，然后把它和你发送过来的公用密匙进行比较。如果两个密匙一致，服务器就用公用密匙加密“质询”并把它发送给客户端软件。客户端软件收到“质询”之后就可以用你的私人密匙解密再把它发送给服务器。用这种方式，你必须知道自己密匙的口令。但是，与第一种级别相比，第二种级别不需要在网络上传送口令。第二种级别不仅加密所有传送的数据，而且“中间人”这种攻击 方式也是不可能的（因为他没有你的私人密匙）。但是整个登录的过程可能需要`10秒`，但是相比输入密码的方式来说`10秒`也不长。

那么如何生成自己的一对密钥呢？

- 打开终端执行 `ssh-keygen`：

  ```bash
  C:\Users\liu>ssh-keygen -t rsa
  Generating public/private rsa key pair.
  Enter file in which to save the key (C:\Users\liu/.ssh/id_rsa):
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:
  Your identification has been saved in C:\Users\liu/.ssh/id_rsa.
  Your public key has been saved in C:\Users\liu/.ssh/id_rsa.pub.
  The key fingerprint is:
  SHA256:5uKV+6ZtcMYsb3KGk3yf77h8GHwf6HNDIehJM+BQ/OM liu@DESKTOP-MEOFCIG
  The key's randomart image is:
  +---[RSA 3072]----+
  |        o.       |
  |       . o       |
  |        o o .    |
  |         . B . . |
  |        So+.= o .|
  |       oo.=Eo..o |
  |      ..+O  .+...|
  |     . o*oO..=.o.|
  |      . o%o.*== .|
  +----[SHA256]-----+
  ```

  该命令会在`~/.ssh/`目录下创建`id_rsa`、`id_rsa.pub`两个文件，分别为您的公钥和私钥。

将公钥拷贝到服务器的`~/.ssh/authorized_keys`文件中就可以了。  
拷贝方法有如下几种：

1. 将公钥通过`scp`拷贝到服务器上，然后追加到 `~/.ssh/authorized_keys` 文件中，这种方式比较麻烦。`scp -P 22 ~/.ssh/id_rsa.pub user@host:~/`。
2. 通过`ssh-copyid`程序，`ssh-copyid user@host`即可，但是这种方式不支持更改端口号（我没找到）。该程序`ubuntu`系统自带无需安装，其实该程序为一个脚本。
3. 可以通过`cat ~/.ssh/id_rsa.pub | ssh -p 22 user@host 'cat >> ~/.ssh/authorized_keys'`，这个也是我比较常用的方法，因为可以更改端口号。

当然还有很多其它办法，再次不做赘述！

### 5.3 Config编辑

- 文件内容：

  ```yaml
  Host my-test
      HostName 192.168.114.155
      Port 22
      User test
      IdentityFile C:\Users\liu\.ssh\id_ed25519
      ServerAliveInterval 30
      TCPKeepAlive yes
  ```

- 项目说明：

  | 项目                | 说明                                  | 备考 |
  | ------------------- | ------------------------------------- | ---- |
  | Host my-test        | 登录的服务器别名 ssh my-test 就可以了 | -    |
  | HostName            | 要登录的服务器ip                      | -    |
  | Port 22             | SSH服务器端口号：默认22               | 22   |
  | User test           | 登录名                                | -    |
  | IdentityFile        | 私钥路径                              | -    |
  | ServerAliveInterval |                                       | 30   |
  | TCPKeepAlive        |                                       | yes  |


## 6. 文件传输

- 上传文件：

  ```bash
  ## 公钥上传
  # Linux
  scp -P 22 ~/.ssh/id_rsa.pub test@192.168.114.155:~/
  # Windows
  scp -P 22 C:\Users\liu\.ssh/id_rsa.pub test@192.168.114.155:~/
  ## 私钥上传
  # Linux
  scp -P 22 ~/.ssh/id_rsa test@192.168.114.155:~/
  # Windows
  scp -P 22 C:\Users\liu\.ssh/id_ed25519 test@192.168.114.155:~/
  ```

  把 `id_rsa.pub` 上传到SSH服务器的`test`用户的主目录下面。

- 下载文件：

  ```bash
  scp -P 22 test@192.168.114.155:~/ssh_host_ed25519_key C:\Users\liu\.ssh/ssh_host_ed25519_key
  ```

  把 `SSH` 服务器的 `ssh_host_ed25519_key` 文件，下载到本地。

## 7. 配置服务器

- 编辑 `/etc/ssh` 目录下的 `sshd_config` 文件：

  ```bash
  sudo nano /etc/ssh/sshd_config
  ```

  | 项目                   | 说明                                                         | 备注 |
  | ---------------------- | ------------------------------------------------------------ | ---- |
  | ssh_host_rsa_key       | 指的是以RSA加密的私钥文件。                                  |      |
  | ssh_host_ecdsa_key     | 指的是以ECDSA加密的私钥文件。                                |      |
  | PermitRootLogin        | 默认值是yes，允许root用户通过SSH登录，如果没有需要，这里可以不用管，具体可参考：[禁止root用户远程登录](https://www.fujieace.com/linux/ssh-permitrootlogin-no.html)，我这里就先不禁止了。 |      |
  | RSAAuthentication      | 默认屏蔽，RSA密钥认证，由于这里我们想设置为ssh密钥登录，这里必须设置为yes。 |      |
  | PubkeyAuthentication   | 默认屏蔽，公钥认证，由于这里我们想设置为ssh密钥登录，这里必须设置为yes。 |      |
  | AuthorizedKeysFile     | 放公钥文件的地方。                                           |      |
  | PasswordAuthentication | 值是yes，允许SSH用密码登录。由于这里我们想设置为ssh密钥登录，因此，需要将PasswordAuthentication设置为no。 |      |

- 编辑 `authorized_keys`：

  ```bash
  sudo mkdir ~/.ssh/
  sudo nano ~/.ssh/authorized_keys
  
  ## 权限变更（锁定）
  sudo chmod 600 ~/.ssh/authorized_keys
  sudo chmod 700 ~/.ssh
  ## 权限变更（开放）
  sudo chmod 777 ~/.ssh/authorized_keys
  sudo chmod 777 ~/.ssh
  ```

- 重置SSH服务器

  ```bash
  systemctl restart sshd
  ```

  或

  ```bash
  /etc/rc.d/init.d/sshd restart
  ```

  私钥的操作权限需要设置为 `600`，否则重置SSH服务器失败。

## 8. 解除私钥密码

- 制作密钥对：

  ```bash
  ssh-keygen -t rsa -b 4096
  ```

  其它：

  ```bash
  ssh-keygen -t dsa /etc/ssh/ssh_host_dsa_key
  ssh-keygen -t ecdsa /etc/ssh/ssh_host_ecdsa_key
  ssh-keygen -t ed25519 /etc/ssh/ssh_host_ed25519_key
  ssh-keygen -t rsa /etc/ssh/ssh_host_rsa_key
  ssh-keygen -t rsa1 /etc/ssh/ssh_host_rsa1_key
  ```

- 解除私钥密码：

  ```bash
  openssl rsa -in ~/.ssh/id_rsa -out ~/.ssh/id_rsa
  ```


## 附1. 安装PowerShell

命令提示符 对部分指令的识别还是有问题的，例如： `~/.ssh/`

- 命令提示符 输入如下指令：

  ```bash
  C:\>cd ~/.ssh/
  The system cannot find the path specified.
  ```

  无法找到该路径，其实这是一个类 `Linux` 的路径。

- PowerShell 输入如下指令：

  ```bash
  PS C:\> cd ~/.ssh/
  PS C:\Users\liu\.ssh>
  ```

  能够正确找到该路径，`PowerShell` 可以执行类 `Linux` 指令。对习惯 `Linux` 用户非常友好。

- 安装 PowerShell 

  输入 `winget install Microsoft.PowerShell`，信息如下，就可以安装 `PowerShell`。

  ```shell
  C:\>winget install Microsoft.PowerShell
  已找到 PowerShell [Microsoft.PowerShell] 版本 7.2.6.0
  此应用程序由其所有者授权给你。
  Microsoft 对第三方程序包概不负责，也不向第三方程序包授予任何许可证。
  Downloading https://github.com/PowerShell/PowerShell/releases/download/v7.2.6/PowerShell-7.2.6-win-x64.msi
    ██████████████████████████████   102 MB /  102 MB
  已成功验证安装程序哈希
  正在启动程序包安装...
  已成功安装
  ```

  