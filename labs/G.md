# Lab G - Scalability

## Learning Objectives

In this lab you will:

- Learn about horizontal scalability.
- Understand how a load balancer works.
- Learn to consolidate logging in a horizontally scaled environment.

Load balancer play with settings learn how it works.

consolidate logs to syslog by overriding config.


setup to ensure compose can access the docker engine


`$Env:COMPOSE_CONVERT_WINDOWS_PATHS=1`

commands
docker-compose up -d
docker-compose ps
docker-compl



Checking on the status of haproxy

http://localhost:1936

user: stats
pass: stats

witness the round-robin effect in all its gory-glory!

TODO - FORCE JAVASCRIPT PAGE RELOAD SO STUDENTS DONT HAVE TO???


## Load Balancing Algorithms 

round robin -
everyone gets a turn start with 3
scale up to 10
scale down to 2 
notice the web app still works!

lowest number of connections (leastconn)
down and up with leastconn
force a delay on one client: http://localhost/slow/5
refresh another client quickly notice it does not use one of the hosts because its busy.
When the delay completes, it's the host that was missing is revealed, of course.

uri - the request URI is hashed the same URI maps to the same app... unless the number of app change. 