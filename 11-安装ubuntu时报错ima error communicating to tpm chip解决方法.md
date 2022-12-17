# 安装ubuntu时报错ima: error communicating to tpm chip解决方法

## 办法1

1. 进入安装时，光标移动到"install ubuntu"

2. 按"e"进入编辑模式，进入命令行模式

3. 找到’‘quite splash’'然后去掉"—"后，添加“nomodeset”（依照不同显卡进行不同显卡驱动选项的添加，如果使用的是Nvidia显卡，添加nomodeset）

4. F10安装

5. 安装成功后重启，选择Ubuntu，同样按e，在quite splash 后加 nomodeset；按F10启动系统

6. 进入图形界面，打开终端
   1.更新apt-get源列表

   ```
   sudo apt-get update
   sudo apt-get upgrade
   ```

   2.添加驱动源

   ```
   sudo add-apt-repository ppa:graphics-drivers/ppa
   sudo apt-get update
   ```

   3.打开软件更新，选择NvidIa驱动版本430(根据显卡选择对应驱动)，选择应用更改即可（一次不行

## 办法2

只需要按照下面链接中的教程提示，Clear TPM 就可以搞定了。如果你的电脑BIOS里已经有“Clear Security Chip”，那直接按回车就可以了，不用完全执行这个教程。Clear之后，按F10保存，然后重启电脑，最下面的那些内容不用看。

这个教程适用于ThinkPad电脑，其它电脑请自己寻找TPM的位置，只要找到就可以了。

https://www.cnblogs.com/cxun/archive/2009/10/19/1586209.html