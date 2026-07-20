## 如何禁止 Ubuntu 系统更新

对于内网文件服务器来说，“稳定压倒一切”确实是金科玉律。既然不需要联网更新，完全可以关闭 Ubuntu 的自动检查、下载和更新机制，防止它在后台偷偷占带宽或在关键时刻自动重启。

你可以按照以下三个步骤彻底封印更新功能：

### 1. 关闭自动更新服务

Ubuntu 默认开启了 `unattended-upgrades` 服务。这是“罪魁祸首”，需要先停掉它。

在终端执行：

```bash
sudo systemctl stop apt-daily.timer
sudo systemctl disable apt-daily.timer
sudo systemctl stop apt-daily-upgrade.timer
sudo systemctl disable apt-daily-upgrade.timer
```

直接编辑可能更为方便：

```bash
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

将里面的参数全部改为 `0`（关闭自动获取和安装）：

```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

一键手动更新（推荐）：

```
sudo apt update && sudo apt upgrade -y && sudo snap refresh
```



------

### 2. 修改配置文件彻底禁用

即使关了服务，有时软件包管理器的配置还是会尝试检查。我们需要修改 `/etc/apt/apt.conf.d/20auto-upgrades` 文件。

1. 使用编辑器打开文件： `sudo nano /etc/apt/apt.conf.d/20auto-upgrades`

2. 将所有值改为 `"0"`。修改后的内容应该如下：

   ```Plaintext
   APT::Periodic::Update-Package-Lists "0";
   APT::Periodic::Download-Upgradeable-Packages "0";
   APT::Periodic::AutocleanInterval "0";
   APT::Periodic::Unattended-Upgrade "0";
   ```
   
   *(如果没有这个文件，可以手动创建一个，内容如上即可。)*

------

### 3. 禁止系统级自动更新提示

为了保险，还可以修改另一个配置文件 `/etc/apt/apt.conf.d/10periodic`：

```bash
sudo nano /etc/apt/apt.conf.d/10periodic
```

同样确保里面关于 Update 的选项都是 `"0"`。

------

### 4. (进阶) 锁定当前内核版本

作为文件服务器，内核更新后的重启往往是最麻烦的。如果你想确保内核版本永远不动，可以锁定当前内核：

```bash
# 查看当前使用的内核
uname -r

# 锁定内核（将 linux-image-generic 替换为你实际的版本名）
sudo apt-mark hold linux-image-generic linux-headers-generic
```

------

### ⚠️ 温馨提示

- **物理隔离安全：** 既然不再更新，请确保该服务器**绝对不会**暴露在公网上。内网环境虽然相对安全，但老旧版本的服务（如 Samba 或 NFS）若存在已知漏洞，仍有风险。
- **手动维护：** 建议每半年或一年，在非业务繁忙期，由人工接入网络进行一次必要的手动更新（`sudo apt update && sudo apt upgrade`），然后再次锁定。



## 禁用系统更新的脚本

### 1. 制作脚本文件

你可以将以下内容保存为 `disable-updates.sh`。

