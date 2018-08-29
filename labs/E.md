# Lab E - Workstations and Clients


```
+-------------+
| workstations|----+
| 172.44.1.2  |    |
| through     |    |
| 172.44.1.6  |    |
+-------------+    |
                   |   +---------------+     ++++++++++++++
                   +---| host computer |-----|  Internet  |
                   |   +---------------+     ++++++++++++++
+--------------+   |
|   server     |---+
| 172.44.1.254 |
+--------------+
```

spin up 5 workstations

docker-compose up -d --scale workstation=5

docker exec -it lab-e_workstation_1 bash 

install alpine  with apt-get


How can we do this to all 5 machines? Ansible is a better way

setup `/etc/ansible/hosts`
add 
[workstations]
lab-e_workstation_[1:5]

Follow this for the most part
https://serversforhackers.com/c/an-ansible-tutorial
