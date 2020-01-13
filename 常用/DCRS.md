# DCRS命令

比赛的时候截图可以截操作步骤，也可以截sh run的结果，记得在最后面放sh run的源码

## 基础

### 配置enable密码

``` shell
(config)#enable password abc
```

### 添加一个用户abc密码abc

``` shell
# 添加用户
(config)#username abc privilege 15 password abc
```

### 关闭WEB管理

``` shell
# 关闭WEB管理
(config)#no ip http server
```

### 开启SSH管理功能，仅允许使用colsole，ssh和telnet管理设备

``` shell
# 开启ssh管理功能
(config)#ssh-server enable
# 允许colsole口管理设备
(config)#authentication line vty login local
# 开启telnet
(config)#telnet-server enable
```

### 开启SNMP功能，只读字符串为public，读写字符串为private，网管服务器连接在服务器区，ip地址为192.168.1.2

``` shell
(config)#snmp-server enable
(config)#snmp-server community ro public
(config)#snmp-server community rw private
(config)#snmp-server securityip 192.168.1.2
```



下列大部分配置都来自题目的笔记

## 在e1/0/9口开启广播风暴控制，参数为400pps

``` shell
(confug)#inter e1/0/9
(config-if-ethernet1/0/9)#storm-control broadcast 400
```



## 配置环路检测时间

配置存在环路时的检测时间间隔为30秒，不存在环路时的检测时间间隔为10秒

``` shell
hostname(config)# loopback-detection interval-time 30 10
```



## 限制端口学习MAC

为终端产生防止 MAC 地址防洪攻击，请配置端口安全，
已划分 VLAN 的端口最多学习到 5 个 MAC 地址，发生违规阻止后续违规流量
通过，不影响已有流量并产生 LOG 日志；

``` shell
(config)#inter e1/0/1
// 限制学习到5个MAC
(config-if-ethernet1/0/1)#switchport port-security
(config-if-ethernet1/0/1)#switchport port-security maximum 5
(config-if-ethernet1/0/1)#switchport port-security violation restrict
// 你也可以使用缩写
(config-if-ethernet1/0/2)#sw port
(config-if-ethernet1/0/2)#sw port max 5
(config-if-ethernet1/0/2)#sw port v rest
...
```

连接 PC1 的接口为专用接口，限定只允许 PC1 的 MAC 地址可以连接；

``` shell
//限定PC1
(config-if-ethernet1/0/6)#switchport port-security
(config-if-ethernet1/0/6)#switchport port-security mac-address 9c-5c-8e-37-31-98
```


## 流量镜像
将某个端口的双向流量镜像至NetLog进行监控和分析

``` shell
// rx是改接口接受的流量，tx是该接口发送的流量
(config)#monitor session 1 source interface Ethernet1/0/2 tx
(config)#monitor session 1 source interface Ethernet1/0/2 rx
// 将E1/0/2的流量镜像至E1/0/4口
(config)#monitor session 1 destination interface Ethernet 1/0/4
```

## ARP

### 开启防ARP扫描功能

单位时间内端口收到ARP数量超过50便认定是攻击DOWN掉此端口

``` shell
// 开启防ARP端口扫描
(config)#anti-arpscan enable
// 限制数量超过50
(config)#anti-arpscan port-based threshold 50
(config)#inter e1/0/1
(config-if-Ethernet1/0/1)#anti-arpscan trust supertrust-port
```

### 在e1/0/1-10口开启ARP保护功能防护网管发出欺骗报文

``` shell
# 开启ARP保护
(config)#inter e1/0/1-10
(config-if-port-range)#arp-guard ip 192.168.30.126
(config-if-port-range)#no shut
```

### 在e1/0/11上配置指定mac1不能访问指定mac2

``` shell
# 创建mac访问控制列表
(config)#mac-access-list extended name
(config-mac-ext-nac1-name)#deny host-source-mac 11-11-11-11-11-11 host-destination-mac 22-22-22-22-22-22
(config-mac-ext-nac1-name)#permit any-source-mac any-destination-mac
(config-mac-ext-nac1-name)#q
(config)#firewall enable
# ACL绑定到端口
(config)#inter e1/0/11
(config-if-ethernet1/0/11)#mac access-group name in
(config-if-ethernet1/0/11)#no shut
```

### vlan下开启ARP阈值

这里假设阈值为50

``` shell
(config)#inter vlan 30
(config-if-vlan30)#ip arp dynamic maximun 50
```

