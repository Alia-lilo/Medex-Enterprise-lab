# ✅ Routers Configuration:

## 🟢 **R-3**

```shell

conf t
hostname R3

inter fa0/0
no shutdown
ip address 172.16.17.133 255.255.255.240
exit
inter fa1/0
no shutdown
ip address 172.16.17.145 255.255.255.252
exit
inter fa0/1
no shutdown
exit

interface fa0/1.10
 encapsulation dot1q 10
 ip address 172.16.17.65 255.255.255.224
exit
interface fa0/1.20
 encapsulation dot1q 20
 ip address 172.16.17.97 255.255.255.224
exit
interface fa0/1.30
 encapsulation dot1q 30
 ip address 172.16.17.1 255.255.255.192
exit

```

## 🟢 **R-4**

```shell

conf t
hostname R4

inter fa0/0
no shutdown
ip address 172.16.17.134 255.255.255.240
exit
inter fa1/0
no shutdown
ip address 172.16.17.146 255.255.255.252
exit
inter fa0/1
no shutdown
exit

interface fa0/1.10
 encapsulation dot1q 10
 ip address 172.16.17.66 255.255.255.224
exit
interface fa0/1.20
 encapsulation dot1q 20
 ip address 172.16.17.98 255.255.255.224
exit
interface fa0/1.30
 encapsulation dot1q 30
 ip address 172.16.17.2 255.255.255.192
exit

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
