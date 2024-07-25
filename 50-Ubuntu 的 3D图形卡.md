# Ubuntu 的 3D图形卡

3D视频加速简介

在 Ubuntu 中大多数的视频硬件会自行工作。然而某些种类的视频硬件 3D 加速功能 (为某些游戏所需) 将不会被自动支持。本部分内容详细介绍如何让某些流行的视频硬件可以在这方面正常工作。 

## 检查3D加速器是否正在工作。

- 要做到这一点，在终端输入：
  ```shell
    glxinfo | grep rendering     
    ```
    
- 如果 3D 加速工作正常，将会显示以下内容：      
  ```shell
    direct rendering: Yes
    ```
    
- 如果没有的话，请按照下面章节的说明激活 3D 加速。

## Nvidia 显卡的 3D 驱动

- 在 Ubuntu 中 Nvidia 显卡并不会自动启用 3D 加速，因为厂商没有发布开源驱动。然而是可以激活 3D 加速的。这有赖于您的显卡类型。

  - 如果您有一块老的 TNT、TNT2、TNT Ultra、GeForce1 或 GeForce2 卡的话，请从 Restricted 软件库中安装 nvidia-glx-legacy 和 nvidia-settings 软件包 (参见 2 ― 添加、删除和更新应用程序)。

  - 或者，如果您有一个更新的卡的话， 请从 Restricted 软件库中安装 nvidia-glx 软件包 (参见 2 ― 添加、删除和更新应用程序).

  - 要启用新的驱动，请在终端运行以下命令：

    ```
    sudo nvidia-glx-config enable 
    ```

  - 您可以通过运行 nvidia-settings 应用程序来调整新驱动的设置。如果您想的话，还可以为该程序添加一个菜单项 (参见 第4.1.1节 ― 菜单编辑)。

- 驱动可去Nvidia 看看

## ATI 显卡的 3D 驱动

- 大多数 ATI 显卡都能自动在 Ubuntu 中正常运行。要检查您的显示是否启用 3D 加速，可以参见 第4.3.2.1节 ― 3D视频加速简介。如果没有启用，以下步骤可以激活它。

  - 从 Restricted 软件库中安装 xorg-driver-fglrx 软件包 (参见 2 ― 添加、删除和更新应用程序)。

  - 您现在需要配置计算机来使用新的驱动，所以在终端中运行命令：

    ```
    sudo dpkg-reconfigure xserver-xorg 
    ```

  - 当出现对话框并询问是否自动检测您的显卡时，选择 是。
    
  - 当提示选择一个驱动时，选择 fglrx。
    
  - 酌情按余下的说明操作。
    
  - 重启您的机器以便所做改变生效。

## glxinfo 安装

Miscellaneous Mesa GL utilities 

| Linux      | Command                                                     |      |
| ---------- | ----------------------------------------------------------- | ---- |
| Debian     | apt-get install mesa-utils                                  |      |
| Ubuntu     | apt-get install mesa-utils                                  |      |
| Alpine     | apk add mesa-demos                                          |      |
| Arch Linux | pacman -S mesa-demos                                        |      |
| Kali Linux | apt-get install mesa-utils                                  |      |
| CentOS     | yum install glx-utils                                       |      |
| Fedora     | dnf install glx-utils                                       |      |
| Raspbian   | apt-get install mesa-utils                                  |      |
| Docker     | docker run cmd.cat/glxinfo glxinfo<br />powered by Commando |      |
| -          | -                                                           | -    |
| -          | -                                                           | -    |



## 参照资源

- [Ubuntu桌面入门指南](https://wiki.ubuntu.org.cn/Ubuntu%E6%A1%8C%E9%9D%A2%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97#3.8.1..E2.80.83.E5.9F.BA.E6.9C.AC.E7.BC.96.E8.AF.91.E5.99.A8)