# 安装简易家庭服务器(Gitea)

用的比较过时的电脑，所以安装的系统是 **Ubuntu Server** 的 **24.04** 版本 。  
不过，服务器版还是太痛苦了，还是选择安装 **Ubuntu Desktop**，然后再设置好之后，切换回 **纯指令界面**，也一样能够节省内存。  
官方网址：`https://docs.gitea.com/installation/database-prep`

## 1. 系统安装

选择桌面版，主要也是为了安装方便，不然 服务器 版的初始设置，还是比较费力气。

### 1.1 系统安装必要参数

- 安装必须的设置项目：

    | 项目   | 设置       | 备注 |
    | ------ | ---------- | ---- |
    | 名字   | vip        |      |
    | 用户名 | vip        |      |
    | 设备名 | vip-server |      |
    | 口令   | qwer1234   |      |

### 1.2 安装 SSH 服务

- **检查 SSH 服务状态**

  ```bash
  systemctl status ssh
  ```

  **如果看到 `active (running)`：** 恭喜你，SSH 已经安装并正在运行。
  
  **如果提示 `Unit ssh.service could not be found`：** 说明系统根本没安装 SSH 服务。
  
  **如果看到 `inactive (dead)`：** 说明安装了但没启动，输入 `sudo systemctl start ssh` 即可。
  
- **尝试本地自测**

  除了查状态，你可以试着自己连接自己：
  
  ```
  ssh localhost
  ```
  
  **如果提示输入密码：** 说明服务工作正常。
  
  **如果提示 `Connection refused`：** 说明服务没开。
  
- **安装 SSH 服务**
  
  ```bash
  sudo apt update && sudo apt install openssh-server -y
  ```
  
  安装完成后，它会自动启动并设置为**开机自启**。
  
- **确认防火墙是否放行**

  有时候服务开了，但连不上，通常是被 Ubuntu 的防火墙（UFW）拦住了：

  ```
  sudo ufw allow ssh
  # 或者
  sudo ufw allow 22
  ```

- **关键操作：获取 IP 地址**

  ```
  ip addr show | grep -i "inet "
  ```



## 2. 安装&设置 Gitea

### 2.1 安装

