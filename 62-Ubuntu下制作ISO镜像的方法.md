# Ubuntu下制作ISO镜像的方法

1. 如果你是直接从cd 压制iso 文件的，执行

   ```
   sudo umount /dev/cdrom
   
   dd if=/dev/cdrom of=file.iso bs=1024
   ```

2. 如果你要把某个文件或者目录压到iso 文件中，需要使用mkisofs 这个工具。这个工具非常强大，还可以压缩DVD 格式的iso 。你想改的参数都可以修改，而且还有-gui 这个参数，最简单的用法如下：

   安装 mkisofs

   ```
   sudo apt install mkisofs
   ```

   制作ISO文件

   ```
   mkisofs -r -o file.iso your_folder_name/
   ```

3. 如果你想装酷生成一个MD5 文件，执行

   ```
   md5sum file.iso > file.iso.md5
   ```

4. 如果你想把一个iso 文件烧制出来更简单了。右键点这个文件，有烧录的选项。