# datacentertopovmm

```
vmm capacity -g vmm-default
vmm stop
vmm unbind 
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

### contrial command server set up
Before you begin, set up servers and/or VMs that meet the specified requirements. Also ensure that the Contrail Command server and all the hosts in the Contrail cluster have /etc/hosts entries for each other over the management network.

em0-->external0-->eth0
em1-->ens3f1
em2-->
```
etc/hosts
10.53.68.75    contrail-command
10.53.72.222    aio-openstack-contrail
10.53.72.221    contrail-compute
10.53.72.220    appformix
10.53.72.22    appformixflows
```
```
contrail command
[root@contrail-command network-scripts]# vi /etc/sysconfig/network-scripts/ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=static
ONBOOT=yes
PREFIX=24
IPADDR=172.16.16.11
[root@contrail-command network-scripts]# service network restart
```
```
[root@aio-openstack-contrail ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=static
ONBOOT=yes
PREFIX=24
IPADDR=172.16.16.12
[root@aio-openstack-contrail ~]# service network restart
```
```
[root@contrail-compute ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=static
ONBOOT=yes
PREFIX=24
IPADDR=172.16.16.13
[root@contrail-compute ~]# service network restart
```
```
[root@appformix ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=static
ONBOOT=yes
PREFIX=24
IPADDR=172.16.16.14
[root@appformix ~]# service network restart
```
```
[root@appformixflows ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens3f1
DEVICE=ens3f1
BOOTPROTO=static
ONBOOT=yes
PREFIX=24
IPADDR=172.16.16.15
[root@appformix ~]# service network restart
```

```

vi /etc/ssh/sshd_config
PasswordAuthentication yes

systemctl restart sshd.service

passwd 
```
```
[root@contrail-command ~]# cat command_servers.yml
---
# Required for Appformix installations
user_command_volumes:
- /opt/software/appformix:/opt/software/appformix
- /opt/software/xflow:/opt/software/xflow
command_servers:
    server1:
        ip: 10.53.68.75
        connection: ssh
        ssh_user: root
        ssh_pass: c0ntrail123
        sudo_pass: c0ntrail123
        ntpserver: 66.129.233.81

        registry_insecure: false
        container_registry: hub.juniper.net/contrail
        container_tag: 2003.1.40-rhel
        container_registry_username: JNPR-FieldUser419
        container_registry_password: TGbZvQ1QTcpPQtfYA0Rp
        config_dir: /etc/contrail

        contrail_config:
            database:
                type: postgres
                dialect: postgres
                password: contrail123
            keystone:
                assignment:
                    data:
                      users:
                        admin:
                          password: contrail123
            insecure: true
            client:
              password: contrail123
```
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-18.06.0.ce
systemctl start docker
check docker version (it happened that the version showed as 18.03 and will not work)
```
[root@contrail-command ~]# docker version
Client:
 Version:           18.06.0-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        0ffa825
 Built:             Wed Jul 18 19:08:18 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.0-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       0ffa825
  Built:            Wed Jul 18 19:10:42 2018
  OS/Arch:          linux/amd64
  Experimental:     false
```
docker login hub.juniper.net --username JNPR-FieldUser419 --password TGbZvQ1QTcpPQtfYA0Rp
docker pull hub.juniper.net/contrail/contrail-command-deployer:2003.1.40-rhel
docker run -td --net host -v /root/command_servers.yml:/command_servers.yml --privileged --name contrail_command_deployer hub.juniper.net/contrail/contrail-command-deployer:2003.1.40-rhel
docker logs -f contrail_command_deployer
```

```
cd /opt/software/appformix/
yum install wget
wget http://dl.appformix.juniper.net/releases/v3.1.15/appformix-3.1.15.tar.gz
wget http://dl.appformix.juniper.net/releases/v3.1.15/appformix-dependencies-images-3.1.15.tar.gz
wget http://dl.appformix.juniper.net/releases/v3.1.15/appformix-network_device-images-3.1.15.tar.gz
wget http://dl.appformix.juniper.net/releases/v3.1.15/appformix-openstack-images-3.1.15.tar.gz
wget http://dl.appformix.juniper.net/releases/v3.1.15/appformix-platform-images-3.1.15.tar.gz
cd /opt/software/xflow/
wget http://dl.appformix.juniper.net/appformix_flows/releases/v1.0.7/appformix-flows-1.0.7.tar.gz
wget http://dl.appformix.juniper.net/appformix_flows/releases/v1.0.7/appformix-flows-ansible-1.0.7.tar.gz
```
defautl vrouter gateway 11.11.0.1

You can check provisioning logs from contrail_command container by 
```
docker logs -f contrail_command

OR

docker exec contrail_command tail -f /var/log/contrail/deploy.log
docker exec -it contrail_command /bin/sh
Contrail Command GUI creates instances.yml file used for cluster provisioning at following location "/var/tmp/contrail_cluster/Cluster-UUID/". Please check and review.
```
```

```
```
global_configuration:
  CONTAINER_REGISTRY: hub.juniper.net/contrail
  REGISTRY_PRIVATE_INSECURE: false
  CONTAINER_REGISTRY_USERNAME: JNPR-FieldUser419
  CONTAINER_REGISTRY_PASSWORD: TGbZvQ1QTcpPQtfYA0Rp
provider_config:
  bms:
    ssh_user: root
    ssh_pwd: c0ntrail123
    ntpserver: 66.129.233.81
    domainsuffix: local
instances:
  aio-openstack-contrail:
    ip: 10.53.72.222
    ssh_user: root
    ssh_pwd: c0ntrail123
    provider: bms
    roles:
      config:
      config_database:
      control:
      webui:
      analytics:
      analytics_database:
      analytics_alarm:
      analytics_snmp:
      vrouter:
        TSN_EVPN_MODE: true
        VROUTER_GATEWAY: 11.11.0.1
      openstack_control:
      openstack_network:
      openstack_storage:
      openstack_monitoring:
      appformix_openstack_controller:
      appformix_bare_host:
  contrail-compute:
    ip: 10.53.72.221
    ssh_user: root
    ssh_pwd: c0ntrail123
    provider: bms
    roles:
      vrouter:
        VROUTER_GATEWAY: 11.11.0.1
      openstack_compute:
      appformix_compute:
      appformix_bare_host:
  appformix:
    ip: 10.53.72.220
    provider: bms
      appformix_controller:
      appformix_bare_host:
      appformix_network_agents:
contrail_configuration:
  CONTRAIL_VERSION: "2003.1.40-rhel"
  CLOUD_ORCHESTRATOR: openstack
  RABBITMQ_NODE_PORT: 5673
  VROUTER_GATEWAY: 11.11.0.1
  ENCAP_PRIORITY: VXLAN,MPLSoUDP,MPLSoGRE
  OPENSTACK_VERSION: queens
  AUTH_MODE: keystone
  KEYSTONE_AUTH_HOST: 10.53.72.222
  KEYSTONE_AUTH_URL_VERSION: /v3
  CONTROL_NODES: 172.16.16.12  #######diff
  PHYSICAL_INTERFACE: ens3f1  ######diff
  TSN_NODES: 172.16.16.12   #######diff
  CONTRAIL_CONTAINER_TAG: "2003.1.40-rhel"
kolla_config:
  kolla_globals:
    enable_haproxy: no
    enable_haproxy: no
    enable_ironic: no
    enable_swift: yes
    openstack_release: queens
    swift_disk_partition_size: 20GB
  kolla_passwords:
    keystone_admin_password: contrail123
  customize:
    swift-proxy-server/proxy-server.conf: |
      [filter:authtoken]
      service_token_roles_required = True
      service_token_roles = admin
    nova.conf: |
             [libvirt]
             virt_type=qemu
             cpu_mode=none
appformix_configuration:
    appformix_license:  /opt/software/appformix/appformix-internal-openstack-3.1.sig
```
```
docker run -td --net host -e action=provision_cluster -v /root/command_servers.yml:/command_servers.yml -v /root/instances.yml:/instances.yml --privileged --name contrail_command_deployer hub.juniper.net/contrail/contrail-command-deployer:2003.1.40-rhel
```
### For JCL

The vQFX is running inside the CentOS VM. You can do a 'virsh console vqfx-re' to get into the console

 spine-01 100.123.13.101/16
 spine-02 100.123.13.102/16
 leaf-01 100.123.13.201/16
 leaf-02 100.123.13.202/16
 
 
