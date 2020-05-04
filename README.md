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
BOOTPROTO=static
ONBOOT=yes
PREFIX=24
IPADDR=10.10.0.11

[root@bms-01 network-scripts]# service network restart
```
bms02
```
[root@bms-02 network-scripts]# cat ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=static
ONBOOT=yes
PREFIX=24
IPADDR=10.11.0.21

[root@bms-02 network-scripts]# service network restart
```

bms03
```
[root@bms-03 network-scripts]# cat ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=none
ONBOOT=yes
PREFIX=24
IPADDR=10.10.0.31

[root@bms-03 network-scripts]# service network restart
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


# use ansible to save and load the configs
```

sudo yum install epel-release
sudo yum repolist
sudo yum install ansible
ansible-galaxy install Juniper.junos

sudo easy_install pip
pip install junos-eznc (not sure why sudo pip install junos-eznc not working)
pip install jxmlease

```

mac:
```
export PATH=$PATH:/Users/wouyang/Library/Python/2.7/bin/
```



# note
1.  the jinja template does not work when using . instead of "" format 
e.g. this one does not work, when there is ether-switching:
```
#jinja2: lstrip_blocks: True
interfaces {
  {% for interface in interfaces %}
    {{ interface.name }} {
      {% for unit in interface.unit %}
        unit {{ unit.number }} {
            family {{ unit.address_family }} {
              {% if unit.address_family == "inet" %}
                address {{ unit.ip_address }};
              {% elif unit.address_family == "ethernet-switching" %}
                interface-mode {{ unit.interface-mode }};
                vlan {
                    members {{ unit.vlan_members }};
                }
              {% endif %}
                description {{ unit.description }};
            }
        }
      {% endfor %}
    }
  {% endfor %}
}

error:
fatal: [leaf-01]: FAILED! => {"changed": false, "msg": "AnsibleUndefinedVariable: 'dict object' has no attribute 'interface'"}
```

this one works:
```
#jinja2: lstrip_blocks: True
interfaces {
  {% for interface in interfaces %}
    {{ interface["name"] }} {
      {% for unit in interface["unit"] %}
        unit {{ unit["number"] }} {
            family {{ unit["address_family’] }} {
              {% if unit["address_family"] == "inet" %}
                address {{ unit["ip_address’] }};
              {% elif unit["address_family"] == "ethernet-switching" %}
                interface-mode {{ unit["interface-mode"] }};
                vlan {
                    members {{ unit["vlan_members"] }};
                }
              {% endif %}
                description {{ unit["description"] }};
            }
        }
      {% endfor %}
    }
  {% endfor %}
}
```
