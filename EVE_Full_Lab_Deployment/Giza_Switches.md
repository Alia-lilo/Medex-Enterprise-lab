# ✅ Switches Configurations

## 🟢SW-4
```shell

conf t
hostname SW4
vlan 10
 name IT
exit
vlan 20
 name mgn
exit
vlan 30
 name emp
exit

interface range e0/0-1
switchport trunk encapsulation dot1q 
switchport mode trunk
switchport trunk allowed vlan 10,20,30

interface range e0/2 - 3
 channel-group 1 mode active
exit
interface Port-channel 1
switchport trunk encapsulation dot1q 
switchport mode trunk
switchport trunk allowed vlan 10,20,30


interface range Ethernet 1/0-3
 switchport mode access
 switchport access vlan 10
exit
interface range e2/0-1
 switchport mode access
 switchport access vlan 10
exit
interface range e2/2-3
 switchport mode access
 switchport access vlan 20
exit

interface range e3/0-3
 switchport mode access
 switchport access vlan 20
exit

interface range e4/0-3
 switchport mode access
 switchport access vlan 30
exit
interface range e5/0-3
 switchport mode access
 switchport access vlan 30
exit
```

## 🟢 SW-5

```shell

conf t
hostname SW5

vlan 10
 name IT
exit
vlan 20
 name mgn
exit
vlan 30
 name emp
exit

interface range e0/0-1
no  channel-group 1 mode on
 channel-group 1 mode passive
exit

interface Port-channel 1
switchport trunk encapsulation dot1q 
switchport mode trunk
switchport trunk allowed vlan 10,20,30

end
show etherchannel summary
show etherchannel 1 port-channel
```
---------------------------------------------
show ip route ospf
