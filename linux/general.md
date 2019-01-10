**check sudoers file**
```
visudo -cs
```

**download package without installing**
```
yum install --downloadonly --downloaddir=/tmp/ sudo
```

**simple benchmark test for CPU**
```
yum install parallel bc
time cat /proc/cpuinfo |grep proc|wc -l|xargs seq|parallel -N 0 echo "2^2^20" '|' bc
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

**test https**
```
openssl s_client -connect google.com:443
```
