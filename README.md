# Ansible OpenStack Wrapper

This playbook is a wrapper for [ansible cloud openstack modules](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#openstack). 
It deploys (multiple) ssh-keys, images, networks, security groups and instances. Every instance will be addded to your inventory and usable for further tasks.
The goal is (hopefully) a more clear usage for deployments.


# Authentication
This role depends on [OpenStack clouds.yaml](https://docs.openstack.org/python-openstackclient/pike/configuration/index.html#clouds-yaml).
After configurating clouds.yaml your cloud can be set by '''os_cloud''' variable.
```
os_cloud: fancycloud
```

# Basic Tasks
Variables required by tasks

## deploy ssh keys
```
os_keys:
  key1:
    file: /tmp/id_rsa1.pub
  key2:
    file: /tmp/id_rsa2.pub
```
## deploy images
Usualy ssh-keys are not added to root user. login user can be set here
```
os_imgs:
  debian-9-openstack:
    file: /tmp/debian-9-openstack-amd64.qcow2
    login: debian
  xenial-server-cloudimg:
    file: /tmp/xenial-server-cloudimg-amd64-disk1.img
    login: ubuntu
```
## deploy networks
os_nets:
  jd:
    net: 10.42.42.0/24
    ext: ext02

## deploy security groups
Currently only one security group can be created.
```
os_ports: [ '22', '80', '443' ]
```

## deploy instances
To minimize Floating IP usage this playbook is able to reference jumphosts with the 'via' parameter.  
Ansible will access such instances via ssh proxy command.  
Furthermore you can add groups to instances. These groups will be used as ansible hostgroups.

> Instances will be split accross Zones. For that reason instance-names must be suffixed with two digit numbers.

```
os_srvs:
  jump01:
    image: debian-9-openstack
    flavor: flavor.42
    key: jd-github
    net: jd
    fip: 'yes'
  bench01:
    groups: ['instances', 'bench']
    image: debian-9-openstack
    flavor: flavor.42
    key: jd-github
    net: jd
    via: jump01
```

## add instances to inventory
All instances will be added to inventory. Default groups for these instances is '''instances'''.

As last step ssh access is printed:
```
ok: [localhost] => (item=jump01) => {
    "msg": "jump01: debian@<EXTERNAL-IP>"
}
ok: [localhost] => (item=bench01) => {
    "msg": "bench01: ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -J debian@<EXTERNAL-IP> debian@bench01"
}
```

## Tags
You can run specific tasks by using one or more following tags:
* os
* os:keys
* os:img
* os:net
* os:srv
* os:inv

# Examples

* [bench.yaml.example](bench.yaml.example)
* [galera.yaml.example](galera.yaml.example)


```
 
```
## Benchmark
[bench.yaml.example](bench.yaml.example) deploys a jumphost and three benchmark nodes.
iperf and ethtool is istalled on all benchmark nodes. After that iperf-server is started on bench01 and client is triggered on the other nodes.
 
### Output
```
$ ansible-playbook bench.yaml

PLAY [localhost] *****************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [stat] **********************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [get_url] *******************************************************************************************************************************************************************************************************************************
skipping: [localhost]

TASK [stat] **********************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [get_url] *******************************************************************************************************************************************************************************************************************************
skipping: [localhost]

TASK [include_role : openstack] **************************************************************************************************************************************************************************************************************

TASK [openstack : os_keypair] ****************************************************************************************************************************************************************************************************************
ok: [localhost] => (item=jd-github)

TASK [openstack : os_image] ******************************************************************************************************************************************************************************************************************
ok: [localhost] => (item=debian-9-openstack)
ok: [localhost] => (item=xenial-server-cloudimg)

TASK [openstack : os_network] ****************************************************************************************************************************************************************************************************************
ok: [localhost] => (item=jd)

TASK [openstack : os_subnet] *****************************************************************************************************************************************************************************************************************
ok: [localhost] => (item=jd)

TASK [openstack : os_router] *****************************************************************************************************************************************************************************************************************
ok: [localhost] => (item=jd)

TASK [openstack : os_security_group] *********************************************************************************************************************************************************************************************************
ok: [localhost] => (item=jd)

TASK [openstack : os_security_group_rule] ****************************************************************************************************************************************************************************************************
ok: [localhost] => (item=jd)

TASK [openstack : os_security_group_rule] ****************************************************************************************************************************************************************************************************
ok: [localhost] => (item=['jd', '22'])
ok: [localhost] => (item=['jd', '80'])
ok: [localhost] => (item=['jd', '443'])

TASK [openstack : os_client_config] **********************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [openstack : os_server] *****************************************************************************************************************************************************************************************************************
ok: [localhost] => (item=jump01)
ok: [localhost] => (item=bench01)
ok: [localhost] => (item=bench02)
ok: [localhost] => (item=bench03)

TASK [openstack : os_server_facts] ***********************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [openstack : set_fact] ******************************************************************************************************************************************************************************************************************
skipping: [localhost] => (item=bench03) 
skipping: [localhost] => (item=bench02) 
skipping: [localhost] => (item=bench01) 
ok: [localhost] => (item=jump01)

TASK [openstack : add_host] ******************************************************************************************************************************************************************************************************************
skipping: [localhost] => (item=bench03) 
skipping: [localhost] => (item=bench02) 
skipping: [localhost] => (item=bench01) 
changed: [localhost] => (item=jump01)

TASK [openstack : add_host] ******************************************************************************************************************************************************************************************************************
changed: [localhost] => (item=bench03)
changed: [localhost] => (item=bench02)
changed: [localhost] => (item=bench01)
skipping: [localhost] => (item=jump01) 

TASK [openstack : wait for instance to be reachable via ssh] *********************************************************************************************************************************************************************************
skipping: [localhost] => (item=bench03) 
skipping: [localhost] => (item=bench02) 
skipping: [localhost] => (item=bench01) 
ok: [localhost -> localhost] => (item=jump01)

TASK [openstack : debug] *********************************************************************************************************************************************************************************************************************
ok: [localhost] => (item=bench03) => {
    "msg": "bench03: ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -J debian@<EXTERNAL-IP> ubuntu@bench03"
}
ok: [localhost] => (item=bench02) => {
    "msg": "bench02: ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -J debian@<EXTERNAL-IP> debian@bench02"
}
ok: [localhost] => (item=bench01) => {
    "msg": "bench01: ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -J debian@<EXTERNAL-IP> debian@bench01"
}
skipping: [localhost] => (item=jump01) 

TASK [openstack : debug] *********************************************************************************************************************************************************************************************************************
skipping: [localhost] => (item=bench03) 
skipping: [localhost] => (item=bench02) 
skipping: [localhost] => (item=bench01) 
ok: [localhost] => (item=jump01) => {
    "msg": "jump01: debian@<EXTERNAL-IP>"
}

PLAY [bench] *********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************

ok: [bench01]
ok: [bench02]
ok: [bench03]

TASK [apt] ***********************************************************************************************************************************************************************************************************************************

ok: [bench02] => (item=['ethtool', 'iperf'])
ok: [bench03] => (item=['ethtool', 'iperf'])
ok: [bench01] => (item=['ethtool', 'iperf'])

PLAY [bench01] *******************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************

ok: [bench01]

TASK [Start iperf server] ********************************************************************************************************************************************************************************************************************

changed: [bench01]

PLAY [bench02,bench03] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************

ok: [bench02]
ok: [bench03]

TASK [Run iperf client] **********************************************************************************************************************************************************************************************************************

changed: [bench02]
changed: [bench03]

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************
bench01                    : ok=4    changed=1    unreachable=0    failed=0   
bench02                    : ok=4    changed=1    unreachable=0    failed=0   
bench03                    : ok=4    changed=1    unreachable=0    failed=0   
localhost                  : ok=20   changed=2    unreachable=0    failed=0
```