```bash
#!/bin/bash

# 确保以 root 权限运行
if [ "$EUID" -ne 0 ]; then 
  echo "请使用 sudo 运行此脚本"
  exit
fi

echo "--- 开始禁用 Ubuntu 自动更新 ---"

# 1. 停止并禁用 apt 系统定时任务
echo "步骤 1: 停止系统更新定时器..."
systemctl stop apt-daily.timer apt-daily-upgrade.timer
systemctl disable apt-daily.timer apt-daily-upgrade.timer
systemctl mask apt-daily.service apt-daily-upgrade.service
systemctl daemon-reload

# 2. 修改 APT 配置文件
echo "步骤 2: 修改 APT 自动更新配置..."
CONF_FILE="/etc/apt/apt.conf.d/20auto-upgrades"

# 创建或覆盖配置文件
cat <<EOF > $CONF_FILE
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Download-Upgradeable-Packages "0";
APT::Periodic::AutocleanInterval "0";
APT::Periodic::Unattended-Upgrade "0";
EOF

# 同样处理 10periodic 文件（如果存在）
if [ -f /etc/apt/apt.conf.d/10periodic ]; then
    sed -i 's/"1"/"0"/g' /etc/apt/apt.conf.d/10periodic
fi

# 3. 禁止释放升级检查（禁止提示升级到新的 Ubuntu 大版本，如 22.04 -> 24.04）
echo "步骤 3: 禁用系统大版本升级检查..."
if [ -f /etc/update-manager/release-upgrades ]; then
    sed -i 's/Prompt=.*/Prompt=never/' /etc/update-manager/release-upgrades
fi

# 4. 锁定当前内核版本 (可选，防止手动 apt upgrade 时误升内核)
echo "步骤 4: 锁定当前内核版本，防止意外变动..."
# 获取当前运行的内核包名并锁定
CURRENT_KERNEL=$(dpkg-query -W -f='${Package}\n' | grep -E "linux-image-$(uname -r)|linux-headers-$(uname -r)" | xargs)
if [ ! -z "$CURRENT_KERNEL" ]; then
    apt-mark hold $CURRENT_KERNEL linux-image-generic linux-headers-generic
    echo "已锁定内核相关包: $CURRENT_KERNEL"
else
    echo "未能自动识别内核包名，请手动确认。"
fi

echo "--- 所有操作已完成！你的文件服务器现在处于静默状态 ---"
```

------

### 2. 如何使用该脚本？

1. **创建文件**： `nano disable-updates.sh`

2. **粘贴代码**： 将上面的代码粘贴进去，按 `Ctrl+O` 保存，`Ctrl+X` 退出。

3. **赋予执行权限**：

   ```bash
   chmod +x disable-updates.sh
   ```
   
4. **运行脚本**：

   ```Bash
   sudo ./disable-updates.sh
   ```

------

### 3. 脚本说明

- **Mask 命令**：脚本中使用了 `systemctl mask`。这比 `disable` 更彻底，它会软链接服务到 `/dev/null`，防止其他服务意外唤醒更新任务。
- **Release Upgrades**：关闭了版本跳转提示，你的服务器将一直停留在当前的 Ubuntu 版本（例如 22.04），直到你手动干预。
- **内核锁定**：即使你哪天手痒输入了 `sudo apt upgrade`，系统也会跳过内核更新，保证驱动和文件系统的稳定性。



## 验证这些设置是否生效

执行完脚本后，我们可以通过检查**服务状态**、**配置文件**以及 **APT 模拟运行**这三个维度来确认“封印”是否成功。

以下是验证步骤：

### 1. 检查关键服务是否已“戴上口罩” (Masked)

在脚本中我们使用了 `mask` 命令，这会让服务彻底失效。你可以运行以下命令：

```bash
systemctl status apt-daily.timer apt-daily-upgrade.timer
```

- **预期结果**：输出中应显示 `Loaded: masked` 和 `Active: inactive (dead)`。
- **含义**：这表示系统定时任务已经无法启动，即使其他程序尝试调用它也会被拒绝。

------

### 2. 检查 APT 周期配置

直接读取配置文件，确认所有的开关都已归零：

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

- **预期结果**：

  ```Plaintext
  APT::Periodic::Update-Package-Lists "0";
  APT::Periodic::Download-Upgradeable-Packages "0";
  APT::Periodic::AutocleanInterval "0";
  APT::Periodic::Unattended-Upgrade "0";
  ```
  
- **验证 Release 升级检查**：

  运行 `cat /etc/update-manager/release-upgrades`，确保 `Prompt=never`。

------

### 3. 检查软件包锁定状态 (Hold)

确认内核和相关组件是否被标记为“禁止变动”：

```bash
apt-mark showhold
```

- **预期结果**：列表里应该出现 `linux-image-generic`、`linux-headers-generic` 以及你当前使用的内核版本号。
- **含义**：这意味着即便你手动运行 `apt upgrade`，系统也会提示“下列软件包将被保留（kept back）”，而不会去更新它们。

