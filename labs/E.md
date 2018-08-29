# Lab E - Workstations and Clients

docker-compose scale workstation=5

docker exec -it lab-e_workstation_1 bash 

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
