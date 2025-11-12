## Prepare the Network Environment
<details>
  <summary>📄 This document covers: </summary>

  - 1️⃣ Modify VMware network
  - 2️⃣ Configure Host Network Settings
  - 3️⃣ Deploy FortiGate
  - 4️⃣ FGT Evaluation License
  - 5️⃣ FortiGate configuration Network Interfaces
  - 6️⃣ Configuring EVE-NG Network Interfaces
  - 7️⃣ Upload the Images to EVE
  - 🌐 EVE-NG Topology Setup
  - ✅ Testing Uploaded Router/Switch Images
  - Switch Shutting Down
</details>

![part-3](/images/part-3.jpg).

## ▶️  Virtual Network Preparation

Before we start working, we need to set up our workspace — just like how we prepare our desk,
notes, laptop, and a good cup of coffee before studying (and yes, we really need that coffee ☕).

Now, let’s prepare our working environment before diving into the actual setup.
But first, let me tell you about a common problem you might face and how to solve it.
##
### 🔆 Troubleshooting Network Connectivity Between FortiGate, EVE-NG, and Servers

🔸After finishing the VLSM design, creating all the interfaces, and completing the EVE-NG lab with routers and switches, I moved on to configuring the servers.


My goal was to connect the server to the EVE-NG lab so I could test HSRP and later connect the FortiGate to the Domain (we’ll explain that part in detail later).
In this setup, the server’s default gateway would be the HSRP virtual IP (vIP) of VLAN 10, and all network traffic would go out through the FortiGate.

