# percona-patroni-ha

## Descripton ##
percona-patroni-ha role to install postgres with High Availability

## Pre-reqs ##

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
postgres_bin_dir=/usr/lib/postgresql/{{ postgres_version }}/bin/  ## this is needed for patroni configuration
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
## Entries in Inventory file ##

- **keep the same order for both groups. **
- ** for etcd3 it really doesn't matter much but etcd2 executes in serial, so it does matter.
```bash
ex:
Correct order:
  if patroni:
       db01
       db02
       db03
  then
      etcd:
        db01
        db02
        db03

Wrong order:
  if patroni:
       db01
       db02
       db03
then  etcd:
        db03
        db02
        db01
```

## Exclude PostgreSQL installation if it is already installed. ##

Most of the time we just have to Configure patroni as Postgresql already installed in client environment. 
In that case we need to set install_postgresql=false in host variables.

## Etcd Version ##

ectd latest stable version is 3. Use version2 only in specific cases. For new installation use version 3.

##  and postgres_data_dir variable ##
Please configure data directory if it different from /var/lib/postgresql

## Deploy with etcd2 ##

ectd2 installation is slower compared with etcd3 due to it serial execution.

  ```bash
   ansible-playbook -i inventory.ini percona-patroni2.yaml --tags=deploy
   ```
or
  ```bash
   ansible-playbook -i inventory.yaml percona-patroni2.yaml --tags=deploy
   ```
## Deploy etcd3 ##
  ```bash
   ansible-playbook -i inventory.ini percona-patroni3.yaml --tags=deploy
   ```
or
  ```bash
   ansible-playbook -i inventory.yaml percona-patroni3.yaml --tags=deploy
   ```


## Sample output ##
```bash
TASK [/home/percona/roles/patroni : Print Patroni final output] *****************************************************************************************************************************************************************************************
skipping: [prasanth-db-01]
skipping: [prasanth-db-02]
ok: [prasanth-db-03] => {
    "msg": [
        "+ Cluster: test (7436504368086593508) ---+-----------+----+-----------+",
        "| Member         | Host        | Role    | State     | TL | Lag in MB |",
        "+----------------+-------------+---------+-----------+----+-----------+",
        "| ip-*********** | *********** | Leader  | running   |  1 |           |",
        "| ip-*********** | *********** | Replica | streaming |  1 |         0 |",
        "| ip-*********** | *********** | Replica | streaming |  1 |         0 |",
        "+----------------+-------------+---------+-----------+----+-----------+"
    ]
}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************************
prasanth-db-01             : ok=28   changed=14   unreachable=0    failed=0    skipped=19   rescued=0    ignored=2
prasanth-db-02             : ok=28   changed=15   unreachable=0    failed=0    skipped=19   rescued=0    ignored=1
prasanth-db-03             : ok=31   changed=17   unreachable=0    failed=0    skipped=16   rescued=0    ignored=1
```


## Cleaning up ##
tag: clean remove the all installation from db nodes. Patroni, etcd, Postgres will be removed.
     clean is disabled by default. To enable it patroni/tasks/main.yaml and uncomment task for clean
     Use it for only test nodes.

  ```bash
   ansible-playbook -i inventory.yaml percona-patroni3.yaml --tags=clean
   ```




