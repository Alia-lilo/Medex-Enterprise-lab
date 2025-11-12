# ✅ Routers Configuration:
> R = Router

## **R-1**

```shell
inter fa0/0
no shutdown
ip address 172.16.16.131 255.255.255.240
exit
inter fa2/0
no shutdown
ip address 172.16.16.153 255.255.255.252
exit
inter fa1/0
no shutdown
ip address 172.16.16.157 255.255.255.252
exit
inter fa0/1
no shutdown
ip address 172.16.16.145 255.255.255.252
exit

# 🟢INTER_VLAN

interface fa0/1.10
encapsulation dot1q 10
ip address 172.16.16.65 255.255.255.224
exit
interface fa0/1.20
encapsulation dot1q 20
ip address 172.16.16.97 255.255.255.224
exit
interface fa0/1.30
encapsulation dot1q 30
ip address 172.16.16.1 255.255.255.192
exit

# 🟢OSPF Configuration 

interface loopback 0
ip add 1.1.1.1 255.255.255.255
exit
router ospf 1
network 1.1.1.1 0.0.0.0 area 0
network 172.16.16.128 0.0.0.15 area 0
network 172.16.16.152 0.0.0.3 area 0
network 172.16.16.156 0.0.0.3 area 0
network 172.16.16.64 0.0.0.31 area 0
network 172.16.16.96 0.0.0.31 area 0
network 172.16.16.0 0.0.0.63 area 0

# 🟢HSRP

int f0/1.10
standby 10 ip 172.16.16.94
standby 10 priority 110  
standby 10 preempt

int f0/1.20
standby 1 ip 172.16.16.126
standby 1 priority 100
standby 1 preempt

int f0/1.30
standby 1 ip 172.16.16.62 
standby 1 priority 100
standby 1 preempt

copy running-config startup-config # to save configuration
```

## **R-2**

```shell
inter fa0/0
no shutdown
ip address 172.16.16.132 255.255.255.240
exit
inter fa2/0
no shutdown
ip address 172.16.16.154 255.255.255.252
exit
inter fa1/0
no shutdown
ip address 172.16.16.161 255.255.255.252
exit
inter fa0/1
no shutdown
ip address 172.16.16.150 255.255.255.252
exit

# 🟢INTER_VLAN

interface fa0/1.10
encapsulation dot1q 10
ip address 172.16.16.66 255.255.255.224
exit
interface fa0/1.20
encapsulation dot1q 20
ip address 172.16.16.98 255.255.255.224
exit
interface fa0/1.30
encapsulation dot1q 30
ip address 172.16.16.2 255.255.255.192
exit

# 🟢OSPF Configuration 

interface loopback 0
ip add 2.2.2.2 255.255.255.255
exit
router ospf 1
network 2.2.2.2 0.0.0.0 area 0
network 172.16.16.128 0.0.0.15 area 0
network 172.16.16.152 0.0.0.3 area 0
network 172.16.16.160 0.0.0.3 area 0
network 172.16.16.64 0.0.0.31 area 0
network 172.16.16.96 0.0.0.31 area 0
network 172.16.16.0 0.0.0.63 area 0

# 🟢HSRP

int fa0/1.10
standby 1 ip 172.16.16.94
standby 1 priority 100
standby 1 preempt

int fa0/1.20
standby 1 ip 172.16.16.126
standby 1 priority 110  
standby 1 preempt

int fa0/1.30
standby 1 ip 172.16.16.62 
standby 1 priority 110
standby 1 preempt

copy running-config startup-config # to save configuration
```

## 📌Router Troubleshooting Commands

- `show ip interface brief` — interfaces summary  
- `show interfaces status` — interface status  
- `show running-config | section interface <interface>` — config section  
- `show running-config interface <interface>` — specific interface config  
- `show ip ospf database` — OSPF LSDB  
- `show ip ospf interface` — OSPF interface info  
- `show ip ospf neighbor` — OSPF neighbors  
- `show ip route ospf` — OSPF routes  
- `show ip protocols` — routing protocols info
- `show arp` — ARP table  


➡️ **Next Step:** [Cairo Switches Configuration](Cairo_Switches_Configuration.md)