🔸 The `extra` network in the VLSM table was originally planned as the link between the VMs, but the setup completely failed due to routing limitations on the FortiGate.
Routing between networks didn’t work unless I used MikroTik as a VM router, added multiple network adapters, configured routing manually, and managed it via Winbox — which involved many extra steps. (it's not complicated if you want to try it)

🔸 The main limitation was that FortiGate only supports three interfaces, and it doesn’t allow overlapping subnets.
For example, interfaces with 172.16.16.64/27 and 172.16.16.130/28 cannot coexist because both fall under the same 172.16.16.0/24 network.
Also,  it doesn’t allow create VLANs as standalone interfaces on FortiGate in this setup.

🔸 Using MikroTik with Winbox made routing possible and even easier to manage, but it added unnecessary complexity and installations that you might not encounter in real-world enterprise environments.
Here, my goal was to find the simplest way to interconnect all VMs while keeping the setup as close as possible to a realistic production scenario.

💭 So, I decided to create a new core network (172.16.18.0/24) dedicated only for communication between FortiGate, EVE-NG, the host, and clouds inside EVE.
This isolated core network simplified routing and avoided conflicts, while keeping the original VLSM plan intact for potential future use.

> Technically, I was supposed to apply this setup on VLAN 20 or 30 (for employees) to implement access policies and permissions.
However, to save time and reduce complexity, I chose VLAN 10, which is dedicated to servers and system components — this made everything clearer and easier to monitor.

### 1️⃣ Modify Virtual Network Editor (vmware):
- We will create a specific network, which is `172.16.18.x` for vmnet2 and `172.16.16.x` for vmnet5
**from VMware** :
- Edit → Virtual Network Editor
- Choose VMnet(2) host only → click change settings and assign the IP `172.16.18.0` `255.255.255.0`.
- Choose VMnet(5) host only → assign the IP `172.16.16.64` `255.255.255.224`.

![Virtual network settings.png](/images/Virtual-network-settings.png).

### 2️⃣ Configure Host Network Settings:

![Host Network Configuration](/images/Host-network-connection.png).
Go to Network Connections → open VMware Network Adapter VMnet2 and VMnet5

Set static IPs for VMnet2 and VMnet5 — one from the Core network range and the other from the VLAN10 range (refer to the VLSM table).

### 3️⃣ Deploy FortiGate:

- Open a virtual machine and select `FGT.ovf`.
- Before powering on the VM, go to VM → **Settings**
- Under **Network Adapters**, configure the following:
  - Network adapter:  Bridged
  - Network adapter1: Custom → `vmnet2` (Host-only)
  - Network adapter2: Custom → `vmnet5` (Host-only)

> 💡 Use `Network adapter3` : **NAT** only if the bridged adapter has no Internet access (e.g., hotspot).  
> If this is unclear, skip it to avoid routing conflicts.

Power on the vm: 
- login: admin
- Password: *(leave it blank and press **Enter**)*
- When prompted, set a new password (I used `123` for simplicity during the lab).

Now, let’s configure the FortiGate network interfaces:

![fgt-vm-settings](/images/fgt-vm-settings.png)

```shell

get system interface physical     #display all ports
config system interface           #Start interface configuration
edit port1                        #this port is always DHCP (Bridged NIC)
set mode static                   #set static IP depending on your host's IP (use ipconfig)
set ip 192.168.1.4 255.255.255.0  # ip from your host and choose ip from same range
set allowaccess http https ping ssh fgfm
set alias WAN                     #set port name to help with identification
end

config system interface
edit port2                       # This port is connected to `VMnet2`
set mode static
set ip 172.16.18.130 255.255.255.0
set allowaccess http https ping ssh fgfm
set alias LAN
end

config system interface
edit port3                       # This port is connected to `VMnet5`
set mode static
set ip 172.16.16.69 255.255.255.224  # same range of VLAN-10 Network
set allowaccess http https ping ssh fgfm
set alias Servers_Network
end

get system interface physical # make sure it's done

# To verify that **ports** is properly connected to VMnet2/VMnet5/WAN, perform the following tests:
execute ping 172.16.18.1
execute ping 8.8.8.8
```

```shell
 # from host

ping 172.16.18.130   
ping 172.16.16.69    
```
If there’s no reply, check:

- The IP addresses and subnet masks of both your VMs and host
- Network adapter assignments and connection types
- Your VMware Network Editor settings

### 🧰 Helpful FortiGate Troubleshooting Commands
You may need these commands to verify connectivity:

```bash
get system status # show all about FGT
get system interface physical # show ports and configuration
show system interface

execute ping <ip> # ping from FortiGate.

```

#

### 👉 For a full list of troubleshooting commands, see [Troubleshooting Reference](/Troubleshooting-Reference.md)

#

### 4️⃣ FGT Evaluation License
![fortigate evaluation license](/images/FGT-license.png)


- Open FortiGate in your browser using either the **WAN IP** or **LAN IP**
- Go to the **Evaluation License** tab, enter the email address you created on **FortiCloud** ➡️ https://customersso1.fortinet.com/

> ⚠️ Note: One email = One license per FortiGate VM
- The FGT will reboot and launch the **GUI..**
![FortiGate VM License](/images/FGT-VM-license.png)


This file contains all FortiGate-related configurations and notes. I separated it to make things easier to follow..
[FGT_Configuration](FGT_Configuration.md)



### 5️⃣ Configuring EVE-NG Network Interfaces

  This is the most important step, getting it right makes everything else much easier down the line, since many upcoming configurations depend on it.

To be honest, I reinstalled EVE-NG multiple times just to fully understand how it works under the hood.

- Open a virtual machine, select the EVE-COMM-VM.ovf file.
- Before you power on the VM, go to VM → Settings
- Select Network card → Bridged
- Select Network card 2→ Custom:  vmnet2 host only
- Select Network card 3 → Custom:  vmnet5 host only
- memory 4RAM, Processor 4 (optional as you need)
- power on, give it time, and log in:
> Username: root
> 
> password: eve

will open blue Eve-NG - Setup

- set your password, or leave it blank, and it will stay “eve”
- short hostname: Medex-lab
- DNS domain name: medex.local
- Use DHCP or Static IP : static
*Note:  we will change this later via the configuration file*
- type the IP address: `172.16.18.140` `255.255.255.0`
- type the Gateway: `empty` (FortiGate IP)
- Primary DNS server: `172.16.16.77` (Domain IP)
- Secondary DNS server: 8.8.8.8
- skip NTP server, choose direct connection

Sometimes, after choosing Static, the system won’t prompt you to enter an IP address automatically. In that case, you’ll need to set it manually — we’ll cover that in two steps.

Once the system finishes booting and the settings are applied correctly:

- Open your browser and enter the assigned IP address.
 You should now see the GUI interface — if it doesn’t appear, follow along with the next steps.

**🔄 If You Selected DHCP or Need to Change the IPs:**
Check your current IP configuration first

``` shell
ip a | less    # View IP addresses line by line
# press "q" to quit
```
To manually add a static IP, you’ll need to modify the configuration file

```shell
nano /etc/network/interfaces
```
> 🔑 Don't modify `eth0` or `pnet0`, because it's the default interface used by EVE-NG for internet access (usually set to Bridged). Changing it might break connectivity or cause issues later **just like it happened to me**.
Instead, we should use `eth1` for the internal lab network, to keep things organized and avoid any unexpected problems.

⚠️ eve-ng can't have 2 gateways, so later when we create routings between all networks, you will understand.

Then edit the file to this:

```shell
# Internet connection (on `eth0` → Bridged)
auto pnet0
iface pnet0 inet dhcp
    bridge_ports eth0
    bridge_stp off
    
# Cloud devices(vmnet2)
iface eth1 inet static
auto pnet1
iface pnet1 inet static
    bridge_ports eth1
    bridge_stp off
    address 172.16.18.140
    netmask 255.255.255.0
 post-up ip route add 172.16.18.0/24 dev eth1

# vmnet5
auto pnet2
iface pnet2 inet static
    bridge_ports eth2
    bridge_stp off
    address 172.16.16.68
    netmask 255.255.255.224
 post-up ip route add 172.16.16.64/27 dev eth2
    
# to save and quit press "ctrl+x" -> "y" -> "enter"
```
` — Change the NIC name depending on what you found in “ip a | less” `

Then from your host:

Ping `172.16.18.130`. If there is no reply, you may have changed the wrong interface or network card.

![Real-time screenshot taken while working on the lab.](/images/Ping-phase1.png).
Here I used `PuTTY` software because it’s easy to copy, paste, and scroll.

After installing EVE-NG, I always run an update. As it’s connected through FortiGate, internet access is already available.

```shell
apt update
apt install -y bridge-utils openvswitch-switch 
systemctl restart openvswitch-switch

/opt/unetlab/wrappers/unl_wrapper -a fixpermissions

```


### 🧰 Helpful EVE  Troubleshooting Commands

You may need these commands to verify connectivity:

```shell
nano /etc/network/interfaces  # internal lab IPs
systemctl restart networking  # to restart the network and apply changes
cat /etc/network/interfaces  # Read Interfaces Configuration
ifconfig | less # again to make sure you assigned the IP
ifdown pnet1 && ifup pnet1 # down interface and up again
reboot # restart eve after update or normal restart
ping 8.8.8.8
```

#

### 👉 For a full list of troubleshooting commands, see [Troubleshooting Reference](/Troubleshooting-Reference.md)

#

### 7️⃣ Upload the Routers/Switches to EVE:

With the foundational setup complete and everything running smoothly, it's time to bring in the routers and switches — and start building the real network.

> ⚠️ **Important:** This is the most critical and delicate step in EVE-NG,
as it requires patience and a clear understanding of what
you're doing — any mistake can corrupt your VM or damage the setup.


After installing **WinSCP**, don’t worry — it’s very simple.

**Left side is your laptop, right side is your EVE system.**

Use WinSCP to upload router and switch images **from your host to the EVE-NG VM**, so you can begin working on them inside the lab.
![WinSCP Interface](/images/WinSCP.png)

👉 Select the router images and upload them to the following path:
> ➡️ /opt/unetlab/addons/dynamips

Select the switch images and upload them to the following path:

> ➡️ /opt/unetlab/addons/iol/bin

⚠️ Make sure the folder names match the image type (dynamips / iol / qemu), otherwise the image won’t appear in the node list.”

After you’re done uploading, open the EVE console (in VM or PuTTy) :

```shell
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions

✅ This command **corrects file permissions and ownership**
to make sure the images are properly recognized and can run inside EVE-NG.
```

It’s a must-do step after every upload or path change, because it scans and fixes all permission issues inside that folder /opt/unetlab/addons/iol/bin

## 🌐 EVE-NG Topology Setup

Now, let’s open your browser and type the EVE IP address `172.16.18.140`

![Medex Lab Topology](/images/EVE_Topology.png).

Create a new lab → click + Add Node If the images are selectable (not greyed out), it means the upload was successful.

### ✅ Testing Uploaded Router/Switch Images

1. Add a **router** and a **switch** to your lab
2. Click **Start** for both devices
3. **Double-click** each one to open the console     
🔹 The **router** will load, initialize, and work fine  
🔻 The **switch**, however, might shut down right after you start it

✅ check this: [Solve Switches-License Problem](Switches-License-Issue-Eve.md) .  

Then, continue to start building the lab in EVE-NG ➡️ [Full Lab Deployment in EVE-NG](/EVE_Full_Lab_Deployment)
