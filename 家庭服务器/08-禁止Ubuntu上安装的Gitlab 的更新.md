##  禁止 Ubuntu 上安装的Gitlab 的更新

如果你是在 Ubuntu 上通过官方仓库（Omnibus 方式）安装的 GitLab，防止它跟随系统更新（如 `sudo apt upgrade`）是非常明智的。GitLab 的版本升级通常涉及数据库迁移，自动更新极易导致服务崩溃。

你可以通过以下几种方式实现“版本锁定”：

------

### 方法一：使用 `apt-mark` 锁定版本 (最推荐)

这是最简单且最有效的方法。它告诉 Ubuntu 的包管理器：**“除非我手动解除锁定，否则不要碰这个包。”**

1. **锁定 GitLab 包：** 根据你安装的版本（社区版 CE 或 企业版 EE），执行：

   ```bash
   # 如果是社区版
   sudo apt-mark hold gitlab-ce
   
   # 如果是企业版
   sudo apt-mark hold gitlab-ee
   ```

2. **确认锁定状态：** 执行以下命令，如果输出中包含 `gitlab-ce`（或 ee），说明锁定成功。

   ```bash
   apt-mark showhold
   ```

3. **未来如何解锁（需要手动升级时）：**

   ```bash
   sudo apt-mark unhold gitlab-ce
   ```

------

### 方法二：修改 Apt 源配置 (进阶)

如果你想彻底从源头上切断更新检查，可以禁用 GitLab 的官方仓库。

1. 找到 GitLab 的源列表文件： 通常在 `/etc/apt/sources.list.d/gitlab_gitlab-ce.list`（或 ee）。

2. 编辑该文件，在以 `deb` 开头的行最前面加上 `#` 号注释掉。

   ```bash
   sudo nano /etc/apt/sources.list.d/gitlab_gitlab-ce.list
   ```

3. 保存并退出（Ctrl+O, Enter, Ctrl+X）。

4. 执行 `sudo apt update`。

**缺点：** 这样你连新版本的安全补丁信息都收不到了。

------

### 方法三：配置无人值守更新 (Unattended-Upgrades)

如果你的 Ubuntu 开启了自动安装安全更新，你需要确保 GitLab 不在自动更新名单中。

1. 编辑配置文件：

   ```bash
   sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
   ```

2. 找到 `Unattended-Upgrade::Package-Blacklist` 部分。

3. 添加 GitLab 进黑名单：

   ```Plaintext
   Unattended-Upgrade::Package-Blacklist {
       "gitlab-ce";
       "gitlab-ee";
   };
   ```

------

### 💡 避坑小贴士

- **不要只禁不升：** GitLab 经常发布重要的安全补丁。建议你每隔几个月，在**手动备份**（执行 `sudo gitlab-backup create`）之后，再手动执行升级。
- **跳版本升级风险：** GitLab 不支持跨大版本直接升级（例如从 15.x 直接跳到 17.x）。如果你长期禁止更新，将来升级时必须遵循[官方升级路径](https://www.google.com/search?q=https://docs.gitlab.com/ee/update/index.html%23upgrade-paths)。