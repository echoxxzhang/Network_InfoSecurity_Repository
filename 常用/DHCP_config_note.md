# DHCP配置

首先需要在交换机上配置好相应的vlan、ip配置具体以题目为准，这里假设单纯在AC上配置DHCP 

！注意此处为单个无线，还有两个无线与帧中继的情况

vlan10: 192.168.10.254/24

vlan 100: 192.168.100.254/24

AP获取的ip为: 192.168.10.0/24

用户获取的ip是: 192.168.100.0/21

``` shell
(config)#vlan 10, 100
(config)#inter vlan 10
(config-if-vlan10)#ip add 192.168.10.254 255.255.255.0
(config-if-vlan10)#inter vlan 100
(config-if-vlan100)#ip add 192.168.100.254 255.255.255.0
(config)#inter e1/0/2
(config-if-ethernet1/0/2)#sw mo trunk
// 这里配置本征vlan是给ap使用的地址
(config-if-ethernet1/0/2)#sw trunk native vlan 10
// 开启DHCP服务！！！重要
(config)#server dhcp
// 这里的ap为这个地址池的名字
(config)#ip dhcp pool ap
// 地址池添加
(dhcp-ap-config)#network-address 192.168.10.0 255.255.255.0
// 网关设置
(dhcp-ap-config)#default-router 192.168.10.254
(config)#ip dhcp pool vlan100
(dhcp-ap-config)#network-address 192.168.100.0 255.255.255.0
(dhcp-ap-config)#default-router 192.168.100.254
(dhcp-ap-config)#dns-server 8.8.8.8
// 查看ac的ip获取，这一步可以跳过（有把握的话），也可以先去做其他的部分
#show ip dhcp binding
// 查看ap的硬件类型，ap的型号在机器后面查看，对应序号即可
#show vendor
(config)#wireless
// 关闭自动选取功能
(config-wireless)#no auto-ip-assign
// 配置静态ac管理地址
(config-wireless)#static-ip 192.168.10.254
// 认证方式
(config-wireless)#ap authentication none
// 指定发现vlan，这里的vlan为管理vlan，可不配置
(config-wireless)#discovery vlan-list 10
// 开启无线功能
(config-wireless)#enable
// 查看是否成功，成功则会显示Managed Success
#show wireless ap sta
(config)#wireless
(config-wireless)#network 1
// 设置无线的ssid为admin
(config-network)#ssid admin
// 设置密文认证模式为wpa
(config-network)#security mode wpa-personal
// 设置用户获取vlan
(config-network)#vlan 100
// 配置wpa密码，长度至少为8位
(config-network)#wpa key 12345678
(config-wireless)#exit
// 即使是两个无线默认都使用1即可
(config-wireless)#ap profile 1
// 此处的29为ap的硬件类型，上面有提及，对应序号即可
(config-ap-profile)#hwtype 29
// 应用
#wireless ap profile apply 1
// 如果成功则会与上面相同
#show wireless ap status
// ---其他的配置----
(config)#wireless
(config-wireless)#network 1
// 隐藏SSID
(config-network)#hide-ssid
// 流控
(config-network)#client-qos enable
// 设置下载速度为2m，即2048
(config-network)#client bandwidth-limit down 2048
// 设置上载速度为1m，即1024
(config-network)#client bandwidth-limit up 1024
(config-network)#exit
// 设置AC断开网络时AP还能正常工作
(config-wireless)#ap profile 1
(config-ap-profile)#ap escape
// 配置2.4频段下工作使用802.11g
(config--ap-profile)#radio 1
(config--ap-profile)#mode g
// 通过黑名单技术禁止某mac通过无线上网
(config-wireless)#mac-authentication-mode black-list
(config-wireless)#known-client 68-a3-c4-e6-a1-be action deny
```

### 帧中继

一般帧中继会在DCRS上做，DHCP在DCWS上，用户获取ip过程为：AP>DCWS(ip找RS)>DCRS(找DHCP)>DCWS(给予ip)>DCRS(转发给AP)>DCWS(转发给AP)>AP

#### DCWS

``` shell
// 在DCWS上的dhcp配置使其交给DCRS，记得要在通向DCRS的口配置trunk
(config)#ip forward-protocol udp bootps
(config)#ip route 0.0.0.0 0.0.0.0 192.168.200.253
```

[](note.assets/两个无线/帧中继/AC.text )

#### DCRS

``` shell
// 在DCRS上配置
(config)#ip forward-protocol udp bootps
(config)#int vlan 10
(config-if-vlan10)# ip helper-address 192.168.10.254
```

[](note.assets/两个无线/帧中继/RS.text )

### 两个无线（非帧中继的模拟题）

出现两个无线这种情况的话你将需要用到两个network，两个network都对应一个ap profile 1里面的radio1和radio2，切记要开启(enable)他们的vap1

为了防止弄错下面贴上配置

#### DCWS

[](note.assets/两个无线/模拟题/AC.text )

#### DCRS

[](note.assets/两个无线/模拟题/RS.text )

注：此配置中交换机与无线控制器任务2中是有几道题目是错误的，所以参考无线的配置即可

## AP静态地址的配置

你可以通过DCWS telnet上AP，AP的初始化地址为192.168.1.10

``` shell
# 静态地址的设置
DCN-WLAN-AP#set management static-ip 192.168.1.100 static-mask 255.555.255.0
# 网关的设置
DCN-WLAN-AP#set static-ip-route gateway 192.168.1.1
```

