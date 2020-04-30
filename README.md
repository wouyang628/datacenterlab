# datacentertopovmm

```
vmm stop (wasn’t necessary all were unbound
vmm unbind (ditto)
vmm config datacenter_vmm_conf.conf -g vmm-default
vmm bind
vmm start
```

bms01
```
[root@bms-01 network-scripts]# cat ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=none
ONBOOT=yes
PREFIX=24
IPADDR=10.10.0.10

[root@bms-01 network-scripts]# service network restart
```
bms02
```
[root@bms-02 network-scripts]# cat ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=none
ONBOOT=yes
PREFIX=24
IPADDR=10.10.0.11

[root@bms-02 network-scripts]# service network restart
```

bms01 can ping bms02
```
[root@bms-01 network-scripts]# ping 10.10.0.11
PING 10.10.0.11 (10.10.0.11) 56(84) bytes of data.
64 bytes from 10.10.0.11: icmp_seq=1 ttl=64 time=107 ms
64 bytes from 10.10.0.11: icmp_seq=2 ttl=64 time=109 ms
64 bytes from 10.10.0.11: icmp_seq=3 ttl=64 time=105 ms
64 bytes from 10.10.0.11: icmp_seq=4 ttl=64 time=109 ms
```
