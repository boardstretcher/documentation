**Dump CDP packets from cisco**
```
tcpdump -nn -v -i eth0 -s 1500 -c 1 'ether[20:2] == 0x2000'
```

**LLDP/CDP packets for force10***
If you are using an offbrand, or force10, that had LLDP instead of CDP, you have to first enable LLDP on the switch: 
```
conf t
protocol lldp
no disable
```
then tcpdump the LLDP packets instead:
```
tcpdump -i eth0 ether proto 0x88cc
```
