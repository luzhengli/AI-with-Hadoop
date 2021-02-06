#! https://zhuanlan.zhihu.com/p/349740981
>    学过计网，也知道内网 、外网、ip 地址、ping 这些概念，但是一旦需要自己配置网络环境（比如让虚拟机访问互联网或者虚拟机和宿主机之间相互访问）时就头大，完全不知道怎么动手。你有类似的烦恼吗？
>
>    **本文的目的**是让计网知识不扎实的朋友可以实现虚拟机下 CentOS7 系统（类 Unix 系统下的网络配置也类似）的网络配置。本文会细致的讲解怎么做，以及为什么这样做（适当深入）。首先会回顾一些计网知识，不需要的可跳过，否则请耐心阅读。

# 网络配置的目标

>   先搞明白网络配置到底要干嘛。

**目标一**：虚拟机可以访问外网（实现外网连通，即可以正常浏览互联网上的网站）

**目标二**：虚拟机和宿主机可以相互访问（实现内网连通，即 宿主机可以ping通虚拟机，虚拟机也可以ping通宿主机）

# 知识回顾

>   我会用**比较粗糙**的语言快速回顾一下与网络配置有关的计网知识，需要进一步了解请看参考或自行 Google。



Q：一般家庭的设备是怎么联网的？

A：一个家庭有多台设备，它们构成一个局域网（即私有网络，也叫**内网**）。内网中每台设备有一个**私有地址**（私有网络下用的 ip 地址）。

然而 TCP/IP 规定了私有地址的报文不能进入**互联网**（专用名词：Internet，首字母大写。一般也把局域网中设备访问互联网称作访问**外网**）。为了能够访问外网，无线路由器（即设备的**默认网关**）提供了一种服务，它能够接受设备的请求报文，然后把它们的私有地址改为**公有地址**（Internet 中是唯一的），然后转发具有公有地址的报文给相应的服务器。服务器接受到请求后返回响应给路由器，路由器再通过**端口**区分应该把响应交给哪个设备。

---

Q：PING（ping）是什么？

A：一般用于检测网络的连通性，如果 A 能 ping 通 B，则表示 A 可以向 B 发送 **ICMP** 报文（ICMP 是一种网络层协议，是 ping 的工作核心，知道有这回事就行）并且接受到响应。例如，如果虚拟机可以 ping 通百度，则表示虚拟机可以访问百度这个网站。



---

Q：DHCP 是什么？

A：如今很多机器启动时是没有 ip 地址（这里指私有地址）的，为了得到一个私有地址，新启动的机器会通过 DHCP 这种服务获得一个临时的私有地址。这种 ip 地址是动态的。DHCP 模式下，每次重启，机器的 ip 地址都可能不同，这个特点不利于远程控制等操作，后面会讲解如何给机器设置一个静态 ip 地址。



---

Q：虚拟机中的桥接模式是什么？

A：我们让虚拟机使用桥接方式连接 Internet，意思是把虚拟机看成是独立于宿主机的一台机器，给它一个新的私有地址。然后虚拟机就可以通过无线路由器转发的方式（即 NAT）访问外网。





# 具体步骤



## 目标一

