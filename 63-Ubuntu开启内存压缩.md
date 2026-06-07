# Ubuntu开启内存压缩

## 1. 安装内存压缩工具

### 案一：安装通用替代包 `zram-tools`

这是目前最推荐的替代方案，它比老的 `zram-config` 更灵活，允许你通过配置文件调整压缩率和内存分配。

1. **更新源并安装：**

   ```bash
   sudo apt update
   sudo apt install zram-tools
   ```
   
2. **配置（可选）：** 你可以编辑 `/etc/default/zramswap` 来调整分配多少内存作为压缩区（默认通常是 60%）。

3. **启动服务：**

   ```bash
   sudo systemctl enable --now zramswap
   ```

### 案二：使用 `zramd`（性能更激进）

如果你觉得 `zram-tools` 太温和，可以尝试 `zramd`，它在处理高负载（比如你同时开 GitLab 和浏览器）时表现更好。

1. **安装：**

   ```bash
   sudo apt install zramd
   ```
   
2. **启动：**

   ```bash
   sudo systemctl enable --now zramd
   ```

### 案三：手动加载内核模块（无需安装新包）

如果你不想安装额外的管理工具，也可以直接利用 Ubuntu 内核自带的模块。这种方式最“硬核”，适合你这种喜欢直接修改系统底层配置的性格。

1. **临时加载：**

   ```bash
   sudo modprobe zram
   # 应该指定设备数量，否则或许的指令无效。
   sudo modprobe zram num_devices=1
   ```
   
2. **配置大小并启用：**（例如设置 4G 的压缩交换空间）

   ```bash
   # 这里的 num_devices 表示创建一个 zram 设备
   echo 4G | sudo tee /sys/block/zram0/disksize
   sudo mkswap /dev/zram0
   sudo swapon /dev/zram0 -p 100
   ```

------

### 如何确认是否生效？

- 无论使用哪种方法，安装完成后运行以下命令查看：

    ```bash
    zramctl
    ```

- 或者常用的：

    ```bash
    swapon --show
    ```

你会看到一个类型为 `partition` 或 `file` 之外的 `/dev/zram0` 设备。

### 为什么这对 8GB 内存很重要？

在没有 zram 的情况下，当 8GB 内存用满，Ubuntu 会把数据写入 SSD（Swap 文件）。SSD 再快也比内存慢得多，这就是你感到“网页卡顿”的根源。 **zram 的逻辑是：** 在内存中划出一块区域，把暂时不用的数据**压缩**后再存放。因为 CPU 压缩数据的速度远高于 SSD 的读写速度，所以即便内存满了，系统依然能保持相对顺滑。

运行 GitLab 的服务器的话，尤其耗费内存，如果加上这个配置，应该能显著缓解浏览器多开网页时的“假死”现象。你可以先在一台设备上试试效果？  
※`Gitlab` 实测，即使开启内存压缩，8GB 内存依然是非常窘迫，不过也许稍微快了一些吧，很难体会到具体的优化效果。



## 2. 调整压缩内存大小

看到 `/dev/zram0` 只有 256MB，说明系统确实启用了 `zram`，但由于是默认配置，这个容量对于 8GB 内存的机器来说只是“杯水车薪”，完全不够用。  
要调整它，最有效的方法取决于你刚才安装的是哪种工具。以下是针对不同情况的调整方案：

### 案一：如果你安装了 `zram-tools` (推荐)

这是最简单的办法，通过修改配置文件即可。

1. **编辑配置文件：**

   ```bash
   sudo nano /etc/default/zramswap
   ```

2. **修改百分比或具体数值：** 找到 `PERCENTAGE=60`（表示占用物理内存的 60%）或者修改 `MAX_SIZE`。对于 8GB 内存，建议将其设置为：

   - `PERCENTAGE=60`
   - 或者直接指定 `ALLOCATION=4096` (即 4GB)

3. **重启服务应用更改：**

   ```Bash
   sudo service zramswap restart
   ```

### 案二：如果你安装的是 `zram-config` (旧版)

这个工具的逻辑比较死板，它通常根据 CPU 核心数创建多个设备（如 zram0, zram1...），每个大小很小。

1. **修改脚本逻辑：**

   ```Bash
   sudo nano /usr/bin/init-zram-swapping
   ```

2. **寻找计算公式：** 找到计算 `mem` 的那行代码。你会发现它可能是把内存总量除以核心数再除以 2。你可以直接把变量改为你想要的数值。

3. **重启系统或重新运行脚本。**

### 案三：手动暴力调整（即刻生效，无需重启）

如果你不想翻配置文件，想直接把 `zram0` 改大，可以执行以下一系列命令：

```Bash
# 1. 禁用当前的 zram 设备
sudo swapoff /dev/zram0

# 2. 重置该设备（必须重置才能改大小）
echo 1 | sudo tee /sys/block/zram0/reset

# 3. 设置新的大小（建议设为 4G，应对浏览器多开）
echo 4G | sudo tee /sys/block/zram0/disksize

# 4. 重新初始化并启动
sudo mkswap /dev/zram0
sudo swapon /dev/zram0 -p 100
```

------

### 为什么只有 256MB？

这通常是因为某些默认配置将 `zram` 的大小限制在了 **核心数 × 64MB** 或者物理内存的一个极小比例。在现在的 Web 环境下，256MB 可能连打开一个 GitLab 的 Dashboard 页面都不够压缩的。

### 调整后的建议