------

### 4. 终极测试：模拟无人值守更新

你可以强制触发一次无人值守更新的检查流程，看看它是否会因为我们的设置而跳过：

```Bash
sudo unattended-upgrade --dry-run --debug
```

- **预期结果**：你应该在输出中看到类似 `Allowed origins are: []` 或者 `No packages found that can be upgraded unattended` 的提示。由于我们在配置里把所有开关都关了，它通常会直接退出或提示没有任务。

------

### 总结验证表

| **检查项**   | **命令**                                   | **理想状态**          |
| ------------ | ------------------------------------------ | --------------------- |
| **定时器**   | `systemctl status apt-daily.timer`         | `inactive` / `masked` |
| **配置值** | `apt-config dump` |grep Periodic|
| **内核锁**   | `apt-mark showhold`                        | 显示内核包名          |
| **版本更新** | `cat /etc/update-manager/release-upgrades` | `Prompt=never`        |





## 恢复默认的自动检查和更新

这套“解除封印”脚本将完全撤销之前的限制，恢复 Ubuntu 默认的自动检查和更新行为。

### 1. 制作脚本文件

你可以将以下内容保存为 `enable-updates.sh`：

```bash
#!/bin/bash

# 检查 root 权限
if [ "$EUID" -ne 0 ]; then 
  echo "请使用 sudo 运行此脚本"
  exit
fi

echo "--- 开始恢复 Ubuntu 自动更新功能 ---"

# 1. 解除服务 Mask 并重新启动定时任务
echo "步骤 1: 恢复系统更新定时器..."
systemctl unmask apt-daily.service apt-daily-upgrade.service
systemctl enable apt-daily.timer apt-daily-upgrade.timer
systemctl start apt-daily.timer apt-daily-upgrade.timer
systemctl daemon-reload

# 2. 恢复 APT 默认配置
echo "步骤 2: 恢复 APT 默认更新频率..."
CONF_FILE="/etc/apt/apt.conf.d/20auto-upgrades"

# 恢复为标准的每日检查配置
cat <<EOF > $CONF_FILE
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
EOF

# 3. 恢复大版本升级提醒
echo "步骤 3: 恢复系统大版本升级检查..."
if [ -f /etc/update-manager/release-upgrades ]; then
    # 恢复为默认的 lts (仅提醒长期支持版本) 或 normal
    sed -i 's/Prompt=never/Prompt=lts/' /etc/update-manager/release-upgrades
fi

# 4. 解锁内核版本
echo "步骤 4: 解锁内核版本..."
# 获取当前被锁定的包并解锁
HELD_PACKAGES=$(apt-mark showhold)
if [ ! -z "$HELD_PACKAGES" ]; then
    apt-mark unhold $HELD_PACKAGES
    echo "已解锁以下软件包: $HELD_PACKAGES"
else
    echo "未发现被锁定的软件包。"
fi

echo "--- 恢复完成！系统现在可以正常接收更新了 ---"
echo "建议立即运行一次: sudo apt update 来同步数据。"
```

------

### 2. 如何进行“安全更新”？

既然这台机器是作为文件服务器，即便解除了封印，我建议你也不要让它完全“裸奔”自动更新，而是采取以下**人工干预步骤**：

1. **运行脚本**：执行上面的 `sudo ./enable-updates.sh`。
2. **手动检查**：执行 `sudo apt update` 查看有多少更新。
3. **按需安装**：
   - 如果只是普通软件更新：执行 `sudo apt upgrade`。
   - **注意**：如果有内核更新，安装后务必安排在业务低峰期**重启服务器**，否则新内核不会生效。
4. **重新封印**：更新完成后，建议再次运行之前的 `disable-updates.sh` 脚本，让服务器重新回归“离线稳定状态”。

------

### 3. 💡 一个小技巧

如果你只是想**偶尔手动更新一下软件**，而不想长期开启后台自动任务，你其实**不需要**运行解除脚本。