1.  把虚拟机网络设为桥接模式

    ![image-20210206214007124](https://gitee.com/llillz/images/raw/master/image-20210206214007124.png)

2.  利用 DHCP（使用 dhclient 命令），为虚拟机分配一个内网中可用的私有地址

    ```bash
    $ ifconfig
    ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.3.22  netmask 255.255.255.0  broadcast 192.168.3.255
            省略部分输出
    ```

3.  使用 vi （vim）修改虚拟机网络配置文件，固定虚拟机的 ip 地址（私有地址）

    ```bash
    $ sudo vi /etc/sysconfig/network-scripts/ifcfg-ens33 # 修改网卡配置文件需要获取管理员权限！
    ```

    配置文件修改后如下（# 开始的表示注释）所示：

    ```
    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    # use static ip
    BOOTPROTO=static 
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_FAILURE_FATAL=no
    IPV6_ADDR_GEN_MODE=stable-privacy
    NAME=ens33
    UUID=ec972809-137d-444e-98ee-633728c1885c
    DEVICE=ens33
    # change to yes
    ONBOOT=yes
    # static ip by using DHCP
    IPADDR=192.168.3.22
    NEWMASK=255.255.255.0
    # gateway---your wireless router ip
    GATEWAY=192.168.3.1
    # default dns is Tecent DNS
    DNS1=119.29.29.29
    ```

    -   **注意**：除了注释部分可以不写，其余每行变量必须都有，但每行**变量的值有些需要自定义**。下面详细介绍一下

        -   `BOOTPROTO=static`：为虚拟机指定一个静态的 ip（默认情况下会由 DHCP 分配一个 ip，也就是动态 ip，但是这不利于我们以后使用远程连接之类的服务）

        -   `ONBOOT=yes`：设置启动虚拟机时自动连接网络

        -   `IPADDR=X.X.X.X`：虚拟机的静态 ip（虚拟机持有的私有地址），有两种获得方法

            1.  【推荐】由程序自动分配一个合法 ip：首先输入命令 dhclient 获得一个私有地址，等待执行后输入命令 ifconfig 查询这个 ip 地址

                ![image-20210206230604787](https://gitee.com/llillz/images/raw/master/image-20210206230604787.png)

            2.  自己指定一个不与当前内网中设备冲突的 ip。

        -   `NEWMASK=X.X.X.X`：子网掩码，这个没啥好说，跟宿主机的网卡的子网掩码一样即可（可以在宿主机中查询到）。

        -   `GATEWAY=X.X.X.X`：默认网关的 ip 地址。如果你是家里通过无线路由器上网的，那么这个就是你的无线路由器的 ip 地址。家庭局域网中的所有设备发出的请求都将通过它转发到互联网上（即 NAT）。查询方法由两种
            1.  直接在路由器背面找
            2.  在宿主机中通过 ipconfig 命令查看
            
        -   `DNS1=X.X.X.X`：默认 DNS 服务器的 ip 地址。指定一个 DNS 服务器的 ip 地址用于 DNS 解析。许多公司都会提供自己的公共 DNS 解析服务，选择一个你觉得上网比较快的就行。这里用的是腾讯提供的 DNS。

4.  刷新网卡或重启使得修改的网络配置生效。这里输入命令直接重启网卡，避免了重启机器。

    ```bash
    $ systemctl restart network.service
    ```

5.  最后测试一下。虚拟机使用 ping 命令尝试访问 Internet （例如 ping qq.com）或直接浏览器访问一下网站

    ![image-20210206204446183](https://gitee.com/llillz/images/raw/master/image-20210206204446183.png)

    虚拟机现在可以访问外网了！



## 目标二

接下来我们要确定两件事：

1.  **宿主机能否 ping 通虚拟机**：我们打开宿主机的 cmd，输入

    ```cmd
    C:\Users\luzhe>ping 192.168.3.22
    
    正在 Ping 192.168.3.22 具有 32 字节的数据:
    来自 192.168.3.22 的回复: 字节=32 时间<1ms TTL=64
    来自 192.168.3.22 的回复: 字节=32 时间<1ms TTL=64
    来自 192.168.3.22 的回复: 字节=32 时间<1ms TTL=64
    来自 192.168.3.22 的回复: 字节=32 时间<1ms TTL=64
    
    192.168.3.22 的 Ping 统计信息:
        数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
    往返行程的估计时间(以毫秒为单位):
        最短 = 0ms，最长 = 0ms，平均 = 0ms
    ```

    可见宿主机能够 ping 通虚拟机

2.  **虚拟机能否 ping 通宿主机**：打开虚拟器，在 bash 下输入

    ![image-20210206214547237](https://gitee.com/llillz/images/raw/master/image-20210206214547237.png)

    可见**虚拟机无法 ping 通宿主机**（这里的问题不是每个人都会遇到，不过也可以看下去，万一以后遇到了呢？）



**现在我们的问题变成了：桥接模式下，虚拟机无法 ping 通宿主机**。这一般是由于宿主机的防火墙默认禁止 ICMP 服务，而虚拟机 ping 宿主机本质就是再向它发送 ICMP 报文，因此失败也就可以理解了。为了解决这个问题，我们**需要修改宿主机防火墙的入站规则，即让宿主机允许被 ping（宿主机能接受 ICMP 报文）**。



**修改宿主机防火墙入站规则步骤如下：**

1.  搜索“高级安全”，选择“Windows Defender 防火墙”

    <img src="https://gitee.com/llillz/images/raw/master/image-20210206220032831.png" alt="image-20210206220032831" style="zoom:50%;" />

2.  点击“入站规则”，找到“文件和打印机共享（回显请求）”，按下图所示进行修改

    ![image-20210206215827757](https://gitee.com/llillz/images/raw/master/image-20210206215827757.png)

3.  进入虚拟机终端下重新测试

    ![image-20210206215918762](https://gitee.com/llillz/images/raw/master/image-20210206215918762.png)

    可以看到现在虚拟机可以正常 ping 通宿主机了



# 部分参考

1.  https://www.bilibili.com/video/BV1bA411b7vs
2.  https://www.jianshu.com/p/4a6294445473
3.  https://blog.csdn.net/FBB360JAVA/article/details/102707812
4.  https://www.zhihu.com/question/304991342/answer/549013567