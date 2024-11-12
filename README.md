# percona-patroni-ha

## Descripton ##
percona-patroni-ha role to install postgres with High Availability

## Pre-reqs ##
watch dog:  After 
```bash
sudo sh -c 'echo "softdog" >> /etc/modules'
sudo sh -c 'echo "KERNEL=="watchdog", OWNER="postgres", GROUP="postgres"" >> /etc/udev/rules.d/61-watchdog.rules'
grep blacklist /lib/modprobe.d/* /etc/modprobe.d/* |grep softdog
for file in $(grep blacklist /lib/modprobe.d/* /etc/modprobe.d/* |grep softdog|cut -d":" -f1); do sed -i 's/softdog//g' $file;done
grep blacklist /lib/modprobe.d/* /etc/modprobe.d/* |grep softdog
```

- **server/node restart required after enabing watchdog**

## Tags: ##
```bash
deploy: use to deploy the whole cluster
etcd: use to install etcd only on nodes
postgresql: use to install postgresql only
clean: run only on new or none prod server. this will remove all packages and delete postgres direcotry (/var/lib/postgresql/)
```
## files ##
```bash
role: patroni
inventory file:  inventory.ini
etcd2 play: percona-patroni2.yaml
etcd3 play: percona-patroni3.yaml
```
## variables ##
```bash

[patroni:vars]
patroni_version=3
patroni_data_dir=/etc/patroni
postgres_bin_dir=/usr/lib/postgresql/14/bin/  ## this is needed for patroni configuration
percona_release=ppg14
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
install_postgresql=true      ## if you want postgresql to initialize by patroni. If postgresql already install make it false
postgres_data_dir=/var/lib/postgresql/{{ postgres_version }}/main ## this is needed for patroni configuration
postgres_version=14
db_connections=200
shared_memory=2GB
wallevel=replica
wal_senders=10
hotstandby=on
patroni_scope=test  ## patroni cluster name
patroni_port=8008
etcd_port=2379
replication_slots=10
db_port=5432
replicator_user=replicator
replication_password=replication
postgres_superuser=postgres
postgres_superuserpassword=postgres
no_failover=false
no_loadbalancer=false
watch_dog_mode='disable'   ## in order to enable watchdog, you need to follow prereq steps to enable watch dod at OS level.
boot_strap_watch_dog=false  ##this will disable watch dog while bootstraping postgres

[etcd:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
etcd_version=3 ### this defines the version of patroni
```

## Deploy with etcd2 ##

ectd2 installation is slower compared with etcd3 due to it serial execution.

  ```bash
   ansible-playbook -i inventory.ini percona-patroni2.yaml --tags=deploy
   ```
## Deploy etcd3 ##
  ```bash
   ansible-playbook -i inventory.ini percona-patroni3.yaml --tags=deploy
   ```
