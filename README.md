# SetupUbuntu22
Setup Ubuntu 22 VMs.  

I have been setting up servers and VMs with CentOS 7 since 2018.  I usually run things in containers on top of that, so I have been slow to change the underlying operating system.  But now with some of the new features in Kubernetes and AI, I need an OS with a more uptodate kernal, thus I have switched to Ubuntu 22.  Here's my notes for how I set it up.

## Create VM using VirtualBox
I created a VM using virtual box using the following prompts:
```
English
Continue without update
US Keyboard, Done
Ubuntu Server
static 192.168.100.179, 192.168.100.0/24
No proxy address
Use an entire disk
Done
Continue
Profile setup: jkozik/password, Ubuntu22179
Skip for now
Install ssh server
Select docker
Reboot now
```
Note:  1-I setup a login jkozik. 2-Setting a staic IP address requires setting a funky ip address mask,  see my example.

Reference: [https://www.youtube.com/watch?v=vxb894qV7-8](https://www.youtube.com/watch?v=vxb894qV7-8)
## Initial setup
```
apt-get update
apt-get upgrade
```
reboot, then set hostname:
```
hostnamectl --pretty set-hostname "Ubuntu 22 on 179"
hostnamectl set-hostname ubuntu22179.kozik.net
```
Verify /etc/hosts looks ok
Reference:  [Server World](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=hostname)
## Setup ssh
```
jkozik@ubuntu22179:~$ mkdir -p /home/jkozik/.ssh && chmod 700 /home/jkozik/.ssh
jkozik@ubuntu22179:~$ cp authorized_keys /home/jkozik/.ssh && chmod 600 authorized_keys
jkozik@ubuntu22179:~$ chown -R jkozik:jkozik /home/jkozik/.ssh
```
Then update on my windows PC /users/jackk/.ssh/config file to point to the new VM's IP address.  Verify that ssh to the new VM works.

It is worth noting that you can setup jump servers:
```
PS C:\Users\jackk> ssh -J jkozik@dell1.kozik.net jkozik@192.168.100.179
```
See  [jumpserver](https://wiki.gentoo.org/wiki/SSH_jump_host) reference.

# Setup Container Environment
## Docker
The Ubuntu install doesn't install the latest version of docker, thus install the latest and enable jkozik to run it without sudo
```
apt update
apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update
apt-cache policy docker-ce
apt install docker-ce
systemctl status docker
usermod -aG docker jkozik
docker run hello-world
```
Login as jkozik and verify groups and docker ps.

References
-[Digitial Ocean Tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

## Docker Compose
A good way to further verify docker is to install docker compose and test it.
```
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
docker compose version
mkdir ~/compose-demo

cd ~/compose-demo
mkdir app
nano app/index.html
nano docker-compose.yml
docker compose up -d
```
Get the content from the [Digital Ocean setup instructions](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)

## Setup Git
```
sudo apt update
sudo apt install git
git --version
git config --global user.name "Jack Kozik"
git config --global user.email "jackkozik@email.com"
git config --list
```
Reference: [Digital Ocean setup git](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-22-04)

## NFS Client
For Kubernetes Nodes, I use PV,PVCs linked to NFS mounts.  Each Worker node needs to have an NFS client installed
```
root@knode203:~# apt install nfs-common
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libnfsidmap1 rpcbind
Suggested packages:
  watchdog
The following NEW packages will be installed:
  keyutils libnfsidmap1 nfs-common rpcbind
0 upgraded, 4 newly installed, 0 to remove and 88 not upgraded.
Need to get 381 kB of archives.
After this operation, 1,447 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libnfsidmap1 amd64 1:2.6.1-1ubuntu1.2 [42.9 kB]
Get:2 http://us.archive.ubuntu.com/ubuntu jammy/main amd64 rpcbind amd64 1.2.6-2build1 [46.6 kB]
Get:3 http://us.archive.ubuntu.com/ubuntu jammy/main amd64 keyutils amd64 1.6.1-2ubuntu3 [50.4 kB]
Get:4 http://us.archive.ubuntu.com/ubuntu jammy-updates/main amd64 nfs-common amd64 1:2.6.1-1ubuntu1.2 [241 kB]
Fetched 381 kB in 0s (955 kB/s)
Selecting previously unselected package libnfsidmap1:amd64.
(Reading database ... 110255 files and directories currently installed.)
Preparing to unpack .../libnfsidmap1_1%3a2.6.1-1ubuntu1.2_amd64.deb ...
Unpacking libnfsidmap1:amd64 (1:2.6.1-1ubuntu1.2) ...
Selecting previously unselected package rpcbind.
Preparing to unpack .../rpcbind_1.2.6-2build1_amd64.deb ...
Unpacking rpcbind (1.2.6-2build1) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../keyutils_1.6.1-2ubuntu3_amd64.deb ...
Unpacking keyutils (1.6.1-2ubuntu3) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../nfs-common_1%3a2.6.1-1ubuntu1.2_amd64.deb ...
Unpacking nfs-common (1:2.6.1-1ubuntu1.2) ...
Setting up libnfsidmap1:amd64 (1:2.6.1-1ubuntu1.2) ...
Setting up rpcbind (1.2.6-2build1) ...
invoke-rc.d: policy-rc.d denied execution of start.
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /lib/systemd/system/rpcbind.socket.
/usr/sbin/policy-rc.d returned 101, not running 'start rpcbind.service rpcbind.socket'
Setting up keyutils (1.6.1-2ubuntu3) ...
Setting up nfs-common (1:2.6.1-1ubuntu1.2) ...

Creating config file /etc/idmapd.conf with new version

Creating config file /etc/nfs.conf with new version
Adding system user `statd' (UID 115) ...
Adding new user `statd' (UID 115) with group `nogroup' ...
Not creating home directory `/var/lib/nfs'.
invoke-rc.d: policy-rc.d denied execution of start.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /lib/systemd/system/nfs-client.target.
/usr/sbin/policy-rc.d returned 101, not running 'start auth-rpcgss-module.service nfs-client.target nfs-idmapd.service nfs-utils.service proc-fs-nfsd.mount rpc-gssd.service rpc-statd-notify.service rpc-statd.service rpc-svcgssd.service rpc_pipefs.target var-lib-nfs-rpc_pipefs.mount'
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.8) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Restarting services...
 /etc/needrestart/restart.d/systemd-manager
 systemctl restart irqbalance.service packagekit.service polkit.service ssh.service systemd-journald.service systemd-networkd.service systemd-resolved.service systemd-timesyncd.service systemd-udevd.service udisks2.service upower.service
Service restarts being deferred:
 systemctl restart ModemManager.service
 /etc/needrestart/restart.d/dbus.service
 systemctl restart networkd-dispatcher.service
 systemctl restart systemd-logind.service
 systemctl restart unattended-upgrades.service
 systemctl restart user@1000.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
root@knode203:~# 
```
- [How to Install NFS Server and Client on Ubuntu 22.04](https://www.tecmint.com/install-nfs-server-on-ubuntu/#:~:text=Install%20the%20NFS%20Client%20on%20the%20Client%20Systems)


## Firewall notes
The firewall appears to be off by default. It uses the ufw command, new to me.  See the reference.
```
jkozik@ubuntu22179:~$ sudo su
root@ubuntu22179:/home/jkozik# ufw app list
Available applications:
  OpenSSH
root@ubuntu22179:/home/jkozik# ufw status
Status: inactive
root@ubuntu22179:/home/jkozik#
```
Reference: [Digital Ocean setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-22-04)

Another reference, [Server World for Ubuntu 22](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=initial_conf&f=3)https://www.server-world.info/en/note?os=Ubuntu_22.04&p=initial_conf&f=3t)
