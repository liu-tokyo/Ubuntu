# Ubuntu下制作ISO镜像的方法

1. 如果你是直接从cd 压制iso 文件的，执行

   ```bash
   sudo umount /dev/cdrom
   
   dd if=/dev/cdrom of=file.iso bs=1024
   ```

2. 如果你要把某个文件或者目录压到iso 文件中，需要使用mkisofs 这个工具。这个工具非常强大，还可以压缩DVD 格式的iso 。你想改的参数都可以修改，而且还有-gui 这个参数，最简单的用法如下：

   安装 mkisofs

   ```bash
   sudo apt install mkisofs
   ```

   制作ISO文件

   ```bash
   mkisofs -r -o file.iso your_folder_name/
   ```

3. 如果你想装酷生成一个MD5 文件，执行

   ```bash
   md5sum file.iso > file.iso.md5
   ```

4. 如果你想把一个iso 文件烧制出来更简单了。右键点这个文件，有烧录的选项。



## 指令说明

- 指令示例：

    ```bash
    mkisofs -l -v -r -J -o ./kylin-monitor.iso Kylin-Monitor-x86_64_V1.0/
    ```

    - `-l` 允许长文件名
    - `-J` 或 -joliet 参数，使用Joliet格式的目录与文件名称
    - `-r` 使用Rock Ridge，并开放全部文件的读取权限
    - `-v` verbose，允许冗长
    - `-o` 设置输出文件名