- **更新系统并安装依赖**

  首先，确保你的系统软件包是最新的，并安装必要的 Git 环境。

  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo apt install git sqlite3 -y
  ```

- **创建运行用户**

  出于安全考虑，Gitea 不应该以 root 身份运行。我们创建一个名为 `git` 的系统用户。

  ```bash
  sudo adduser \
     --system \
     --shell /bin/bash \
     --gecos 'Git Version Control' \
     --group \
     --disabled-password \
     git
  ```

- **下载并配置目录**

  前往 [Gitea 官网下载页面](https://dl.gitea.com/gitea/) 获取最新的二进制文件（通常选择 `gitea-1.x.x-linux-amd64`）。

  ```bash
  # 下载（请根据最新版本替换链接），当前最新版是 1.25.4
  # wget -O gitea https://dl.gitea.com/gitea/1.21.7/gitea-1.21.7-linux-amd64
  wget -O gitea https://dl.gitea.com/gitea/1.25.4/gitea-1.25.4-linux-amd64
  
  # 赋予执行权限并移动到系统路径
  chmod +x gitea
  sudo mv gitea /usr/local/bin/gitea
  
  # 创建必要的文件夹结构
  sudo mkdir -p /var/lib/gitea/{custom,data,log}
  sudo chown -R git:git /var/lib/gitea/
  sudo chmod -R 750 /var/lib/gitea/
  sudo mkdir /etc/gitea
  sudo chown root:git /etc/gitea
  sudo chmod 770 /etc/gitea
  ```
  
- **手动创建并赋权家目录**

  ```bash
  # 强制创建家目录（如果不存在）
  sudo mkdir -p /home/git
  # 将整个家目录的所有权交给 git 用户和组
  sudo chown -R git:git /home/git
  # 设置正确的目录权限（文件夹 750, 内部文件 640）
  sudo chmod -R 750 /home/git
  ```
  
  检查并创建 .ssh 目录（预处理）

  ```bash
  sudo mkdir -p /home/git/.ssh
  sudo chown -R git:git /home/git/.ssh
  sudo chmod 700 /home/git/.ssh
  ```
  
  更新 `/etc/passwd` 文件
  
  ```bash
  sudo usermod -d /home/git git
  ```

- **创建 Systemd 服务**

  为了让 Gitea 随系统启动，我们需要创建一个服务文件： 

  ```bash
  sudo nano /etc/systemd/system/gitea.service
  ```

  将以下内容粘贴进去：

  ```ini
  [Unit]
  Description=Gitea (Git with a cup of tea)
  After=network.target
  
  [Service]
  RestartSec=2s
  Type=simple
  User=git
  Group=git
  WorkingDirectory=/var/lib/gitea/
  ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
  Restart=always
  Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
  
  [Install]
  WantedBy=multi-user.target
  ```

  启动服务：

  ```bash
  sudo systemctl enable --now gitea
  ```
  
  查询服务状态：
  
  ```bash
  sudo systemctl status gitea
  ```
  
- **验证用户配置**

  检查系统中的 `git` 用户是否指向了正确的家目录：

  ```bash
  getent passwd git
  ```

  **正确输出应类似于：** `git:x:1001:1001:Git Version Control,,,:/home/git:/bin/bash` *注意倒数第二个字段必须是 `/home/git`。*

  如果不存在 `/home/git` 的话，停止 gitea 服务，运行如下指令之后，再次查看，没有问题的话，重新启动 gitea 服务。

  ```bash
  sudo systemctl stop gitea
  sudo usermod -d /home/git git
  ```

- **重启 Gitea 并继续安装**

  ```bash
  sudo systemctl restart gitea
  ```

  现在回到浏览器，刷新安装页面或重新点击“立即安装”。由于它现在拥有了 `/home/git` 的完全控制权，报错应该会消失。

### 2.2 设置

- **通过网页进行初始设置**

  Gitea 默认在 **3000** 端口运行。

  1. 在浏览器访问：`http://你的服务器IP:3000`
  
     ```
     http://localhost:3000
     ```
  2. **数据库设置**：如果你是个人使用，数据库类型选 **SQLite3** 最简单，路径保持默认即可。
  3. **一般设置**：修改“站点标题”，将“域名”和“基础 URL”改为你服务器的实际 IP 或域名。
  4. **管理员帐号**：建议立即创建一个管理员账号（在页面底部的“可选设置”中）。
  
     | 项目         | 设置值                | 备注 |
     | ------------ | --------------------- | ---- |
     | 管理员用户名 | vip                   |      |
     | 邮箱地址     | liuzhijun@hotmail.com |      |
     | 管理员密码   | qwer1234              |      |
     | SSH 服务端口 | 22                    | 默认 |
  
     *这些配置选项将写入以下位置: `/etc/gitea/app.ini`*

  **安装完成后的安全加固**

  为了安装成功，我们给了 `git` 用户写入 `/etc/gitea` 的权限。但安装完成后，为了防止黑客通过 Gitea 漏洞篡改你的系统配置，建议将配置文件改为**只读**：

  等到你在网页上点击“立即安装”并成功跳转到登录页面后，请回到终端执行：

  ```bash
  # 将配置文件设为只读，仅允许 git 用户读取
  sudo chmod 750 /etc/gitea
  sudo chmod 640 /etc/gitea/app.ini
  ```

  

### 2.3 常见注意事项

- **防火墙**：如果你开启了 UFW，记得允许 3000 端口：`sudo ufw allow 3000/tcp`。

- **反向代理**：长期使用建议配置 Nginx 并加装 SSL 证书（HTTPS），安全性更高。

### 2.4 服务器 IP 改变

- **服务器端 IP 地址变化之后，需要修改 Gitea 的配置文件：**

  ```
  sudo nano /etc/gitea/app.ini
  ```

  检查 **`[server]`** 部分，修改相应的 IP 地址：

  ```ini
  [server]
  SSH_DOMAIN = 192.168.5.46
  DOMAIN = 192.168.5.46
  HTTP_PORT = 3000
  ROOT_URL = http://192.168.5.46:3000/
  APP_DATA_PATH = /var/lib/gitea/data
  DISABLE_SSH = false
  SSH_PORT = 22
  LFS_START_SERVER = true
  LFS_JWT_SECRET = pTWrhEGMinxACdPtFx1ql_Od8WGFoXGqzPZkvagSYxQ
  OFFLINE_MODE = true
  ```

## 3. 配置反向代理

配置 Nginx 反向代理不仅能让你通过域名（如 `git.example.com`）访问 Gitea，还能方便地配置 SSL 证书（HTTPS），提升安全性。

### 3.1 安装 Nginx

- **如果你还没有安装 Nginx，请先运行：**

  ```bash
  sudo apt update
  sudo apt install nginx -y
  ```

### 3.2 创建 Gitea 配置文件

