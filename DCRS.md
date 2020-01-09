# DCRS命令

下列大部分配置都来自题目的笔记

## 配置环路检测时间

配置存在环路时的检测时间间隔为30秒，不存在环路时的检测时间间隔为10秒

hostname(config)# loopback-detection interval-time 30 10

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



## 开启防ARP扫描功能

单位时间内端口收到ARP数量超过50便认定是攻击DOWN掉此端口

``` shell
// 开启防ARP端口扫描
(config)#anti-arpscan enable
// 限制数量超过50
(config)#anti-arpscan port-based threshold 50
(config)#inter e1/0/1
(config-if-Ethernet1/0/1)#anti-arpscan trust supertrust-port
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
(config)#server dhcp
(config)#ip forward-protocal udp bootps
(config)#inter vlan 10
(config-if-vlan10)#ip helper-address 10.0.0.6
// vlan20 vlan30的配置相同这里不重复写
(config)#inter vlan 40
(config-if-vlan40)#ip helper-address 10.0.0.1
```

