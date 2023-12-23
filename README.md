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
