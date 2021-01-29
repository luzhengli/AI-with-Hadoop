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



