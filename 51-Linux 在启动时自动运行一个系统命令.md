# Linux 在启动时自动运行一个系统命令

有时它对于将自定义命令添加到启动进程中将是很有用的，这样您的计算机在每次启动时都可以执行它。

- 以管理权限编辑 crontab (参见 第1.3.3节 ― root 用户和 sudo 命令)：
  
    ```
    sudo crontab -e
    ```
- 插入以下内容： 
  
    ```
    @reboot /home/user/command
    ```
    
    用您命令的绝对路径替换掉 **/home/user/command**。
- 保存该文件并退出

## 参照资源

- [Ubuntu桌面入门指南](https://wiki.ubuntu.org.cn/Ubuntu%E6%A1%8C%E9%9D%A2%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97#3.8.1..E2.80.83.E5.9F.BA.E6.9C.AC.E7.BC.96.E8.AF.91.E5.99.A8)