# SUSE12笔记

## 问题一：初装系统，无法使用终端远程ssh

### 解决方法：

#### 修改sshd_config

```shell
vim /etc/ssh/sshd_config
# 做如下修改：
PermitRootLogin yes
PasswordAuthentication yes
```

#### 查看防火墙状态

```shell
service SuSEfirewall2 status
# 如果是开启状态则按下面操作添加规则（或者直接关闭防火墙，再重启ssh服务即可）
vim /etc/sysconfig/SuSEfirewall2
# 做如下修改
FW_SERVICES_EXT_UDP="22"
FW_SERVICES_EXT_TCP="22"
```

#### 重启ssh和SuSEfirewall2服务

```shell
service sshd restart
service SuSEfirewall2 restart
```

## 问题二：root密码忘记；如何重置密码

### 解决方法：

1. 进入grub菜单引导界面后，按e
2. 在启动项后面加 init=/bin/bash，按Ctrl-x或者F10
3. 直接进入/bin/bash界面，然后就可以准备开始修改密码了
4. 输入 mount -n / -o remount,rw （注意是逗号，不是点号）
5. 输入 /usr/bin/passwd重置root密码
6. 修改完成后输入 mount -n / -o remount,ro，将根文件系统设为原来的状态
7. 输入exit退出系统，重新启动系统，用新密码登陆。

附图：

![suse重置密码0](https://github.com/itwhs/suse/blob/main/Saved%20Pictures/%E9%87%8D%E7%BD%AEsuse%E5%AF%86%E7%A0%810.png)

![重置suse密码1](https://github.com/itwhs/suse/blob/main/Saved%20Pictures/%E9%87%8D%E7%BD%AEsuse%E5%AF%86%E7%A0%811.png)

![重置suse密码2a](https://github.com/itwhs/suse/blob/main/Saved%20Pictures/%E9%87%8D%E7%BD%AEsuse%E5%AF%86%E7%A0%812a.png)

![重置suse密码2b](https://github.com/itwhs/suse/blob/main/Saved%20Pictures/%E9%87%8D%E7%BD%AEsuse%E5%AF%86%E7%A0%812b.png)

# 问题三：什么是文件描述符/文件句柄/文件指针？它们的区别与联系是什么？

### 参考资料：

1. [句柄](https://www.jianshu.com/p/ad879061edb2)
2. [inode](https://www.jianshu.com/p/d60a2b44e78e)

# 问题四：获取指定网卡ip地址：

## 解决方法：

```shell
#比如eth0网卡
ifconfig eth0 | grep -E 'inet addr:|inet 地址:' | awk '{print $2}' | cut -c 6-


#步骤解析：
linux-ascj:~ # ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 00:0C:29:18:C6:0D  
          inet addr:192.168.3.229  Bcast:192.168.3.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe18:c60d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1495 errors:0 dropped:0 overruns:0 frame:0
          TX packets:25 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:108566 (106.0 Kb)  TX bytes:2588 (2.5 Kb)
linux-ascj:~ # ifconfig eth0 | grep -E 'inet addr:|inet 地址:' 
          inet addr:192.168.3.229  Bcast:192.168.3.255  Mask:255.255.255.0
linux-ascj:~ # ifconfig eth0 | grep -E 'inet addr:|inet 地址:' | awk '{print $2}' 
addr:192.168.3.229
linux-ascj:~ # ifconfig eth0 | grep -E 'inet addr:|inet 地址:' | awk '{print $2}' | cut -c 6-
192.168.3.229
```

## 扩展：

获取主机ip，但是无法指定是哪个网卡的ip，便于统计ip

```shell
ip addr | awk '/^[0-9]+: / {}; /inet.*global/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}'

#步骤解析：
linux-ascj:~ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:18:c6:0d brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.229/24 brd 192.168.3.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe18:c60d/64 scope link 
       valid_lft forever preferred_lft forever

linux-ascj:~ # ip addr | awk '/^[0-9]+: / {}; /inet.*global/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}'
192.168.3.229
linux-ascj:~ # ip addr | awk '/^[0-9]+: / {}; /inet.*eth0/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}'
192.168.3.229
#可以试试该eth0字段，看看是否有ip变动
```


# 问题五：RH2288HV5服务器安装suse系统，无法显示启动项

### 原因解析：是挂载的虚拟CD安装系统，服务器bios默认是UEFI引导，需要改成Legacy Boot模式才能正常安装系统

## 解决办法：

1.把Boot Type改为Legacy Boot

![RH2288HV5bios1](https://github.com/itwhs/suse/blob/main/Saved%20Pictures/RH2288HV5bios1.png)

2.进Advanced里面，找到Console Redirection，点进去，改为Disabled。保存重启。（如果不改这个，引导没显示，无法进入装好的系统）

Console Redirection网上说是串口重定向，把指定的物理串口或虚拟串口的数据映射到指定的系统串口。

![RH2288HV5bios2](https://github.com/itwhs/suse/blob/main/Saved%20Pictures/RH2288HV5bios2.png)
![RH2288HV5bios3](https://github.com/itwhs/suse/blob/main/Saved%20Pictures/RH2288HV5bios3.png)
