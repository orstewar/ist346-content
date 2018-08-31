# Lab F -  Services and Application Architectures

## Learning Objectives

In this lab you will:

- Learn about multi-tier application architectures.
- Understand how to monitor and check the logs of a service.

### Lab Setup At A Glance

In this lab, we will have one server computer, running Ansible, and 5 identical workstation computers. We will use the docker-compose `scale` option to create 5  containers from the same workstation image. The IP Addresses of the workstations and server are included in the diagram for refrence. 

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

## Before you begin 

### Prep your lab environment. 

This lab prep's a little different as we will be scaling the workstation image to use 5 containers.

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-F` folder:  
`PS ist346-labs> cd lab-F`
5. View the Docker compose files for this lab:  
`PS ist346-labs\lab-F> dir *.yml`


## Part 1: Two-Tier Service

Walk through a simple service example where you setup a static website using nginx. 2-tier client-server


## Part 2: Three Tier Service 

Setup Django or let's chat or something like that.

## Part 3: N-Tier Service

Django with a nginx front end.



## Questions 

