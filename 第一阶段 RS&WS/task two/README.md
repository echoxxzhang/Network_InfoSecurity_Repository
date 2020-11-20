# RS&WS记录

## 前期准备

- `language chinese` 可以设置中文
- 在纸上把需要用到vlan的写上
- `show run`后创建一个记事本在上面配置

如果是一个设备可以接入多个端口则设定一个固定端口即可

## RS基础

- 基础配置

  - 配置vlan时

    ```jsx
    vlan 2,10,20,30,66,100,101,200
    ```

  - 配置int vlan

    ```jsx
    int vlan 2
    ip add 192.168.2.1 255.255.255.0
    int vlan 10
    ip add 192.168.10.1 255.255.255.0
    ......
    ```

  - 配置端口（如果遇到DCWS则单独配置Trunk）

    ```jsx
    int e1/0/2
    sw mo acc
    sw acc vlan 2
    int e1/0/10
    sw mo acc
    sw acc vlan 10
    ......
    ```

- enable 密码

# RS安全

## 远程管理方式

### telnet

交换机开启远程管理，使用 telnet方式账号为 DCN2018，密码为 123456

```bash
# 使能telnet
telnet-server enable
# 创建用户，权限默认15
username DCN2018 privilege 15 password 123456
```

### ssh

答案文档记得放`show run`中使能与创建的命令

交换机开启远程管理，使用 SSH 方式账号为 DCN2018，密码为 123456

```bash
# 使能ssh
ssh-server enable
# 创建用户，权限默认15
username DCN2018 privilege 15 password 123456
```

### arp端口扫描

- arp防止端口扫描

  ```jsx
  # 使能防扫描
  anti-arpscan enable
  # 限制数量50
  anti-arpscan port-based threshold 50
  ```

### arp欺骗

为了防止 vlan40 网段 arp 欺骗，需要在交换机上开启 ip dhcp snooping,并在接口下绑定用户

```bash
# 使能dhcp snooping
ip dhcp snooping enable
# 使能dhcp snooping binding绑定
ip dhcp snooping binding enable

# 防止vlan40的端口进行arp欺骗，（6-9口绑定了vlan40）
int e1/0/6-9
# 在接口下绑定用户，如果没使能binding绑定则会报错
ip dhcp snooping binding user-control
# 由于RS为dhcp服务器，所以来自SW的dhcp请求需要被通过（1-2口）
# 则要在端口1-2设置trust信任端口
int e1/0/1-2
ip dhcp snooping trust
```

### 端口流量镜像

将连接 DCFW 的双向流量镜像至 Netlog 进行监控和分析；遇到这种题记得分析源端口与目的端口

```jsx
# 假设DCFW为e1/0/2，Netlog为e1/0/4
# 源端口，没要求读写则不用配置tx/rx
monitor session 1 source int e1/0/2
# 目的端口
monitor session 1 source int e1/0/4
```

### 环路检测

防止来自 vlan200 接口下的单端口环路，并配置存在环路时的检测时间间隔为 30 秒，不存在环路时的检测时间间隔为 10 秒

```jsx
# 全局模式下配置存在与不存在环路检测时间
loopback-detection interval-time 30 10
# 进入vlan200接口下的端口配置
loopback-detection specified-vlan 200
# 如果题目没有要求则默认配置shutdown
loopback-detection control shutdown
```

### MAC地址学习

为终端产生防止 MAC 地址防洪攻击，请配置端口安全，已划分 VLAN 的端口最多学习到 5 个 MAC 地址，发生违规阻止后续违规流量通过，不影响已有流量并产生 LOG 日志；连接 PC1 的接口为专用接口，限定只允许 PC1 的 MAC 地址可以连接；

```jsx
# 已划分VLAN的端口最多学习到5个 MAC 地址
# 需要提前开启学习功能
mac-address-learning cpu-control
# 假设1、2、7为已划分vlan端口，PC1为6端口，所以不做最大配置
int e1/0/1-2,7
# 使能端口安全
switchport port-security
# 最多学习5个MAC地址
switchport port-security maximum 5
# 发生违规组织后续违规流量
switchport port-security violation restrict
# 限定只允许PC1的MAC地址可以连接
int e1/0/6
switchport port-security
switchport port-security mac-address 9c-5c-1c-22-22-44
```

