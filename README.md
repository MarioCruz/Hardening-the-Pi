# Hardening-the-Pi

This is from @alexellisuk Blog post http://blog.alexellis.io/hardened-raspberry-pi-nas/#2hardeningthepi

Add new user
Add a brand-new user to the system and delete the built-in pi user. We don't even want to risk leaving the default user in-tact.
```
sudo useradd alex -s /bin/bash -m -G adm,sudo
sudo passwd alex
``` 
Enter new UNIX password:  
Retype new UNIX password:  
passwd: password updated successfully  
We picked the sudo group because that allows our new user to elevate super-user permissions. They will still be prompted for a password every time they elevate though. Here's the line in /etc/sudoers which controls that behavior:

# Allow members of group sudo to execute any command
```
%sudo    ALL=(ALL:ALL) ALL
```
If you need to run commands on your Pi ssh non-interactively which need sudo access then you should change the line below. This is so your new user isn't prompted for a password from the sudo command. Your script would have no way to provide it.

# Allow members of group sudo to execute any command
```
%sudo    ALL=(ALL) NOPASSWD: ALL
```

This configuration mirrors the official Ubuntu 16.x cloud-provisioned images (cloudinit) which do not require a password for sudo access.

Try to make do without this change invoking the rule of least privilege.

Finally set a password for your new user, log out and log back in as the new user. Remove the pi user:
```
# sudo userdel pi
# sudo rm -rf /home/pi
```
Optimize GPU memory-split
This line will optimize the split of memory between the system and your Pi. A reboot is required but check the output of free -h before and after.

```
# free -h
```

Mem          total       used       free
              925M        72M       853M  
-/+ buffers/cache:        30M       895M

Swap:          99M         0B        99M

```
# echo "gpu_mem=16" | sudo tee -a /boot/config.txt
```

2.3 Change hostname
You should change the hostname of your NAS. Use the nano editor to edit /etc/hosts and /etc/hostname then reboot.

The avahi-daemon runs as a systemd service and will allow you to discover the Pi from a Mac or PC.

● avahi-daemon.service - Avahi mDNS/DNS-SD Stack
   Loaded: loaded (/lib/systemd/system/avahi-daemon.service; enabled)
   Active: active (running) since Sat 2016-12-03 15:48:50 UTC; 22min ago
 Main PID: 391 (avahi-daemon)
   Status: "avahi-daemon 0.6.31 starting up."
   CGroup: /system.slice/avahi-daemon.service
   
# Configure a static IP address
You will have an IP address issued via DHCP from your home router. It's a good idea to set this to a permanent number. Pick a high number so that it won't get allocated to another device on your network.

If you want to double check that nobody has that IP then type in
```
ping -c 1 192.168.0.240
```

Add these lines to /etc/dhcpcd.conf first checking if your home router assigns the 192.168.0.0/24 or 192.168.1.0/24 range.

```
interface eth0  
static ip_address=192.168.0.240/24  
static routers=192.168.0.1  
static domain_name_servers=8.8.8.8  
Reboot your Pi. The DNS entry here corresponds to Google's primary server.
```

2.5 Disallow password logins over SSH
Given enough time or a weak enough password a brute-force attacker with access to your Pi could gain entry to your Pi. That barrier can be made effectively much higher by disallowing SSH to accept password logins.

First generate an ssh-key if you don't already have one with the ssh-keygen program. Answer yes to all defaults.

Now copy the key into the Pi with ssh-copy-id:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'alex@naspi.local'"  
and check to make sure that only the key(s) you wanted were added.
A side-effect here is that we will not need to enter our password when connecting to the Pi over SSH. Do not lose your ssh-keys which were generated in ~/.ssh/ and keep them private.

Add PasswordAuthentication no to your SSHD config file:

echo "PasswordAuthentication no" | sudo tee -a /etc/ssh/sshd_config  
Reboot your system and check that you can still access the Pi remotely.

To check your system logs for SSH type in:
```
sudo journalctl _COMM=sshd --since=today
```
