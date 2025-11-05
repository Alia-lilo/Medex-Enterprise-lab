# 🚀 Full Lab Deployment in EVE-NG

Now that everything is ready, it’s time to start building our lab in EVE-NG.

We’ll begin by adding the routers and switches exactly as shown in the network diagram. After that, we’ll power on each device and configure them one by one.

Once the basic setup is complete, we’ll run a few ping tests to make sure everything is connected and working properly.

We’ll start with the Cairo branch first — and after it’s fully configured and tested, we’ll move on to the Giza branch, which should be faster and easier to complete.

![EVE Lab](images/EVE_Topology.png).

When adding Cloud nodes in EVE-NG, make sure that you create multiple cloud interfaces and connect each one to a separate VPC device. For example:
Connect cloud0 to one VPC only
Connect cloud1 to another VPC
and so on ..

![Clouds in EVE](/images/eve-cloud.png)

Open each VPC, assign it an IP address in a suitable range,

The cloud that connected to R1 is VMnet2
ping 172.16.18.140 - The one that responds successfully is the correct cloud linked to VMnet2.

The cloud that connected to SW2 is VMnet5
ping 172.16.16.68 - The one that responds successfully is the correct cloud linked to VMnet5.


```shell
# vmnet5 to switch
ip 172.16.16.80 255.255.255.224 172.16.16.68  
sh ip  
ping 172.16.16.68 
```

```shell
# vmnet2 to R1
ip 172.16.18.142 255.255.255.0 172.16.18.140  
sh ip  
ping 172.16.18.140 
```
➡️ **Next Step:** [Cairo Routers Configuration](Cairo_Routers.md)
   	