### 配置认证服务器

为了控制接入网络 PC，需要在交换 Eth1/0/10 口开启 DOT1X 认证，配置认
证服务器，IP 地址是 172.16.100.40，radius key 是 dcn2018

```jsx
# 开启dot1x认证
dot1x enable
# 进入e1/0/10开启dot1x认证
int e1/0/10
dot1x enable
# 开启dot1x端口认证
dot1x port-method portbased
# 配置认证服务器，IP 地址是 172.16.100.40，radius key 是 dcn2018
radius-server authentication host 172.16.100.40 key 0 dcn2018
# 开启aaa认证配置认证服务器
aaa enable
```

### snmp

- SNMP V1/V2

  配置snmp读取时的团体字符，只读团体字符串为public，读写团体字符串为private

  ```jsx
  # 使能snmp服务
  snmp-server enable
  # 设置只读团体字符串为public
  snmp-server community ro public
  # 设置读写团体字符串为private
  snmp-server community rw private
  
  # 配置snmp安全IP（网管主机）1.1.1.10
  snmp-server securityip 1.1.1.10
  # 配置snmp trap功能将trap信息发送到1.1.1.5，版本为v1，团体字符为usertrap
  # 先启用trap功能
  snmp-server enable trap
  snmp-server host 1.1.1.5 v1 usertrap
  ```

- SNMP V3

  ```jsx
  # 使能snmp服务
  snmp-server enable
  # 配置 snmpv3用户名、密码与用户组
  # 用户名tester密码test用户组testgroup不使用私有算法加密，使用MD5算法鉴别
  snmp-server user tester testgroup authNoPriv auth md5 test
  # 配置snmpv3组
  # 使用鉴别且加密的安全级别，读视图为test，写视图为test，通告视图为test
  snmp-server group testgroup authpriv read test write test notify test
  
  # 配置MIB视图
  # 视图名为test，视图子树名为1，子树包含在视图中
  snmp-server view test 1 include
  # 配置snmp trap功能
  # 使能snmp trap
  snmp-server enable traps
  # 设置接受trap的管理端ip为1.1.1.5，v3版本，不鉴别不加密， 用户名为tester
  snmp-server host 1.1.1.5 v3 noauthnopriv tester
  ```

### 访问管理 am

接入 DCRS E1/0/9，仅允许 IP 地址 192.168.30.1-192.168.30.10 为源的数
据包为合法包，以其它 IP 地址为源地址，交换机直接丢弃

```jsx
am enable
int e1/0/9
am port
am ip pool 192.168.30.1 30
```



# DCHP

- point

  ```bash
  service dhcp
  wireless
  ap secape
  ```

- 无线配置介绍

  - 截图的时候，vap0默认关联network1，vap1关联network2
  - 做题要看dhcp在SW还是在RS上做，如果在RS上做就需要开启帧中继

  ```jsx
  # 无线配置
  wireless
  enable # 打开无线功能 ！！
  ap authentication none # ap配置无认证登录
  static-ip 192.168.1.1 # ap管理IP
  # 配置无线类型
  ap profile 1 # 默认1即可
  name DCAP # 配置ap方案名称
  hwtype 29 # 默认为29，具体可使用show vendor查看对应的ap型号
  
  # network下可以设置：ssid、用户最大数、带宽、用户隔离、时间限制
  # 假如配置的是network1，那么可以不用配置vap 0
  # 假如配置的是network2，那么就需要配置vap 1并使能
  # radio 1是2.4Ghz，radio 2是5Ghz
  # 配置ap network1
  network 1
  security mode wpa-perspnal # 配置网络安全模式
  ssid beijing1# 配置ssid名称
  vlan 110 # 配置管理vlan
  wpa key 12345678 # 配置网络的WPA秘钥，至少8位
  
  # ------其他--------
  # 配置channel和功率
  ap datebase 00-03-0f-20-fd-c0
  radio 1 channel 1
  
  # AC给AP下发配置
  end # 回退到特权模式
  wireless ap profile apply 1
  
  # 关联硬件类型
  # 默认为29，WL8200-I2，如果是WL8200-I1则为59
  hwtype 29
  ap escape # 配置ap
  
  # 查看ap状态
  show wireless ap status 
  # 查看无线
  show wireless
  # 可以用这个命令查看设备型号来查看设备数字
  show vender
  ```