- **创建一个新的虚拟主机配置文件：**

  ```
  sudo nano /etc/nginx/sites-available/gitea
  ```

  将以下内容粘贴进去（记得将其中的 **`git.example.com`** 替换为你的实际域名）：

  ```nginx
  server {
      listen 80;
      server_name git.example.com;
  
      # 限制上传文件大小，防止大仓库推送失败
      client_max_body_size 512M;
  
      location / {
          proxy_pass http://localhost:3000;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
  ```

### 3.3 启用配置并测试

- **执行以下命令建立软链接并重启 Nginx：**

  ```bash
  # 启用配置
  sudo ln -s /etc/nginx/sites-available/gitea /etc/nginx/sites-enabled/
  
  # 检查 Nginx 语法是否正确
  sudo nginx -t
  
  # 重启 Nginx
  sudo systemctl restart nginx
  ```

### 3.4 配置 HTTPS (使用 Let's Encrypt)

- **手动配置 SSL 比较繁琐，推荐使用 Certbot 自动获取免费证书。这不仅安全，还能自动完成 Nginx 的配置修改。**

  ```bash
  sudo apt install certbot python3-certbot-nginx -y
  sudo certbot --nginx -d git.example.com
  ```

  按照提示操作后，`Certbot` 会自动更新你的 `Nginx` 文件，将 HTTP 流量重定向到 HTTPS。

### 3.5 修改 Gitea 配置文件

为了确保 Gitea 生成的链接（如 Clone 链接）正确，你需要更新 Gitea 的 `app.ini`。

1. **打开配置文件：**

   ```bash
   sudo nano /etc/gitea/app.ini
   ```

2. **找到 `[server]` 部分，修改以下两项：**

   ```bash
   DOMAIN = git.example.com
   ROOT_URL = https://git.example.com/
   ```

3. **重启 Gitea：**

   ```bash
   sudo systemctl restart gitea
   ```

   **核心参数说明**

   | **参数**                   | **说明**                                                     |
   | -------------------------- | ------------------------------------------------------------ |
   | **`client_max_body_size`** | 非常重要。如果设得太小，你往仓库 `git push` 大文件时会报错 `413 Request Entity Too Large`。 |
   | **`proxy_set_header`**     | 确保 Gitea 能获取到访客的真实 IP 和协议（HTTP/HTTPS），而非始终看到 127.0.0.1。 |

### 3.6 检查 app.ini 中 SSH 端口的设置

检查 `app.ini` 中的 SSH 配置非常重要，因为如果这里的设置与你服务器实际运行的 SSH 服务不匹配，Gitea 界面上显示的 Git 克隆地址（如 `git@yourdomain.com:repo.git`）就会失效。

- **定位并打开配置文件**

  在之前的安装步骤中，配置文件通常位于 /etc/gitea/app.ini。使用以下命令打开它：

  ```bash
  sudo nano /etc/gitea/app.ini
  ```

- **检查 `[server]` 部分**

  滚动查找 `[server]` 标签，重点关注以下几个参数：

  | **配置项**            | **示例值**        | **说明**                                                  |
  | --------------------- | ----------------- | --------------------------------------------------------- |
  | **`DISABLE_SSH`**     | `false`           | 必须为 `false` 才能启用 SSH 访问。                        |
  | **`SSH_PORT`**        | `22`              | **最关键的一项**。这是 Gitea 界面显示的端口。             |
  | **`SSH_LISTEN_PORT`** | `22`              | Gitea 内置 SSH 监听的端口（通常保持与 `SSH_PORT` 一致）。 |
  | **`DOMAIN`**          | `git.example.com` | 决定了克隆地址中 `@` 符号后面的域名。                     |
  | **`SSH_DOMAIN`**      | `git.example.com` | 如果 SSH 访问使用不同的域名，可以在此单独指定。           |

- **特殊情况：如果你修改了系统的 SSH 端口**

  如果你为了安全将 Ubuntu 系统的 SSH 端口从 `22` 改成了（例如）`2222`，那么 `app.ini` 必须同步修改：

  - **`SSH_PORT = 2222`**
  - **`SSH_LISTEN_PORT`**：如果你使用的是 Gitea 转发模式（默认），这一项通常不需要改，或者设为 `2222`。

- **保存并重启生效**

  修改完成后，按 `Ctrl + O` 保存，`Ctrl + X` 退出。然后必须重启 Gitea 服务才能应用更改：

  ```bash
  sudo systemctl restart gitea
  ```

**验证方法**

重启后，登录 Gitea 网页端，打开任意一个仓库，查看右上角的 **SSH 克隆链接**。

- 如果显示：`git@yourdomain.com:user/repo.git`（说明使用的是默认 22 端口）。
- 如果显示：`ssh://git@yourdomain.com:2222/user/repo.git`（说明非标端口配置成功）。



