❇️ Configure FortiGate to allow all networks to communicate with each other and access the Internet.

Here, we’ll access the FortiGate Web GUI using 172.16.18.130 to start working on its configuration and routing.
Everything related to the FortiGate setup will be done here.

![Fortigate Interfaces](/images/fgt-interfaces.png) 

These are the interfaces we already created earlier, and we can modify them anytime if needed.

![Edit LAN Interface](/images/fgt-LAN.png)


## ➤ Static Routes

Go to Network → Static Routes, and create the following routes:

1- Default Route (for WAN access)
  - Destination: 0.0.0.0/0.0.0.0
  - Gateway: 192.168.1.1
  - Interface: port1 (WAN)

This route sends all unknown traffic to the WAN gateway for internet access.

#

2- Route for LAN (Servers_Network)

Since the FortiGate is directly connected to the servers network (port3 – 172.16.16.64/27):

  - Destination: 172.16.16.64/27
  - Gateway: 172.16.16.94
  - Interface: port3 (Servers)

This route forwards traffic from the FortiGate to the HSRP virtual gateway in EVE Lab Vlan-10.

✅ No gateway is needed here because the network is directly connected at Layer 3.
 ## 🔆 **Very important to understand**

The network 172.16.16.64/27 is located on the internal switch behind R1, not on the same cable that connects FortiGate and R1.

When the FortiGate receives traffic from the Internet that is going to for the server
(for example, a reply from 8.8.8.8 going back to the server 172.16.16.77),
it must know how to reach that destination.

However, FortiGate doesn’t see the 172.16.16.64/27 network as directly connected (because it’s actually behind R1).
So we have to add a manual route, like this:

3- Route for LAN (Servers_Network)
 - Destination: 172.16.16.64/27
 - GW 0.0.0.0
 - Interface: port3 (Servers)

🔹 That means:
If you receive traffic going to for the 172.16.16.64/27 (Servers_Network), send it through port3 —
where the router (R1) is located and knows how to reach that network.”

✅ Summary
Our current setup:

- FortiGate has IP 172.16.18.130/24 on port3.
- R1 has IP 172.16.18.131/24 on the same link (cloud).
- The Servers Network (172.16.16.64/27) is located behind R1.

so, we must add a static route on FortiGate

Any traffic going to the network  should be routed through port3


## ➤ Policies

> If you prefer, you can add the same configuration using the CLI commands below,
or enter the same configuration from the GUI.

🔷 Policy 1 – LAN to WAN

This policy allows users in the LAN network to access the Internet through the WAN interface.

```shell
config firewall policy
    edit 0

        set name "LAN-to-WAN"
        set srcintf "LAN"
        set dstintf "WAN"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
    next
end
```

🔷 Policy 2 – LAN to Servers Network


This policy allows traffic from the LAN (port2) to communicate with the `Servers_Network` (port3).
Example: a user in the LAN subnet can access a server hosted in the Servers network.

```shell
config firewall policy
    edit 0
        set name "LAN-to-Servers"
        set srcintf "LAN"
        set dstintf "Servers"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat disable
    next
end

```
⚠️ Note: NAT is disabled because both subnets are internal; NAT is only needed when traffic leaves the private network to the Internet.


🔷 Policy 3 – Servers Network to WAN

This policy allows servers (on port3) to access the Internet through the WAN interface.

```shell
config firewall policy
    edit 0
        set name "Servers-to-WAN"
        set srcintf "Servers"
        set dstintf "WAN"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
    next
end
```

✅ At this point, routing has been configured on the FortiGate.
After configuring routing on the router in EVE and adding a VM to the 172.16.16.64 network, we’ll be able to verify the connectivity like this.

ping <server ip>
ping <HSRP Vip>

![server and fgt pings eachother.png](/images/server-pings-fgt.png)








## You may need these commands to help you in FGT CLI:

```bash
get system status # show all about FGT
get system interface # show interfaces and configuration
get router info routing-table all # show static Routes
get router info routing-table details 172.16.x.x
execute ping <ip> # ping a host from the FortiGate.
execute traceroute <ip> # trace route to a host.
get system arp 
show firewall policy
diag debug enable
```
#  
### 🔄 **Still in Progress**

⏳ This isn’t the end!  

There are still more FortiGate features I want to try later just as Policies, Web Filter, App Control, Traffic Shaping, logging&  Monitoring plus a few others I might add as I go.  
I’ll update this section whenever I implement something new.
