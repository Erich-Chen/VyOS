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

set interfaces ethernet eth1 address '192.168.0.1/24'
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