## 4. 测试 SSH 连通性

测试 Gitea 的 SSH 连通性是确保你能正常 `git push` 的最后一步。这不仅是检查端口，更是验证你的 **本地私钥** 与 **Gitea 上的公钥** 是否配对成功。

### 4.1 确认本地已生成密钥对

- 在你的 **本地电脑**（不是服务器）上，检查是否存在 SSH 密钥：

  **Windows (PowerShell):** `ls ~/.ssh/`
  **macOS/Linux:** `ls -l ~/.ssh/id_rsa.pub`

- 如果没有看到 `id_rsa.pub` 或 `id_ed25519.pub`，请运行以下命令生成：

  ```bash
  ssh-keygen -t ed25519 -C "your_email@example.com"
  ```


### 4.2 将公钥添加到 Gitea

1. 打开本地的公钥文件（例如 `id_ed25519.pub`），复制全部内容。
2. 登录 Gitea 网页，点击右上角头像 -> **设置 (Settings)** -> **SSH / GPG 密钥**。
3. 点击 **增加密钥**，将内容粘贴进去并保存。

------

### 4.3 执行连通性测试

在你的 **本地终端** 输入以下命令（请根据你的实际域名和端口修改）：

- **如果使用默认 22 端口：**

  ```bash
  ssh -T git@yourdomain.com
  ```

- **如果你修改了 SSH 端口（例如 2222）：**

  ```
  ssh -T git@yourdomain.com -p 2222
  ```

### 4.4 结果判定

你会看到类似下面的反馈：

- **成功：**

  > `Hi there, username! You've successfully authenticated with key named YourKeyName, but Gitea does not provide shell access.` *这表示配置完全正确！你可以开始克隆代码了。*

- **失败 (Permission denied)：** 说明公钥没对上。请检查你上传到 Gitea 的公钥是否来自你当前使用的私钥。

- **失败 (Connection refused/timeout)：** 说明网络不通。请检查服务器防火墙是否放行了对应的端口。

------

### 4.5 进阶提示：配置本地 SSH Config

- **为了避免每次都要输入端口号或长域名，你可以在本地创建/修改 `~/.ssh/config` 文件：**

    ```Plaintext
    Host mygit
        HostName yourdomain.com
        User git
        Port 2222          # 如果是默认22则不需要这行
        IdentityFile ~/.ssh/id_ed25519
    ```

配置后，你只需要运行 `ssh -T mygit` 即可测试，克隆代码也可以简写为 `git clone mygit:username/repo.git`。



## 5. 客户端安装

### 5.1 安装 Git for Windows

- **安装 Git for Windows**
  
  ```powershell
  winget install --id Git.Git --source winget
  ```
  
  如果如下两个工具软件没有安装的话，最好也安装上：

  ```powershell
  winget install --id 7zip.7zip --source winget
  winget install --id WinMerge.WinMerge --source winget
  ```
  
  **注意：**如果 Windows10 尚未安装 Winget 工具，执行如下指令进行安装（终端管理员）：
  
  ```powershell
  $progressPreference = 'silentlyContinue'
  Write-Host "Installing WinGet PowerShell module from PSGallery..."
  Install-PackageProvider -Name NuGet -Force | Out-Null
  Install-Module -Name Microsoft.WinGet.Client -Force -Repository PSGallery | Out-Null
  Write-Host "Using Repair-WinGetPackageManager cmdlet to bootstrap WinGet..."
  Repair-WinGetPackageManager
  Write-Host "Done."
  ```
  
- **配置 Git 信息**

  ```powershell
  git config --global user.email "xxx@yyy.com"
  git config --global user.name "xxx yyyzzz"
  ```

  ***注：**换成自己的邮件地址、用户名。*

### 5.2 安装 TortoiseGit

- **安装 TortoiseGit （用户界面更加友好）：**

  ```powershell
  winget install --id TortoiseGit.TortoiseGit --source winget
  ```

- **设置 Git 执行文件路径**（Git 事先安装的话，默认已经正确设置）：

  TortoiseGit 本身只是一个外壳（GUI），它必须依赖底层的 `git.exe` 才能工作。

  1. 在任意文件夹内 **右键** -> **TortoiseGit** -> **Settings** (设置)。
  2. 在左侧菜单找到 **General** (常规)。
  3. 在右侧找到 **Git.exe Path**：
     - 点击 **Browse** (浏览)，找到你安装 Git 的位置。
     - 通常路径是：`C:\Program Files\Git\bin`。
  4. 点击 **Check now** (立即检查)按钮。如果下方显示了版本号（如 *git version 2.x.x*），说明配置成功。

