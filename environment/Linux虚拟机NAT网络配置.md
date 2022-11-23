1.编辑虚拟机的NAT网络配置

![image-20221123144205397](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221123144205397.png)



2.编辑windows虚拟网卡配置

![image-20221123144346915](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221123144346915.png)



3.修改服务器/etc/sysconfig/network-scripts/的配置文件  ifcfg-ens160

```properties
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
# BOOTPROTO改为static
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens160
UUID=9142ef29-067d-4a05-b5e9-ecf5bd93b32c
DEVICE=ens160
# ONBOOT改为yes
ONBOOT=yes
# 添加以下配置
IPADDR=192.168.119.16
GATEWAY=192.168.119.2
NETMASK=255.255.255.0
DNS1=8.8.8.8
DNS2=114.114.114.114

```



**主要是三步设置中的网关和子网掩码要对应上**

