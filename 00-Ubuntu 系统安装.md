# Ubuntu 系统安装

## 1. 虚拟内存

- **查询虚拟内存设置**

  ```bash
  sudo swapon --show
  ```

  如果系统配置了虚拟内存，该命令会输出类似以下的信息：

  ```bash
  NAME      TYPE SIZE USED PRIO
  /swapfile file   2G   0B   -2
  ```

- **临时关闭当前的交换空间**，停用所有交换文件：

  完全禁用交换空间后，如果物理内存耗尽，系统将无法使用磁盘作缓冲，这会导致系统直接触发 OOM Killer 强行终止高内存占用的程序，甚至造成系统死机。请确保您的物理内存（RAM）足够大。
  
  ```bash
  sudo swapoff -a
  ```

- **从开机挂载项中移除交换文件配置**

  打开系统挂载配置文件：
  
  ```bash
  sudo nano /etc/fstab
  ```

  找到包含 `/swapfile` 的那一行（通常类似于 `/swapfile none swap sw 0 0`），在行首添加 `#` 将其注释掉，或者直接删掉这一行，然后保存并退出。 **验证方法：** 运行 `cat /etc/fstab`，确认文件中关于 `/swapfile` 的行已被注释或删除。
  
- **删除物理交换文件释放磁盘空间**

  删除系统中的交换文件实体：
  
  ```bash
  sudo rm -f /swapfile
  ```

  **验证方法：** 运行 `ls -l /swapfile`，如果提示 `No such file or directory` 则说明文件已被彻底删除。
  
- **其它下常见操作：**

  ```bash
  ## 查看当前的内存和交换空间大小：
  free -h
  
  ## 禁用现有的交换文件：
  sudo swapoff /swapfile
  
  ## 调整文件大小：
  sudo fallocate -l 4G /swapfile
  ## 如果 fallocate 不支持，可以使用
  sudo dd if=/dev/zero of=/swapfile bs=1M count=4096
  
  ## 设置正确的权限：
  sudo chmod 600 /swapfile
  
  ## 将其标记为交换空间：
  sudo mkswap /swapfile
  
  ## 重新启用交换文件：
  sudo swapon /swapfile
  ```

  