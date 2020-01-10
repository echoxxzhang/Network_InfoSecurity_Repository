

# 命令

## DCRS
### 配置环路检测时间

配置存在环路时的检测时间间隔为30秒，不存在环路时的检测时间间隔为10秒

hostname(config)# loopback-detection interval-time 30 10

### 限制端口学习MAC

为终端产生防止 MAC 地址防洪攻击，请配置端口安全，
已划分 VLAN 的端口最多学习到 5 个 MAC 地址，发生违规阻止后续违规流量
通过，不影响已有流量并产生 LOG 日志；

``` shell
// 限制学习到5个MAC
(config-if-ethernet1/0/1)#switchport port-security
(config-if-ethernet1/0/1)#switchport port-security maximum 5
(config-if-ethernet1/0/1)#switchport port-security violation restrict
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



