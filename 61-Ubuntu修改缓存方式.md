# Ubuntu修改缓存方式

> 修改物理内存和swap使用优先级

写在前面的话：我最近把我只有512M的老爷机加了一跟512M的的内存。但是我发现，当机器运行一段时间后越来越慢，一看系统监视器发现 `swap` 里面居然驻留了`200M`的数据，我想：好你个乌斑兔儿，居然好好的物理内存你不吃，来吃`swap`！所以，自己就准备对它进行点点“教育”。

在`ubuntu`里面，`swappiness`的值的大小对如何使用`swap`分区是有着很大的联系的。`swappiness=0`的时候表示最大限度使用物理内存，然后才是 `swap` 空间，`swappiness＝100`的时候表示积极的使用`swap`分区，并且把内存上的数据及时的搬运到`swap`空间里面。两个极端，对于 `ubuntu`的默认设置，这个值等于`60`，建议修改为`10`。具体这样做：

1. 查看你的系统里面的`swappiness`

    ```bash
    cat /proc/sys/vm/swappiness
    ```

    不出意外的话，你应该看到是 `60`

2. 修改`swappiness`值为`10`

    ```bash
    sudo sysctl vm.swappiness=10
    ```

3. 以上是临时性的修改，在你重启系统后会恢复默认的`60`，所以，还要做一步：

    ```bash
    sudo nano /etc/sysctl.conf
    ```

    在这个文档的最后加上这样一行:

    ```properties
    vm.swappiness=10
    ```

然后保存，重启。OK，你的设置就生效了。你会发现，现在乌斑兔儿跑得更快了！
当然，你可以用其他编辑器进行修改，如`kate`、`vi`、`vim`、`gedit`……只需要把`nano`替换成它们就OK了！考虑到大多数人都用的`gnome`桌面，用`gedit`更普遍。