# 复制与粘贴

在命令行中。双击字符串或是利用光标可以选中字符串。选中字符串后，点击鼠标中键（按压滚轮），可以直接粘贴选中的内容。



# 切换工作路径

命令 `cd -` 可以返回上次的路径。

# 组合使用多个命令

多个命令可以写在同一行，用分号（；）分割。这样系统会依次执行这些命令。



例如：

```bash
[luyao@localhost test]$ cd ~; ls; cd -
Desktop    Downloads  perl5     Public     test
Documents  Music      Pictures  Templates  Videos
/home/luyao/test
[luyao@localhost test]$ 
```



# 系统监测

## 检测设备状态

命令：`tail -f /var/log/message`

解释： **`/var/log/message` 文件是一个系统日志**，设备的挂载和卸载都会记载在这个文件中。上述命令截取了该日志文件的最后几行内容，一旦设备状态发生改变，这个日志内容就会更新，我们就能捕捉到设备状态的变化。

实例：虚拟机下 CentOS7 监视 USB 设备（这里我在宿主机上插入了一个 USB-type C 口的扩展坞，`/var/log/message`  文件产生了如下新的内容）

```
Feb  7 19:59:17 localhost dbus[1175]: [system] Activating via systemd: service name='net.reactivated.Fprint' unit='fprintd.service'
Feb  7 19:59:17 localhost systemd: Starting Fingerprint Authentication Daemon...
Feb  7 19:59:17 localhost dbus[1175]: [system] Successfully activated service 'net.reactivated.Fprint'
Feb  7 19:59:17 localhost systemd: Started Fingerprint Authentication Daemon.
Feb  7 20:00:01 localhost systemd: Created slice User Slice of root.
Feb  7 20:00:01 localhost systemd: Started Session 19 of user root.
Feb  7 20:00:01 localhost systemd: Removed slice User Slice of root.
```

观察前 5 行内容可以发现，Linux 激活了一个名为 `net.reactivated.Fprint` 的设备。