- 无线基础配置

  ```bash
  # 第一步先配置Trunk
  
  # 如果在WS上配置DHCP
  # 则需要在连接ac的端口上配置trunk
  int e1/0/23
  sw mo tr
  # 这里110为管理vlan
  sw trunk native vlan 110
  # 连接rs的端口配置access
  int e1/0/24
  sw mo acc
  sw acc vlan 66
  
  # 如果在RS上配置DHCP
  # 则需要在WS上连接ac和rs的口配置trunk，RS上还需要做帧中继
  int e1/0/23
  sw mo tr 
  sw tr na vlan 110
  sw tr allowed vlan 110,111,222
  int e1/0/24
  sw mo tr
  sw tr na vlan 66
  sw tr allowed vlan all
  ```

  ### 无线必做内容

  DCWS配置VLAN110为管理VLAN, AP动态方式注册到AC，AC管理IP为192.168.110.254； 数据VLAN为111和222，分别下发网段192.168.111.0/24，192.168.222.0/24，网关为最后一个可用IP

  ```bash
  # 进入无线子接口
  wireless
  # 开启无线
  enable
  # 关闭ap认证，一般为关闭，如果有要求mac地址认证则输入mac
  ap authentication none
  # 配置一个静态ip给ap，这里题目管理ip为192.168.110.254
  static-ip 192.168.110.254
  ```

  - 由于无线需要自动获取ip，所以需要配置dhcp与地址池

    ```bash
    # 创建一个dhcp池
    ip dhcp pool vlan110
    # 声明一个网段
    network-address 192.168.110.0 255.255.255.0
    # 开启dhcp服务
    service dhcp
    ```

  - 配置无线ssid

    ```bash
    wireless
    # 如果配置静态ip则需要配置不自动获取ip
    no auto-ip-assign
    network 1
    vlan 110
    # 设置ssid
    ssid wifissid
    # 设置vlan
    vlan 110
    # 配置wifi密码
    security mode wpa-personal
    wpa key encrypted 12345678
    exit
    # 配置ap profile
    ap profile 1
    hwtype 29
    ap escape # 配置ap
    ```

  ## OPTION 43方式注册上线

  AP 通过 option43 方式进行正常注册上线。（如果题目出现

  ```bash
  wireless
  enable
  no authentication none
  static-ip 192.168.110.254
  # 在管理vlan下创建
  ip dhcp pool vlan110
  network-address 192.168.110.0 255.255.255.0
  # open 43 hex 0104是固定的，后面的数用计算机计算
  # C0是十六进制的192，A8是十六进制的168
  # 6E是十六进制的110，FE是十六进制的254
  option 43 hex 0104C0A86EFE
  ```

## 前期准备

- `language chinese` 可以设置中文
- 在纸上把需要用到vlan的写上
- `show run`后创建一个记事本在上面配置

如果是一个设备可以接入多个端口则设定一个固定端口即可

## RS基础

- 基础配置
- enable 密码

## RS安全

- 远程管理方式
- arp端口扫描
- arp欺骗
- 端口流量镜像
- 环路检测
- MAC地址学习
- 配置认证服务器
- snmp
- 访问管理 am

## DCHP

- point

  ```bash
  service dhcp
  wireless
  ap secape
  ```

