## ❌Switch Shutting Down Problem & Switch License:


Cisco IOL switches shut down by default without a valid license.  
To fix this, we’ll generate a local license using a Python script and apply it to EVE-NG.
Let’s take it step by step:

In the switch path in EVE (`/opt/unetlab/addons/iol/bin`), we’ll add **two** files:

1️⃣ `license.py` → Python script to generate the license.

2️⃣ `iourc` → The file where the license is stored.

> 💡 **Tip:** This path is the default location for IOL images in EVE-NG.
> 
```shell
#! /usr/bin/python
#print "*********************************************************************"
#print "Cisco IOU License Generator - Kal 2011, python port of 2006 C version"
import os
import socket
import hashlib
import struct
# get the host id and host name to calculate the hostkey
hostid=os.popen("hostid").read().strip()
hostname = socket.gethostname()
ioukey=int(hostid,16)
for x in hostname:
 ioukey = ioukey + ord(x)
print "hostid=" + hostid +", hostname="+ hostname + ", ioukey=" + hex(ioukey)[2:]
# create the license using md5sum
iouPad1='\x4B\x58\x21\x81\x56\x7B\x0D\xF3\x21\x43\x9B\x7E\xAC\x1D\xE6\x8A'
iouPad2='\x80' + 39*'\0'
md5input=iouPad1 + iouPad2 + struct.pack('!i', ioukey) + iouPad1
iouLicense=hashlib.md5(md5input).hexdigest()[:16]
print "\nAdd the following text to ~/.iourc:"
print "[license]\n" + hostname + " = " + iouLicense + ";\n"
print "You can disable the phone home feature with something like:"
print " echo '127.0.0.127 xml.cisco.com' >> /etc/hosts\n"
```

Open the Eve console:

```bash
python2 /opt/unetlab/addons/iol/bin/license.py
# او
cd /opt/unetlab/addons/iol/bin
python2 license.py
cd
```

You’ll see the license output like this, make sure to copy it and keep it safe for the next step

```bash
[license]
9bb9484430e3ae8f63b6b0b088f02181;
```

> You can copy and paste it to the required location if you're comfortable working in Ubuntu, but I'm using the most reliable method to avoid any typos.
>

2️⃣ The second file is **`iourc`** (You can either create it directly inside EVE, or create it on your host and upload it using **WinSCP**).

🔹In the Eve console:

```bash
touch /opt/unetlab/addons/iol/bin/iourc # Creat `iourc` file

chmod 0755 /opt/unetlab/addons/iol/bin/iourc # modify permissions

nano iourc # to edit file (it's empty)

# paste the output of python2 license.py
[license]
Medex_Lab = 9bb9484430e3ae8f63b6b0b088f02181; # must be your hostname of Eve

#to save and quit press "**ctrl+x**" -> "**y**" -> "**enter**"

/opt/unetlab/wrappers/unl_wrapper -a fixpermissions

echo "127.0.0.1 iou" >> /etc/hosts      #⚠️ “This step is mandatory; without it, the license check will fail.”
```

🔹In your Host:

Open Notepad, paste the license inside, save the file as `iourc`, and upload it using WinSCP to the same switch path: /opt/unetlab/addons/iol/bin

```bash
[license]
Medex_Lab = 9bb9484430e3ae8f63b6b0b088f02181; # must be your hostname of Eve
```

To confirm the **hostname**, type `hostname` in the EVE console, it will display the exact name, and you need to copy it exactly as shown.

🔹 Then in Eve

```bash
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
echo "127.0.0.1 iou" >> /etc/hosts
```
```
🔑 If you did everything correctly but skipped this step:

 `echo "127.0.0.1 iou" >> /etc/hosts` The switch won’t work because when Eve tries
to launch the switch, it needs to resolve the name `iou` to an IP address. If it
doesn’t find that mapping in `/etc/hosts` or DNS, the license fails, and the switch
shuts down immediately.

This tricks the `license.py` script into thinking it contacted Cisco. This bypasses
the license check since we don't have a real Cisco license server.
This command ensures that `iou` always resolves to `127.0.0.1`, which is essential
for the license to work.

I won’t lie to you, I once spent a whole day stuck, not knowing what the problem was,
and it turned out I had missed the echo command, and I also wrote the wrong
hostname in the license. I had written it like this:

[license]
eve-ng = 9bb9484430e3ae8f63b6b0b088f02181;
```
### 🔁 Reboot the EVE VM to make sure everything loads correctly.

**Now you should be able to run the switch.**

- Switches path
- License script
- `iourc` file
- `fixpermissions`
- `echo "127.0.0.1 iou" >> /etc/hosts`
-------
