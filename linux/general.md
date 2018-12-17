**check sudoers file**
```
visudo -cs
```

**download package without installing**
```
yum install --downloadonly --downloaddir=/tmp/ sudo
```

**extract package contents**
```
rpm2cpio sudo-1.8.rpm | cpio -idmv
```

**list swapping processes in order**
```
for file in /proc/*/status ; do \
awk '/VmSwap|Name/{printf $2 " " $3}END{ print ""}' $file; \
done | sort -k 2 -n -r | less
```
