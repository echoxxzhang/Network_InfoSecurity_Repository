2018省赛，DCRS&DCWS做题记录

## DCRS

1. vlan&端口
2. 静态路由
3. dhcp
4. 安全配置

将2口双向流量镜像至4口

``` shell
monitor session 1 source inter e1/0/2 both
monitor session 1 destination inter e1/0/4
```

MAC地址学习数量限定

``` shell
int e1/0/*
sw port-security
sw port-security maximun 5
sw port-security violation restrict
```

接入DCRS Eth4，仅允许IP地址192.168.20.1-10为源的数据包为合法包，以其它IP地址为源地址，交换机直接丢弃

``` shell
am enable
inter e1/0/4
am port
am ip-port 192.168.20.1 10
```



开启防ARP扫描，如果超过50个则认定是攻击，shutdown

``` shell
anti-arpscan enable
anti-arpscan port-based threshold 50
# 设置超级信任端口，避免误封
inter e1/0/1
anti-arpscan trust super
```

## DCWS

* DCWS配置VLAN110为管理VLAN, AP动态方式注册到AC，AC管理IP为192.168.110.254； 数据VLAN为111和222，分别下发网段192.168.111.0/24，192.168.222.0/24，网关为最后一个可用IP，DNS:8.8.8.8，需要排除网关，地址租约为2天； AP静态方式注册到AC；

* 配置默认路由，使无线用户可以访问Internet;

* 配置2.4G频段下工作，使用802.11g协议；

``` shell
vlan 110,111,222,66
int vlan 110
ip add 192.168.110.254 255.255.255.0
int vlan 111
ip add 192.168.111.254 255.255.255.0
int vlan 222
ip add 192.168.222.254 255.255.255.0
int vlan 66
ip add 192.168.66.253 255.255.255.0
# 配置默认路由
ip route 0.0.0.0 0.0.0.0 192.168.66.254
int e1/0/24
sw mo tr
# 配置本证vlan
int e1/0/23
sw mo trunk
sw trunk na vlan 110
# 开启dhcp服务
service dhcp
ip dhcp pool vlan110
network-address 192.168.110.0 255.255.255.0
default-router 192.168.110.254
ip dhcp pool vlan111
network-address 192.168.111.0 255.255.255.0
default-router 192.168.111.254
dns 8.8.8.8
default-router 192.168.111.254
lease 2 0 0
ip dhcp pool vlan222
network-address 192.168.222.0 255.255.255.0
default-router 192.168.222.254
dns 8.8.8.8
default-router 192.168.111.254
lease 2 0 0
# 配置无线
wireless
no auto-ip-assign
enable
ap authentication none
discovery vlan-list 110
static-ip  192.168.110.254
ap profile 1
hwtype 29
radio 1
mode g
# 应用无线配置
end
wireless ap profile apply 1
```

注

DCRS与DCWS的端口需要配置trunk与trunk native vlan