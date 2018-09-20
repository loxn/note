<https://blog.csdn.net/wyh9459/article/details/53559993>

一.宿主机ping通虚拟机

使用host-only模式

![img](D:\笔记\img\1)

![img](D:\笔记\img\2)

全局设置不用管

![img](D:\笔记\img\3)

vi /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0

HWADDR=08:00:27:7C:13:31

TYPE=Ethernet

ONBOOT=yes

NM_CONTROLLED=yes

BOOTPROTO=static

IPADDR=192.168.137.10

NETMASK=255.255.255.0

GATEWAY=192.168.137.1

DNS1=192.168.137.1

二.配置host-only访问外网

![img](D:\笔记\img\4)