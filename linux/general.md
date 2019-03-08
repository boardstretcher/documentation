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
**using ldapsearch**
```
ldapsearch -x -D "CN=jenkins,OU=ServiceAccounts,DC=dev,DC=net" \
-W -H ldaps://dc01.dev.net -b "DC=dev,DC=net" \ 
-s sub 'uid=boardstretcher'
```
**using screen to share**
```
# on the sending side
screen -S some_name
# on the reading side
screen -x some_name
```
**using parallel**
```
# More about the tutorial
man parallel_tutorial
# Compress all *.html files in parallel – 2 per CPU thread
parallel --jobs 200% gzip ::: *.html
# Convert all *.wav to *.mp3 using lame
parallel lame {} -o {.}.mp3 ::: *.wav
# Chop bigfile into 1MB blocks and grep for the string foobar
cat bigfile | parallel --pipe grep foobar

```
**diagnosing boot times and systemd**
```
systemd-analyze blame
```
