# check_mk-ansible-livestatus_query

## Purpose:
Generate csv report from severall check_mk remote sites using **lq** (livestatus query binary) and publishing it on apache web server using ansible. 

## Components of this project:
* inventory
* playbook

### Inventory
Has next template. It can be customized as you need. 
```
[all]
hostname1 site_name=SITE_NAME1
hostname2 site_name=SITE_NAME2
...
hostnameN site_name=SITE_NAMEN
hostname_webserver


[all:vars]
ansible_connection=ssh
ansible_ssh_user=root
ansible_ssh_pass=your_root_password
host_key_checking=false


[check_mk_sites]
hostname1
hostname2
...
hostnameN


[webserver]
hostname_webserver
```