- **检查效果：** 再次输入 `zramctl`，你应该能看到 `DISKSIZE` 变成了 4G。
- **配合 Swappiness：** 既然有了 4GB 的高速压缩空间，建议把系统的 `vm.swappiness` 调高一点（比如设为 `100` 或 `150`），让系统更积极地把那些不常用的浏览器插件数据丢进 `zram` 压缩，从而腾出物理内存给正在操作的网页。
  - 临时调整：`sudo sysctl vm.swappiness=100`
  - 调整参照：[Ubuntu修改缓存方式](61-Ubuntu修改缓存方式.md)

调整完之后，可以再试着打开那几个“吃内存”的网页，观察一下 `zramctl` 里的 `DATA`（原始大小）和 `COMPR`（压缩后大小），你会发现 8GB 内存其实能塞下很多东西。



# 脏数据回写机制

> 优化 Linux 的脏数据回写机制（利用内存缓冲）

## 设置及查询

机械硬盘慢，我们可以让系统尽可能多地利用内存（RAM）来做缓存，把要写入磁盘的数据先攒在内存里，然后再批量写入机械硬盘。

- 调整 Ubuntu 的内核参数（在 `/etc/sysctl.conf` 中追加）：

  ```Plaintext
  vm.dirty_background_ratio = 5
  vm.dirty_ratio = 10
  ```
  *让系统更频繁、更小批量地在后台回写数据，避免瞬间爆发的大流量写入将机械硬盘彻底卡死。*

  `vm.dirty_background_ratio = 5`   
  ※当系统内存中 **有 5% 变成 dirty pages（脏页）时，Linux 内核会启动后台写回线程，把这些脏数据异步写入磁盘**。

- 查询当前设置

  ```bash
  cat /proc/sys/vm/dirty_background_ratio
  cat /proc/sys/vm/dirty_ratio
  ```

  使用 sysctl（更标准）

  ```bash
  sysctl vm.dirty_background_ratio
  sysctl vm.dirty_ratio
  ```

  输出示例：

  ```bash
  vm.dirty_background_ratio = 10
  vm.dirty_ratio = 20
  ```
  
  查看所有 dirty 相关参数：
  
  ```bash
  sysctl -a | grep dirty
  ```
  
  

## 什么是 dirty pages？

- **Dirty pages** = 已被修改但尚未写入磁盘的内存页。
- 应用程序写文件时，数据通常先写入内存（页缓存），稍后再由内核写回磁盘。

## `vm.dirty_background_ratio` 的作用

- 这是一个 **百分比阈值**。
- 当脏页占系统总内存 **超过 5%** 时，内核会启动 **后台写回（background flush）**，以较低优先级将数据写入磁盘。
- 这是一个“提前开始清理”的机制，避免脏页堆积过多。

## `vm.dirty_ratio` 的作用

当系统内存中 **dirty pages（脏页）达到 dirty_ratio%** 时：

- 内核会 **阻塞应用程序写入**（foreground writeback）
- 强制把脏页写回磁盘
- 这是一个 **硬上限**

dirty_ratio 越大 → 系统允许更多脏页堆积 → 写回更集中 → I/O 峰值更大 dirty_ratio 越小 → 更频繁写回 → 延迟更稳定，但吞吐量可能下降

## 与 `vm.dirty_ratio` 的区别（关键）

| 参数                          | 作用                   | 触发时机          |
| ----------------------------- | ---------------------- | ----------------- |
| **vm.dirty_background_ratio** | 后台开始写回           | 比较低（如 5%）   |
| **vm.dirty_ratio**            | 阻塞应用写入、强制写回 | 较高（如 10–20%） |

`dirty_background_ratio` 是 **温和阈值**：开始后台写回。

`dirty_ratio` 是 **硬性上限**：达到后应用写入会被阻塞，直到写回完成。

## 你的设置意味着什么？

`vm.dirty_background_ratio = 5`   → 当脏页达到 **5% 内存** 时，内核就会开始后台写回，属于比较积极的写回策略。

适用于：

- 数据库、日志系统等对 I/O 延迟敏感的场景
- 避免一次性积累大量脏页导致 I/O 峰值

## 不同场景的最佳 dirty_ratio

### 🖥️ **桌面 / 普通用途（SSD/HDD）**

- **10–20%**
- 你不希望 I/O 峰值太大，也不希望太频繁写盘

### 💾 **HDD 机械硬盘（尤其是低速盘）**

- **10–15%**
- HDD 写入慢，脏页堆积太多会导致卡顿

### ⚡ SSD（NVMe/SATA）**

- **15–20%**
- SSD 写入快，允许更多脏页不会造成明显卡顿

### 🗄️ 数据库 / 日志服务器（I/O 延迟敏感）**

- **10–15%（官方推荐）**
- 避免一次性大量写回造成延迟抖动

### 🧪 大量顺序写入（如备份、视频录制）**

- **20–40%**
- 允许更多脏页堆积，提高吞吐量
- 但风险是写回时 I/O 峰值更大

## 📌 你应该用多少？（给你一个明确答案）

如果你不告诉我你的机器配置，我给你一个 **通用最优解**：

### **SSD：15%**

### **HDD：10%**

### **数据库：10–15%**

### **普通桌面：10–20%**

## 📌 官方默认值对比表

| 参数                          | 默认值 | 官方建议（数据库） | 作用             |
| ----------------------------- | ------ | ------------------ | ---------------- |
| **vm.dirty_background_ratio** | 10%    | 3%                 | 后台写回阈值     |
| **vm.dirty_ratio**            | 20%    | 15%                | 前台强制写回阈值 |