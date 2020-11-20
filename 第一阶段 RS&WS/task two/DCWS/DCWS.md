# 无线控制器AC - DCWS

这里主要说明DCWS的一些安全设置与DHCP的其他设置

如果需要看DHCP的设置请点**[这里](../other/DHCP_config_note.md)**

## AP静态地址的配置

你可以通过AP的WEB界面配置，当ap获取到了dhcp的动态地址之后你可以在浏览器中输入这个ip地址，然后进入WEB界面中进入**有线配置**，将`IP获取方式`改为**Static IP**

<img src="note.assets/image-20200115174935348.png" alt="image-20200115174935348" style="zoom: 80%;" />



## 要求不能使用Trunk的情况下可以使用hybrid模式

假设题目有要求AC与AP的线路不能使用Trunk的话你需要使用hybrid模式进行配置，假设题目中用户使用的vlan为10,20，管理vlan为101

``` shell
(config)#inter e1/0/3
(config-if-Ethernet1/0/3)#sw mo hybrid
# 这里需要设置带标签头的vlan通过
(config-if-Ethernet1/0/3)#sw hybrid allowed vlan 10,20 tag
# 设置管理vlan不带标签通过
(config-if-Ethernet1/0/3)#sw hybrid allowed vlan 101 untag
# 这里设置与ac的本征vlan101
(config-if-Ethernet1/0/3)#sw hybrid native vlan 101
```

## 要求保留DHCP地址

``` shell
# 这里是保留一个范围，从哪里到哪里
(config)#ip dhcp excluded-address 172.16.10.1 172.16.10.10
```

## DCWS的有线用户指定vlan从本地DHCP上动态获取IP

``` shell
(config)#vlan 30,40,100
(config)#ip forward-protocol udp bootps
(conifg)#inter vlan 30
(config-if-vlan30)#ip helper-address 192.168.100.254
(conifg)#inter vlan 40
# 这里192.168.100.254为DCWS的本地ip
(config-if-vlan30)#ip helper-address 192.168.100.254
```

## 在DCRS上做帧中继

* 要注意如果DCRS上有做帧中继的话DCRS也需要配置hybrid模式，记得截图！

``` shell
#*** DCRS ***
(config)#ip forward-protocol udp bootps
(config)#inter vlan 10
(config-if-vlan10)#ip helper-addresss 192.168.100.254
```

## 各类DHCP安全配置

### 开启ARP抑制功能

``` shell
(config)#wireless
(config-wireless)#network 1
(config-network)#arp-suppression
```

### 开启自动强制漫游功能

``` shell
(config)#wireless
(config-wireless)#dynamic-blacklist
(config-wireless)#force-roaming mode auto
```

### 当组播的成员个数超过8个时组播M2U功能关闭

``` shell
(config)#wireless
(config-wireless)#network 1
(config-wireless)#igmp snooping m2u
(config-wireless)#m2u threshold 8
```