- **TortoiseGit 使用标准的 SSH 客户端:**

  如果你在 Linux 服务器（如你之前提到的 `/home/git`）上配置了 SSH 密钥，但 TortoiseGit 报错，通常是因为它默认使用的是内置的 **Putty (TortoisePlink.exe)**，这和 Linux 习惯的 `OpenSSH` 不太一样。

  如果你想让它表现得和命令行 Git 一模一样：

  1. 在 **Settings** -> **Network** 中。
  2. 将 **SSH Client** 的路径修改为 Git for Windows 自带的 SSH：
     - 路径通常为：`C:\Program Files\Git\usr\bin\ssh.exe`。
  3. 这样它就会直接读取你 `C:\Users\用户名\.ssh\id_rsa` 下的密钥，而不是要求 `.ppk` 文件。

### 5.3 创建 Git 用密钥

- **如何指令创建密钥：**

  ```
  ssh-keygen -t ed25519 -C "xxx@yyy.com"
  ```

  ***注：**把 `xxx@yyy.com` 换成自己的真实邮件地址。*

- **把 公钥 登录到 Gitea 服务端：**

  用文本方式，打开如下文件：

  ```
  C:\Users\%USERNAME%\.ssh\id_ed25519.pub
  ```

  把文件内的内容拷贝之后，登录到 Gitea 的 SSH 密钥内。

### 5.4 仓库文件没有绿色对勾

可能需要 **重新启动客户端电脑**，就能解决，如果仍然无法解决，使用如下办法解决：

- **修改图标覆盖设置（最有效）**

  Windows 对图标覆盖的数量有限制（通常只有 11-15 个），如果你安装了 OneDrive、Dropbox、云盘等，它们会抢占名额，导致 TortoiseGit 的图标排不上号。

  1. 右键任意位置 -> **TortoiseGit** -> **Settings** (设置)。
  2. 左侧选择 **Icon Overlays** (图标覆盖)。
  3. 在右侧找到 **Status cache** (状态缓存)，将其改为 **Shell**（如果原本是 Shell，尝试改为 **Default**）。
  4. 在下方的 **Drive Types** (驱动器类型) 中，确保勾选了 **Fixed drives** (固定驱动器) 和 **Network drives** (如果你在用网盘/虚拟机)。

- **注册表“上位”法（强力修复）**

  如果上面的设置无效，说明你的 Windows 注册表被其他软件占满了。

  1. 按下 `Win + R`，输入 `regedit` 回车。
  2. 跳转到路径： `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers`
  3. 你会看到很多文件夹。Windows 会按**字母顺序**加载前 11-15 个。
  4. **技巧**：在所有以 `Tortoise` 开头的文件夹名字前面**加几个空格**（例如把 `TortoiseAdded` 改成 `   TortoiseAdded`），让它们的排名强行排到最前面。
  5. **重启电脑**或在任务管理器里重启 `explorer.exe` (资源管理器)。

- **检查目录是否被排除**

  有时候你不小心把整个盘符加入了“排除列表”。

  1. 回到 **Settings** -> **Icon Overlays**。
  2. 查看 **Exclude paths** (排除路径) 框里有没有你的代码目录。如果有，删掉它。
  3. 在 **Include paths** (包含路径) 中，尝试手动添加你的仓库路径（例如 `D:\Work\*`）。

- **确认这真的是一个 Git 仓库**

  虽然听起来很基础，但请确认文件夹内是否有隐藏的 `.git` 文件夹。

  - 如果没有 `.git` 文件夹，它只是普通文件，自然没有图标。
  - 如果之前在 Linux `/home/git` 下修改了权限或路径，导致克隆不完整，图标也不会出现。

- **刷新缓存**

  如果只是偶尔卡住，可以尝试：

  - 在文件夹内右键 -> **TortoiseGit** -> **Clean up...**。
  - 勾选 **Refresh shell overlays**，点击确定。



## 附1. Ubuntu 界面切换

- **设置为纯指令模式启动：**

  ```bash
  sudo systemctl set-default multi-user.target
  ```

  立即重启生效：

  ```bash
  sudo reboot
  ```

  **效果：** 重启后，你将不会看到登录背景和桌面，而是直接看到黑底白字的 `login:` 提示符。输入你的用户名和密码即可操作。

- **临时回到桌面**

  在指令界面登录后，输入：

  ```bash
  sudo systemctl start gdm3
  ```

  (如果是 Lubuntu，请将 `gdm3` 换成 `sddm` 或 `lightdm`)

- **永久改回桌面启动：**

  ```bash
  sudo systemctl set-default graphical.target
  ```

  

