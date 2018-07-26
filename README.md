# openstack-wrapper

This Role is a wrapper for [ansible cloud openstack modules](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#openstack). 
It deploys (multiple) ssh-keys, images, networks, security groups and instances. Every instance will be addded to your inventory and usable for further tasks.
The goal is (hopefully) a more clear usage for deployments.

## Requirements

This role depends on [OpenStack clouds.yaml](https://docs.openstack.org/python-openstackclient/pike/configuration/index.html#clouds-yaml).
After configurating clouds.yaml your cloud can be set by '''os_cloud''' variable.
```
os_cloud: fancycloud
```

## Role Variables

### deploy ssh keys
```
os_keys:
  key1:
    file: /tmp/id_rsa1.pub
  key2:
    file: /tmp/id_rsa2.pub
```
### deploy images
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
### deploy networks
os_nets:
  jd:
    net: 10.42.42.0/24
    ext: ext02
A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

### deploy security groups
Currently only one security group can be created.
```
os_ports: [ '22', '80', '443' ]
```

### deploy instances
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

### add instances to inventory
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

### Tags
You can run specific tasks by using one or more following tags:
* os
* os:keys
* os:img
* os:net
* os:srv
* os:inv

## Example Playbook

[bench.yaml.example](https://github.com/derJD/ansible-openstack-wrapper/blob/master/bench.yaml.example)
deploys a jumphost and three benchmark nodes.
iperf and ethtool is istalled on all benchmark nodes. After that iperf-server is started on bench01 and client is triggered on the other nodes.
[galera.yaml.example](https://github.com/derJD/ansible-openstack-wrapper/blob/master/galera.yaml.example)
deploys a jumphost and three galera nodes. galera and mariadb packages are installed. 

```
# deploy misc node and install htop on it
- hosts: localhost
  vars:
    os_cloud: fancycloud
    os_keys:
      jd-github:
        file: /home/derJD/.ssh/id_rsa.pub
    os_imgs:
      debian-9-openstack:
        file: /tmp/debian-9-openstack-amd64.qcow2
    os_nets:
      misc:
        net: 10.42.42.0/24
        ext: ext42
    os_ports: [ '22', '80', '443' ]
    os_srvs:
      misc01:
        groups:
          - 'instances'
          - 'misc'
        image: debian-9-openstack
        flavor: falvor.42
        key: jd-github
        net: misc
        fip: 'yes'
  roles:
    - derJD.openstack_wrapper

- hosts: misc
  tasks:
    - apt: name=htop state=present
```

## License

BSD

## Author Information

It's my first role.