- 无线配置介绍

  - 截图的时候，vap0默认关联network1，vap1关联network2
  - 做题要看dhcp在SW还是在RS上做，如果在RS上做就需要开启帧中继

  ```shell
  # 无线配置
  wireless
  enable # 打开无线功能 ！！
  ap authentication none # ap配置无认证登录
  static-ip 192.168.1.1 # ap管理IP
  # 配置无线类型
  ap profile 1 # 默认1即可
  name DCAP # 配置ap方案名称
  hwtype 29 # 默认为29，具体可使用show vendor查看对应的ap型号
  
  # network下可以设置：ssid、用户最大数、带宽、用户隔离、时间限制
  # 假如配置的是network1，那么可以不用配置vap 0
  # 假如配置的是network2，那么就需要配置vap 1并使能
  # radio 1是2.4Ghz，radio 2是5Ghz
  # 配置ap network1
  network 1
  security mode wpa-perspnal # 配置网络安全模式
  ssid beijing1# 配置ssid名称
  vlan 110 # 配置管理vlan
  wpa key 12345678 # 配置网络的WPA秘钥，至少8位
  
  # ------其他--------
  # 配置channel和功率
  ap datebase 00-03-0f-20-fd-c0
  radio 1 channel 1
  
  # AC给AP下发配置
  end # 回退到特权模式
  wireless ap profile apply 1
  
  # 关联硬件类型
  # 默认为29，WL8200-I2，如果是WL8200-I1则为59
  hwtype 29
  ap escape # 配置ap
  
  # 查看ap状态
  show wireless ap status 
  # 查看无线
  show wireless
  # 可以用这个命令查看设备型号来查看设备数字
  show vender
  ```

- 无线基础配置

  ```bash
  # 第一步先配置Trunk
  
  # 如果在WS上配置DHCP
  # 则需要在连接ac的端口上配置trunk
  int e1/0/23
  sw mo tr
  # 这里110为管理vlan
  sw trunk native vlan 110
  # 连接rs的端口配置access
  int e1/0/24
  sw mo acc
  sw acc vlan 66
  
  # 如果在RS上配置DHCP
  # 则需要在WS上连接ac和rs的口配置trunk，RS上还需要做帧中继
  int e1/0/23
  sw mo tr 
  sw tr na vlan 110
  sw tr allowed vlan 110,111,222
  int e1/0/24
  sw mo tr
  sw tr na vlan 66
  sw tr allowed vlan all
  ```

  ### 无线必做内容

  DCWS配置VLAN110为管理VLAN, AP动态方式注册到AC，AC管理IP为192.168.110.254； 数据VLAN为111和222，分别下发网段192.168.111.0/24，192.168.222.0/24，网关为最后一个可用IP

  ```bash
  # 进入无线子接口
  wireless
  # 开启无线
  enable
  # 关闭ap认证，一般为关闭，如果有要求mac地址认证则输入mac
  ap authentication none
  # 配置一个静态ip给ap，这里题目管理ip为192.168.110.254
  static-ip 192.168.110.254
  ```

  - 由于无线需要自动获取ip，所以需要配置dhcp与地址池

    ```shell
    # 创建一个dhcp池
    ip dhcp pool vlan110
    # 声明一个网段
    network-address 192.168.110.0 255.255.255.0
    # 开启dhcp服务
    service dhcp
    ```

  - 配置无线ssid

    ```bash
    wireless
    # 如果配置静态ip则需要配置不自动获取ip
    no auto-ip-assign
    network 1
    vlan 110
    # 设置ssid
    ssid wifissid
    # 设置vlan
    vlan 110
    # 配置wifi密码
    security mode wpa-personal
    wpa key encrypted 12345678
    exit
    # 配置ap profile
    ap profile 1
    hwtype 29
    ap escape # 配置ap
    ```

  ## OPTION 43方式注册上线

  AP 通过 option43 方式进行正常注册上线。（如果题目出现

  ```bash
  wireless
  enable
  no authentication none
  static-ip 192.168.110.254
  # 在管理vlan下创建
  ip dhcp pool vlan110
  network-address 192.168.110.0 255.255.255.0
  # open 43 hex 0104是固定的，后面的数用计算机计算
  # C0是十六进制的192，A8是十六进制的168
  # 6E是十六进制的110，FE是十六进制的254
  option 43 hex 0104C0A86EFE
  ```