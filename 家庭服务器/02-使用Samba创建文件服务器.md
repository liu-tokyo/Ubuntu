## 使用Samba创建文件服务器

在 Ubuntu 上搭建 **Samba** 是实现“全家桶”互联最明智的选择。它能让您的 Windows 像访问本地磁盘（D 盘、E 盘）一样访问 Ubuntu 上的 GitLab 仓库或公共文件夹。

以下是为您定制的快速配置指南：

### 1. Ubuntu 端：安装并创建共享

打开终端，执行以下步骤：

**第一步：安装 Samba**

```bash
sudo apt update
sudo apt install samba
```

**第二步：创建共享目录并设置权限** 假设您想共享 `/home/username/myshare` 这个文件夹：

```Bash
mkdir -p ~/myshare
chmod 777 ~/myshare
```

**第三步：添加共享用户** Samba 需要独立的密码。建议使用您现有的 Ubuntu 用户名（例如 `pi`）：

```Bash
sudo smbpasswd -a [您的用户名]
# 然后按提示输入两次密码，这个密码是给 Windows 登录用的
```

------

### 2. 修改配置文件

编辑 Samba 的配置文件：

```bash
sudo nano /etc/samba/smb.conf
```

在文件末尾添加以下内容：

```ini
[HomeShare]
   comment = Ubuntu Shared Folder
   path = /home/[您的用户名]/myshare
   browseable = yes
   read only = no
   guest ok = no
   create mask = 0777
   directory mask = 0777
   valid users = [您的用户名]
```

*按 `Ctrl + O` 保存，`Ctrl + X` 退出。*

**最后重启服务：**

```bash
sudo systemctl restart smbd
```

------

### 3. Windows 25H2 端：挂载网络驱动器

在您的 Windows 25H2 电脑上，操作非常丝滑：

1. 打开 **文件资源管理器**。
2. 在地址栏直接输入：`\\Ubuntu的IP地址\HomeShare`（例如 `\\192.168.1.10\HomeShare`）。
3. 在弹出的对话框中，输入您刚才在 **第三步** 设置的用户名和密码。
4. **进阶操作：** 点击顶部的“**映射网络驱动器**”，给它分配一个盘符（比如 **Z: 盘**）。

------

### 4. 25H2 的特殊注意事项

由于 Windows 11 25H2 的安全性很高，如果您连接失败，请检查以下两点：

- **SMB 1.0 协议：** 25H2 默认禁用了极不安全的 SMB 1.0。好在按照上面的配置，Ubuntu 默认使用 SMB 3.0，所以**不需要**在 Windows 功能里开启 SMB 1.0（开启反而不安全）。
- **网络配置文件：** 确保您的 Windows 25H2 网络类型设置为“**专用（Private）**”而非“公用（Public）”，否则防火墙可能会拦截文件共享请求。

------

### 💡 针对您家庭多台电脑的“终极建议”

既然您有 多台笔记本，每台都输一遍 IP 地址太麻烦了。

- **固定 IP：** 请在您的路由器后台，把那台 Ubuntu 的 IP 地址根据 MAC 地址**静态绑定**。
- **统一盘符：** 建议把所有 Windows 笔记本的共享盘全部挂载为同一个盘符（例如 `S:` 盘，代表 Storage）。这样无论您坐在哪台电脑前，路径都是统一的。