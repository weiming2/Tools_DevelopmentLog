解决方法：
第一步：编写 rc-local.service 文件。
直接将文件夹里面的 rc-local.service 放入到/etc/systemd/system/rc-local.service，确定位置即可，不需要改动文件

第二步：激活 rc-local.service。
sudo systemctl enable rc-local.service

第三步：手动创建 /etc/rc.local，并赋予执行权限。
ubuntu 18.04 系统默认已经将 /etc/rc.local 文件移除了，因此，我们需要手动创建一个，并将需要开机执行的命令写入到文件中
sudo chmod +x /etc/rc.local
rc.local里面的sh文件需要有执行权限

 第四步：启动这个服务并查看它的状态。
sudo systemctl start rc-local.service
sudo systemctl status rc-local.service

可以看到绿色的active 字眼
第五步：reboot 重启验证。