你只需要直接运行： `sudo apt update && sudo apt upgrade`

虽然你之前锁定了内核（Hold），但 `apt` 会跳过内核只更新其他软件（比如 Samba、NFS 等）。这样既更新了服务插件，又保证了系统底层（内核）的绝对不动。



## 创建文件服务器

在内网文件服务器的架构中，选择 **Samba** 还是 **NFS**，主要取决于你的“客户机”是谁，以及你对文件权限控制的精细程度。

以下是它们的核心区别对比：

### 核心区别一览表

| **特性**     | **Samba (SMB/CIFS)**                | **NFS (Network File System)**              |
| ------------ | ----------------------------------- | ------------------------------------------ |
| **主要对象** | **跨平台**（Windows, macOS, Linux） | **类 Unix 系统**（Linux, BSD, macOS）      |
| **设计理念** | 模拟本地硬盘，适合办公协作          | 像挂载本地目录，适合服务器集群             |
| **性能**     | 稍慢（因协议层较重，小文件处理多）  | **极快**（协议精简，高并发下表现优异）     |
| **配置难度** | 较复杂（需配置用户、权限、共享名）  | 简单（仅需配置 IP 白名单和目录路径）       |
| **权限管理** | 非常精细（支持 ACL，模拟 Win 权限） | 依赖用户 UID/GID（两端 ID 不一致会很麻烦） |
| **安全性**   | 支持用户名/密码验证，数据加密       | 传统版依赖 IP 校验（Kerberos 可增强安全）  |

------

### 1. Samba：全能型选手

Samba 是 SMB 协议的开源实现。如果你的局域网里有 **Windows 电脑**，它是唯一且必须的选择。

- **优点**：
  - **完美的 Windows 兼容性**：在 Windows 资源管理器里输入 `\\IP` 就能直接用。
  - **身份认证**：可以为每个家庭成员或员工设置独立的账号密码。
  - **功能丰富**：支持打印机共享、回收站功能（vfs objects）。
- **缺点**：
  - 在 Linux 挂载时，性能略逊于 NFS。
  - 配置文件的参数非常多（`/etc/samba/smb.conf`）。

------

### 2. NFS：性能型选手

NFS 是 Linux 阵营的“原生”协议。如果你是多台 **Linux 服务器** 之间互传数据（比如 Web 服务器读取图片服务器），它是首选。

- **优点**：
  - **效率极高**：CPU 开销极小，对于大文件的传输速度非常接近物理带宽上限。
  - **透明挂载**：挂载后就像在自己硬盘的一个文件夹里操作一样顺滑。
- **缺点**：
  - **对 Windows 不友好**：虽然 Win10/11 专业版支持 NFS 客户端，但配置极其痛苦且容易出现乱码。
  - **权限坑多**：它是基于数字 ID（UID）匹配的。如果服务器上张三的 ID 是 1001，而客户机上 1001 是李四，那权限就会错乱。

------

### 如何选择？

- **选 Samba 的场景**：
  - 你要给家里的笔记本、iPad、手机提供视频或文件存储。
  - 办公环境下，Windows 电脑需要映射网络驱动器。
- **选 NFS 的场景**：
  - 你在跑 Linux 集群（如 K8s、多台 VPS），需要一个共享的存储池。
  - 你有一台 Linux 播放器（如 Kodi、树莓派）需要极高码率的 4K 视频流，追求极致吞吐量。

------

### 稳定性建议（针对你的文件服务器）

既然你追求稳定且不再更新系统：

1. **Samba 建议**：在 `smb.conf` 中显式指定 `server min protocol = SMB2`（甚至 SMB3），禁用古老且不安全的 SMB1，这能避免很多 Windows 挂载失败的问题。
2. **NFS 建议**：由于你是内网使用，建议固定 NFS 的端口（在 `/etc/default/nfs-kernel-server` 修改），这样如果你以后想开简单的防火墙规则，会方便很多。