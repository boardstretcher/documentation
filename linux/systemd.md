## Chrooting a centos 7 environment with nspawn
- Once you are booted and logged in, you can use init or halt to exit the chroot
```
su - root
cd /root/
mkdir chroot
yum --releasever=7 --nogpg --installroot=/root/chroot install systemd centos-release passwd yum
systemd-nspawn -D /root/chroot/
passwd
logout
systemd-nspawn -D /root/chroot/ -b
halt
```
## Create the container service
```
cat > /etc/systemd/system/new-chroot.service << EOF
[Unit]
Description=new-chroot container

[Service]
ExecStart=/usr/bin/systemd-nspawn -jbD /root/chroot/ 
KillMode=process
EOF
systemctl start new-chroot.service
```
## List and connect to the container
- Press ^] three times within 1s to exit session
```
machinectl list
machinectl login chroot

```
## Running commands in a systemd container
```
systemd-nspawn -D /root/chroot/ /bin/bash
systemd-nspawn -D /root/chroot/ /bin/echo "hello"
```
## Networking
```
# On the host
yum install bridge-utils
brctl addbr my-bridge
ip a a 10.1.1.78 dev my-bridge
```
