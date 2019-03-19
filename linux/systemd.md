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
```
