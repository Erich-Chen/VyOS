# VyOS
VyOS guideline  
这份官方UserGuide已经很完整很清晰了，在这里整理一下自己使用中的思路，因为一些设置上和顺序上还是有些别的想法。  

## 安装和初次登陆
* 光盘启动
* 使用用户名密码vyos/vyos登陆
```
install image
reboot
```

## 添加新用户
```
# 自带的vyos似乎删不掉（也许可以直接修改config.boot文件）；至少把密码修改一下
set system login user vyos authentication plaintext-password newpassword
# 添加新用户
set system login user jsmith full-name "Johan Smith" 
set system login user jsmith authentication plaintext-password mypassword 
set system login user jsmith level admin 
commit
show system login 
save
```

## 连接网络
使用VMWare的话，我的设置是一个网卡（eth0）为Bridge，另一个网卡（eth1）为host-only。  
```
set interfaces ethernet eth0 address dhcp
set interfaces ethernet eth0 description 'OUTSIDE'

set interfaces ethernet eth1 address '10.1.1.1/24'
set interfaces ethernet eth1 description 'INSIDE'

commit
save
```
每次提交都需要commit生效，save保存到配置文件以保证重启后仍然有效。后面就不再写了。  

## 启用SSH
```
set service ssh port '22'

commit
save
```

##  一些配置
```
#Configure Source NAT for our "Inside" network.
set nat source rule 100 outbound-interface 'eth0'
set nat source rule 100 source address '10.1.1.0/24'
set nat source rule 100 translation address masquerade

#Configure a DHCP Server:
set service dhcp-server disabled 'false'
set service dhcp-server shared-network-name LAN subnet 10.1.1.0/24 default-router '10.1.1.1'
set service dhcp-server shared-network-name LAN subnet 10.1.1.0/24 dns-server '10.1.1.1'
set service dhcp-server shared-network-name LAN subnet 10.1.1.0/24 domain-name 'internal-network'
set service dhcp-server shared-network-name LAN subnet 10.1.1.0/24 lease '86400'
set service dhcp-server shared-network-name LAN subnet 10.1.1.0/24 start '10.1.1.101' stop '10.1.1.199'

#And a DNS forwarder:
set service dns forwarding cache-size '0'
set service dns forwarding listen-on 'eth1'
set service dns forwarding name-server '8.8.8.8'
set service dns forwarding name-server '114.114.114.114'

#commit and save
commit
save
```
此时如果需要宿主机和虚拟机通讯，可以设置host-only（vmnet1）网卡，使用  
* IP: 10.1.1.xx
* Mask: 255.255.255.0
* Gateway: 10.1.1.1
* DNS: 10.1.1.1 / 8.8.8.8 / 114.114.114.114 / any other
```
# On Host (我的系统是Windows 10)
ping www.baidu.com -S 10.1.1.xx
```

## 防火墙设置
```
set firewall name OUTSIDE-IN default-action 'drop'
set firewall name OUTSIDE-IN rule 10 action 'accept'
set firewall name OUTSIDE-IN rule 10 state established 'enable'
set firewall name OUTSIDE-IN rule 10 state related 'enable'

set firewall name OUTSIDE-LOCAL default-action 'drop'
set firewall name OUTSIDE-LOCAL rule 10 action 'accept'
set firewall name OUTSIDE-LOCAL rule 10 state established 'enable'
set firewall name OUTSIDE-LOCAL rule 10 state related 'enable'
set firewall name OUTSIDE-LOCAL rule 20 action 'accept'
set firewall name OUTSIDE-LOCAL rule 20 icmp type-name 'echo-request'
set firewall name OUTSIDE-LOCAL rule 20 protocol 'icmp'
set firewall name OUTSIDE-LOCAL rule 20 state new 'enable'
set firewall name OUTSIDE-LOCAL rule 30 action 'drop'
set firewall name OUTSIDE-LOCAL rule 30 destination port '22'
set firewall name OUTSIDE-LOCAL rule 30 protocol 'tcp'
set firewall name OUTSIDE-LOCAL rule 30 recent count '4'
set firewall name OUTSIDE-LOCAL rule 30 recent time '60'
set firewall name OUTSIDE-LOCAL rule 30 state new 'enable'
set firewall name OUTSIDE-LOCAL rule 31 action 'accept'
set firewall name OUTSIDE-LOCAL rule 31 destination port '22'
set firewall name OUTSIDE-LOCAL rule 31 protocol 'tcp'
set firewall name OUTSIDE-LOCAL rule 31 state new 'enable'

set interfaces ethernet eth0 firewall in name 'OUTSIDE-IN'
set interfaces ethernet eth0 firewall local name 'OUTSIDE-LOCAL'

commit
save

```

抄不下去了，VyOS算是很方便的了，User Guide也很完整。  
https://wiki.vyos.net/wiki/User_Guide  

## 其他命令
```
ping www.google.com
traceroute www.google.com
# 郑重推荐MTR
mtr www.google.com

monitor interfaces ethernet eth0 traffic

show configuration
# 可以直接修改配置文件
vi /config/config.boot
# 然后加载新的配置文件
conf
load
commit
save
```
