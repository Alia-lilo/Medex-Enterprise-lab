➡️ You can find the troubleshooting commands at the bottom of this page or click [here](Routing.md#troubleshooting-commands).

### ➡️ Why Routing is Important ⁉️

Without routing, the device could only talk to devices in the same network.  
Routing allows communication between networks, provides Internet access, and facilitates branch connections.  
That’s why we need to teach each device the paths to other networks — so that whenever traffic is sent, it knows exactly where to go.  
to do this, we’ll go through each component in our project and configure the necessary routes step by step:  
first, let’s understand how routing works in each part of the lab.

## Routing & Traffic Flow Summary in the EVE–FortiGate Lab

🔶 **OUR Goal:**

Establish full communication between:
- The servers outside EVE (in network `172.16.16.64/27`)
- The routers inside EVE (with HSRP configuration)
- The FortiGate firewall (responsible for Internet access and filtering)

All traffic should first pass through the HSRP routers, then go to the FortiGate, and finally reach the Internet.  
The server sends any traffic destined outside its own network to the gateway (VIP = 172.16.16.94).  
The VIP (HSRP) receives the traffic and forwards it through the active router.  
The active router (for example, R2) sends the traffic to R1 as the next hop.  
R1 then forwards the traffic to the FortiGate through the network 172.16.18.0/24.  
Finally, the FortiGate routes the traffic out from port1 to the Internet via the default route (0.0.0.0/0 → 192.168.1.1).  

#

🔶 **Network Overview:**

| Component                            | Network           | Description                            |
| ------------------------------------ | ----------------- | -------------------------------------- |
| **FortiGate port1 (WAN)**            | `192.168.1.0/24`  | connected to Internet (Gateway = 192.168.1.1) |
| **FortiGate port2 (LAN)**            | `172.16.18.0/24`  | Main Network inside EVE – vmnet2      |
| **FortiGate port3 (Server Network)** | `172.16.16.64/27` | Servers Network – vmnet5                |
| **R1 ↔ R2**                          | `172.16.16.x`     | Include HSRP، VIP is `172.16.16.94`    |
| **Server**                           | `172.16.16.77/27` | Outside EVE، GW = 172.16.16.94         |

#

🔶 **Traffic Path (How Traffic Moves)**

🔸From Server → Internet

The server sends any traffic destined outside its own network to the gateway (VIP = 172.16.16.94).  
The VIP (HSRP) receives the traffic and forwards it through the active router.  
The active router (for example, R2) sends the traffic to R1 as the next hop.  
R1 then forwards the traffic to the FortiGate through the network 172.16.18.0/24.  
Finally, the FortiGate routes the traffic out from port1 to the Internet via the default route (0.0.0.0/0 → 192.168.1.1).  
Now, let’s start configuring the routing on each part of the network..

#

### 🔶 EVE-NG Console

When we configured the EVE-NG network earlier and edited the /etc/network/interfaces file, you might have noticed that we didn’t set a gateway (gw).

you can see EVE configuration from here:  
[Configuring EVE-NG Network Interfaces](2__Prepare_the_Network_Environment.md#5%EF%B8%8F%E2%83%A3-configuring-eve-ng-network-interfaces).

That’s because EVE-NG cannot have two gateways, and there was a line like this:

` post-up ip route add 172.16.18.0/24 dev eth1`

> These static routes are persistent because they are applied automatically each time the interface comes up.
> 

What does that mean?


🔸 after the interface is brought up, add a route.  
This is similar to a static route, but not the same, because a static route includes a next-hop (via), it’s a direct route, which tells the system:  
The network `172.16.18.0/24` is directly connected to interface `eth1`, So, if any traffic is going to that network, send it through `eth1`.  

> We wrote it inside the file so the route remains persistent even after a restart — not temporary.

🔶 Understanding the configuration line

- `post-up`: run this command after the interface is up and becomes operational.
- `ip route add`: add a route to the routing table
- `172.16.18.0/24`: the destination network, The route applies to any IP from `172.16.18.0` to `172.16.18.255`.
- `dev eth1`: Send packets for that network out via the network interface `eth1`.
No gateway was given, so the kernel will try to reach hosts on that network directly through eth1 (using ARP).
This method is used when the destination network is directly connected to that link.


> We created a specific route for the network we want to reach through a particular interface (such as eth1), 
For example, if we want to reach any VM or device in the 172.16.16.0 network, we use the eth2 interface

➡️ We added a specific route for each network we want to reach through its interface: 

🔸For the `core` network connected to `vmnet2`:

```shell
post-up ip route add 172.16.18.0/24 dev eth1
```

🔸For the `servers` network connected to `vmnet5`:

```shell
post-up ip route add 172.16.16.64/27 dev eth2
```

>⚠️ Important Note:
>We wrote the command this way because these networks are in the same broadcast domain (same switch/bridge).
>If the network isn’t actually reachable on that interface, the traffic won’t be delivered.
>
>So make sure that:
> - The vmnet2 is connected to eth1 and sees devices in 172.16.18.x, and
> - The vmnet5 is connected to eth2 and sees devices in 172.16.16.x.
>
> **Once these routes are added, EVE-NG can communicate properly with both FortiGate and the internal server network.**
>

### 🔶 Routers (OSPF)

If you’d like to start by checking the router configuration first, you can see it here: [Routers Configuration](docs/3-EVE-NG_Full_Lab_Deployment/Cairo_Routers_Configuration.md)

Let's talk about OSPF routing and what benefits it gives us.

⁉️ Why I chose OSPF?

I chose OSPF (Open Shortest Path First) because it’s a dynamic routing protocol that automatically updates routes when the network topology changes. 
Even if the network is small, OSPF helps me practice real-world routing behavior — like fast failover, automatic route updates, and smooth route management. 
So instead of manually configuring static routes, OSPF allows routers to learn and update paths automatically, which is more scalable and realistic for enterprise environments.

We added the following configuration:

```
interface loopback 0
ip add 1.1.1.1 255.255.255.255
exit
router ospf 1
network 1.1.1.1 0.0.0.0 area 0
network 172.16.16.128 0.0.0.15 area 0
```

🔶 Understanding OSPF Configuration Commands:

- `interface loopback 0`

This command creates a virtual interface called `Loopback 0`.
It’s not a physical interface (no real cable connected), but a virtual one that’s always up and independent by router.

We usually use it as the Router ID in OSPF, so the router can identify itself using a fixed IP address.
When sending Hello messages or forming neighbor relationships, the router says, "I’m the router with ID 1.1.1.1" `

- `router ospf 1`
This starts an OSPF process. The number 1 here is called the process ID — it’s just a local identifier on the router.

- `network 1.1.1.1 0.0.0.0 area 0`

This tells the router to include the interface with IP 1.1.1.1 in the OSPF process, under area 0 (the backbone area), the area that all networks should connect to

⁉️Why do we add the Loopback interface to OSPF?

because it mainly holds the Router ID, and we want other routers to see and learn it.
If we don’t advertise it in OSPF, other routers won’t know that this router has a network `1.1.1.1`.

By including it, we can ping the Loopback IP from any other router in the topology and verify that OSPF is working properly.
The `0.0.0.0` wildcard means “match this exact IP address only,” not the whole `1.1.1.x` network — because the Loopback interface has only one IP.

- `network 172.16.16.128 0.0.0.15 area 0`
This command adds another network to OSPF.
Network: `172.16.16.128`

🔹Wildcard mask: `0.0.0.15` → (which equals subnet mask `255.255.255.240` or `/28`). 
That means any interface on the router that has an IP address from this network will be included in OSPF.

When we add all these networks to area 0, the router starts sending Hello messages on those interfaces.
Once it discovers neighbors, they exchange routing tables, form adjacency, and learn routes to each other’s networks.

--------------------------------------------------------------

### 🔶 Static Routing Configuration (R1, R2).

We used OSPF so that all networks inside the EVE lab can communicate with each other.
But for the lab to connect with external networks — like the FortiGate and servers — we also needed to add routing from EVE to reach them.

To keep things simple, I used static routes, since this is just a simulation.
(In real-world environments, this would usually be much easier and more automated.)

🔸 Step 1 — Configure static routes on R1

R1 sends traffic directly to FGT  
R1 needs to know how to reach any traffic outside its directly connected networks (such as the Internet through the FortiGate).
Since FortiGate is the device connected to the Internet, we add a default route pointing to it:

```shell
ip route 172.16.18.0 255.255.255.0 172.16.18.130
show ip route # Check Routing on R1
```
➡️ This means: any traffic going to the 172.16.18.0 network (vmnet2) should go through FortiGate (port2).

🔹 At first, the server couldn’t access the Internet — ping 8.8.8.8 failed.

So, I ran `tracert 8.8.8.8` from the server to check where the traffic stopped.

It reached the router but couldn’t find a route to continue — simply because we hadn’t added routing on the router yet.

So we fixed it by adding this route: `ip route 0.0.0.0 0.0.0.0 172.16.16.69`

That’s the default route, the path the device uses to send packets outside its local network.

It basically means:

> “If you receive traffic for any IP that doesn’t belong to your connected networks, send it to this next hop 172.16.16.69.”  

🔸 Step 2 — Configure static routes on R2

sends traffic directly to FGT through R1  

```shell
conf t
ip route 172.16.18.0 255.255.255.0 172.16.16.154
exit
show ip route # Check Routing on R2
```

➡️ This tells R2 to send any traffic destined for 172.16.18.0/24 (vmnet2) through FortiGate (port2).

----
# 🔁 Failover Scenarios

⁉️ But then, a new issue appeared —

If R2 goes down, and it’s the one forwarding traffic to R1, what happens next?

Actually, R1 is the router that knows the path to the FortiGate.
R2 simply forwards traffic to R1.
But if R1 goes down, R2 won’t know how to reach the FortiGate because its route was pointing to R1:
` ip route 0.0.0.0 0.0.0.0 172.16.16.69 `


⚠️ If R1 goes down:

Currently, R2 sends all its traffic to R1,
→ so traffic will stop if R1 fails.

We need to provide an alternate path to reach the FortiGate directly..

✅ Solutions:
1️⃣ Cloud connection:

Connect R2 to the same Cloud (network 172.16.18.0/24) as R1.
This way, it can reach the FortiGate directly even if R1 goes down.

2️⃣ Backup Route:
Add a secondary static route on R2 as a backup:

```shell
ip route 172.16.18.0 255.255.255.0 172.16.18.130 10
```
(The higher metric means it will only be used when the primary route fails.)

3️⃣ OSPF (Recommended in future):


Enable OSPF between R1, R2, and the FortiGate.
This way, if any router goes down or a path changes, OSPF will automatically recalculate and reroute traffic.

I plan to test this later when I’m more familiar with FortiGate,
since I’m currently using the limited version and don’t want it to cause unexpected issues.

🔶 **Final Notes**

🟢 FortiGate can communicate directly with R1 (Layer 3 reachable).  
🟢 The servers outside EVE can only see the 172.16.16.64/27 network.  
🟢 HSRP provides redundancy inside EVE, but if only R1 is connected to the FortiGate, a backup route is essential.  
🟢The most important rule in routing:  
Every hop must know both the forward and return paths.

------------------------------------------------------------------------------------------------

### 🔶 FortiGate Routing Configuration


➤ Static Routes

1- Default Route (for WAN access)
  - Destination: 0.0.0.0/0.0.0.0
  - Gateway: 192.168.1.1
  - Interface: port1 (WAN)

This route sends all unknown traffic to the WAN gateway for internet access.

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

"Since R1 is in the same broadcast domain, we use interface routing without a next-hop."

🔹 That means:
If you receive traffic going to for the 172.16.16.64/27 (Servers_Network), send it through port3 — where the router (R1) is located and knows how to reach that network.”

✅ Summary
Our current setup:

- FortiGate has IP 172.16.18.130/24 on port3.
- R1 has IP 172.16.18.131/24 on the same link (cloud).
- The Servers Network (172.16.16.64/27) is located behind R1.

so, we need to add a static route on FortiGate

Any traffic going to the network  should be routed through port3

➤ Policies
.

.

.

“Full configuration available in the FGT Configuration documentation.”

[FortiGate Configuration](FGT_Configuration.md#-static-routes)

-------------  
### 🫒 Route Summary Table

| Source Device               | Destination Network | Next Hop / Gateway       | Interface                | Purpose / Notes                   |
| --------------------------- | ------------------- | ------------------------ | ------------------------ | --------------------------------- |
| **Server**                  | 0.0.0.0/0           | 172.16.16.94 (HSRP VIP)  | NIC (LAN)                | Default route to internal gateway |
| **HSRP VIP (172.16.16.94)** | 0.0.0.0/0           | Active Router (R1 or R2) | VLAN10                   | Forward traffic to active router  |
| **R2**                      | 0.0.0.0/0           | 172.16.16.154 (R1)        | fa0/0                   | Send traffic to R1                |
| **R1**                      | 0.0.0.0/0           | 172.16.18.130 (FGT)      | fa1/0                    | Default route toward FortiGate    |
| **EVE-NG (Debian Host)**    | 172.16.18.0/24      | via eth1                 | Core network (vmnet2)    | Route to FortiGate & routers      |
| **EVE-NG (Debian Host)**    | 172.16.16.64/27     | via eth2                 | Servers network (vmnet5) | Route to internal servers         |
| **FortiGate**               | 0.0.0.0/0           | 192.168.1.1              | port1 (WAN)              | Internet access                   |
| **FortiGate**               | 172.16.16.64/27     | port3                    | Internal servers         | Forward traffic to R1/HSRP        |


### 🔍 ROUTING Troubleshooting Commands ⏩ [Troubleshooting Reference](/Troubleshooting-Reference.md)  
---------------