## 指定vlan开启ip dhcp snooping并在接口下绑定用户

这里假设绑定vlan40

``` shell
(config)#ip dhcp snooping enable
(config)#ip dhcp snooping binging enable
# 假设e1/0/1是vlan20的不需要设置则设置安全域
(config)#inter e1/0/1
(config-if-Ethernet1/0/1)#ip dhcp snooping trust
# 假设e1/0/2是vlan40
(config-if-Ethernet1/0/1)#inter e1/0/2
(config-if-Ethernet1/0/2)#ip dhcp snooping binding user-control
```



## 环路检测

此处防止vlan200接口下的单端口环路，并配置存在环路时的检测时间间隔未30秒，不存在环路的检测时间为为10秒

``` shell
(config)#loopback-detection interval-time 30 10
(config)#inter e1/0/10
(config-if-Ethernet1/0/10)#loopback-detection specified-vlan 200
(config-if-Ethernet1/0/10)#loopback-detection control shutdown
```



## 配置认证服务器

按题意需要在e1/0/10口开启DOT1X认证，配置认证服务器，IP地址是172.16.100.40，radius key是dcn2018

``` shell
(config)#radius-server authentication host 172.16.100.40 key 0 dcn2018
(config)#aaa enable
(config)#inter e1/0/10
(config-if-Ethernet1/0/10)#dot1x enable
(config-if-Ethernet1/0/10)#dot1x port-method portbased
```



## 配置指定vlan自动获取IP

VLAN20、vlan30、vlan10 用户采用动态获取 IP 地址方式，DHCP 服务器在AC 上配置(10.0.0.6)，前十个地址为保留地址，vlan40 用户也动态获取 ip，DHCP server(10.0.0.1)为 DCFW。

在**DCRS**上配置：

``` shell
# 记得开启dhcp服务，否则会配置失败
(config)#server dhcp
(config)#ip forward-protocal udp bootps
(config)#inter vlan 10
(config-if-vlan10)#ip helper-address 10.0.0.6
......# vlan20 vlan30的配置相同这里不重复写
(config)#inter vlan 40
(config-if-vlan40)#ip helper-address 10.0.0.1
```

## 指定vlan2时间(工作日8-18点)内访问vlan3段的ip

这里假设
vlan2的ip段为172.16.100.0
vlan3的ip段为192.168.100.0
在e1/0/10口上做绑定

``` shell
(config)#time-range name
(config-time-range-work)#periodic weekdays 08:00:00 to 18:00:00
(config)#ip access-list extended work
(config-ip-ext-nacl-work)#permit ip 172.16.100.0 0.0.0.255 192.168.100.0 0.0.0.255 time-range work
(config)#inter e1/0/10
(config-if-Ethernet)#ip access-group work in
```

## 在交换机上配置DHCP server helper

配置交换机使VLAN110用户可以通过DHCP方式获得ip，地址池为pool-vlan110，dns为114.114.114.114和8.8.8.8，租期为2天，vlan110最后的20个地址不被动态分配出去

``` shell
(config)#server dhcp
(config)#ip dhcp pool pool-vlan110
(dhcp-pool-vlan110-config)#network-address 192.168.110.0 255.255.255.0
(dhcp-pool-vlan110-config)#default-router 192.168.1.254
(dhcp-pool-vlan110-config)#dns-server 114.114.114.114 8.8.8.8
(dhcp-pool-vlan110-config)#lease 2
(dhcp-pool-vlan110-config)#q
(config)#ip dhcp excluded-address 192.168.234 192.168.110.254
```

# 大概题型

| 类型    | 具体题目                                         | 难度 |
| ------- | ------------------------------------------------ | ---- |
|         | 基本的VLAN与路由配置                             | *    |
|         | 流量镜像端口                                     | -    |
|         | enable密码、用户新建、ssh、telnet、control的配置 | *    |
| ARP类型 | 防ARP扫描、ARP阈值、防ARP欺骗                    | -    |
|         | 指定MAC不能访问指定MAC                           | -    |
|         | 广播风暴抑制                                     | -    |
|         | 防MAC泛洪攻击                                    | --   |
|         | 防环路检测                                       | -    |
|         | ACL的配置                                        | -    |
|         | 少见情况                                         |      |
|         | RIP的配置                                        | -    |
|         | dhcp snooping、ip snooping                       | --   |
| 重定向  | 重定向指定之间内访问                             | --   |
| snmp    | snmp的服务器的配置                               | -    |
| STP     | MSTP的配置                                       | --   |

