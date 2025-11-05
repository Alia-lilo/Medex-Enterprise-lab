
### 🔍 Troubleshooting Commands

| Device       | Command                                         | Notes                               |
 ------------ | ----------------------------------------------- | ----------------------------------- |
| FortiGate    | `get router info routing-table all`             | Shows all learned and static routes |
| Cisco Router | `show ip route`                                 | Displays routing table              |
| FortiGate    | `get router info routing-table details 0.0.0.0` | Verify gateway for internet traffic |
| Cisco Router | `show ip route 0.0.0.0`                         | Confirms next-hop for default route |
| Both         | `execute traceroute <IP>` / `traceroute <IP>`   | Check hop-by-hop path               |
| FortiGate    | `get system interface physical`                 | Confirms IP and link status         |
| Cisco Router | `show ip interface brief`                       | Quick overview of IPs and status    |
| FortiGate    | `show router static`                   | Shows all configured static routes  |                               |
| Cisco Router | `show running-config`                   | section ip route`                   | Lists static routes in config |
| FortiGate    | `get router info routing-table static` | Verifies if static route is active  |                               |
| Cisco Router | `show ip route static`                 | Confirms static routes applied      |                               |
| Both         | `execute traceroute <destination>`     | Should show next-hop you configured |                               |
| FortiGate    | `show router static`                   | Shows all configured static routes  |                               |
| Cisco Router | `show running-config`                   | section ip route`                   | Lists static routes in config |
| Cisco Router | `show ip route static`                 | Confirms static routes applied      |                               |
| Both         | `execute traceroute <destination>`     | Should show next-hop you configured |                               |
| FortiGate    | `get router info ospf neighbor`                                 | Verify adjacency with routers                     |
| Cisco Router | `show ip ospf neighbor`                                         | Check if neighbor state = FULL                    |
| FortiGate    | `get router info ospf database`                                 | Lists LSAs learned                                |
| Cisco Router | `show ip ospf database`                                         | Same purpose                                      |
| FortiGate    | `get router info ospf interface`                                | Check active OSPF interfaces                      |
| Cisco Router | `show ip ospf interface`                                        | Includes area and hello/dead timers               |
| Cisco Router | `show ip route ospf`                                            |                                                   |



🔄 Still in Progress  
⏳ This isn’t the end!

