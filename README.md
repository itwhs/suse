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

