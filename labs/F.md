# Lab F -  Services and Application Architectures

## Learning Objectives

In this lab you will:

- Learn about multi-tier application architectures.
- Understand how to monitor and check the logs of a service.

### Lab Setup At A Glance

In this lab, we will deploy several different setups. This time there will not be a default `docker-compose.yml` in your lab folder, but rather one for each application architecture we deploy.

This introduces the `-f` option to the `docker-compose` command to allow the user to provide a configuration in place of the standard `docker-compose.yml` file.

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

Walk through a simple service example where you setup a static website using nginx [https://www.nginx.com/](https://www.nginx.com/). This is a classic example of a 2-tier client-server application architecture with web browser on your host being the client and the Docker container running the Nginx web server being the server of course.

Inside the container we will expose TCP port 80, the well-known port for the HTTP protocol to the host. This allows the web browser running on the host to have access to the Nginx web server runnin inside the Docker container.

```
+-------------------+
| Docker: nginx     |--+
|http://webserver:80|  |
+-------------------+  |  +-------------------+   ++++++++++++++
                       +--|host computer      |---|  Internet  |
                          |http://localhost:80|   ++++++++++++++
                          +-------------------+
```

1. Let's bring up the environment:  
`docker-compose -f 2-tier.yml up -d`  
Notice how we've included `-f 2-tier.yml` to the command to specify that specific docker-compose file.
1. Let's see what's running:  
`docker-compose -f 2-tier.yml ps`  
The output should reveal that a container named `nginx_webserver` is running. Furthermore the container has TCP ports 22 and 80 in use, but only port 80 is exposed to the host `0.0.0.0:80 -> 80/tcp`
1. Let's view our website:  
Open up a browser, like chrome on your host, and enter the following address: `http://localhost:80` you should see the **TechMag**



## Part 2: Three Tier Service 

Setup Django or let's chat or something like that.

## Part 3: N-Tier Service

Django with a nginx front end.



## Questions 

