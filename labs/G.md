# Lab G - Scalability

## Learning Objectives

In this lab you will:

- Learn about horizontal scalability.
- Understand how a load balancer works.
- Learn to consolidate logging in a horizontally scaled environment.

### Lab Setup At A Glance

In this lab we will demonstrate how applications are scaled horizontally. Specifically we will run a web application built using the [Flask Web Framework](http://flask.pocoo.org) through a load balancer. The Load balancer we will use is called [HAProxy](http://www.haproxy.org/ ). It is a very fast, reliable load balancer which comes with a variety of configurable algorithms for balancing load across servers on a network. We will spin up multiple instances of the same Flask web application behind the load balancer in so that we can get a clear picture of how traffic is distributed by a load-balancing application like HAProxy.

```
          +----LOAD BALANCER--+   +---CLIENT:BROWSER----+   ++++++++++++++
          | Docker: haproxy   |---| host computer       |---|  Internet  |
   +------|http://haproxy:80  |   | http://localhost:80 |   ++++++++++++++
   |      +-------------------+   +---------------------+  
   |
+----SERVER:Flask----+
| Docker: webapp     |--+
| http://webapp:8080 |  | 
+--------------------+  |
   | 1 or more instances|  
   +--------------------+
```
## Before you begin 

### Prep your lab environment. 

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. IMPORTANT: This lab requires access to Docker's internals, you must enter this command:
`PS ist346-labs> $Env:COMPOSE_CONVERT_WINDOWS_PATHS=1`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-G` folder:  
`PS ist346-labs> cd lab-G`
5. Start the lab environment in Docker:  
`PS ist346-labs\lab-G> docker-compose up -d`
6. Verify your `haproxy` load balancer and `webapp` are up and running:
`PS ist346-labs\lab-G> docker-compose ps`  
You should see `haproxy` has tcp ports `80` and `1936` open. The `webapp` is running on tcp port `8080`.
8. Open a web browser like Chrome on your host. Enter the address `http://localhost` and you will see the sample web application:  
![sample web application screenshot](images/lab-g-webapp.png)  

**NOTE:** The webpage is designed to reload every 3 seconds so that you can see what happens on subsequent HTTP requests. No need to hit the refresh button in your browser!

### Understanding the Sample Web Applications

There's some really important information page of the **Sample Web Application**. It's designed to help you understand the behavior of the load balancer.

- **HOSTNAME** This indicates the host name of the docker container which served the page. Currently on page refresh we get the same host name each time. That's because there is only one `webapp` container running on the backend of the load balancer.
- **IP Address** This displays the IP Address of the docker container which served the page. Again you will see the same IP address on each page refresh because we only have one container.
- **Current Date/Time** This displays the current date/time on the server. This information should change with each request. Its primary purpose is to help you see that new content is being loaded with each request.

### HAProxy Statistics Report

The HAProxy load balancer has a special web UI for inspecting the traffic on the load balancer. Let's open it up.

1. Open a new window in your web browser. Typically `CTRL` + `n` does this. Keep your other tab showing the **Sample Web Application** open.
1. Enter the address: `http://localhost:1936` into your browser's address bar.
1. You will prompted for a username and password enter `stats` for both.

You will now see a **statistics report**. HA Proxy consists of two parts a **frontend** and a **backend**. The front end is the part of the load balancer exposed to the internet. In this example the frontend uses port 80. The backend consists of 1 or more applications which are load-balanced. These appear under the heading `default_service` and if everything is running as it should they will appear in green. The only service on the backend at this point is `lab-g-webapp_1`

While you have been reading this, the **Sample Web Application** has reloaded a few times. If you click `Refresh Now` in the **statistics report** you see the number of sessions and bytes in/out increase. 

### Scaling the app

Let's scale our app to 3 instances and then observe what happens.

1. To scale the `webapp` service so there are 3 instances (instead of 1), we type:
`PS ist346-labs\lab-G> docker-compose scale webapp=3` 
1. Go back to your browser running the **Statistics Report** and click the `Refresh Now` link. You should now see 3 instances of the `lab-g-_webapp` container. We have scaled the app.
1. Go back to your browser running the **Sample Web Application** on `http://localhost`. Notice now when the page refreshes, on each request you get one of three different HOSTNAMEs and IP Addresses. 
1. The load balancer cycles through each of the 3 docker containers in a predictable pattern. That's because the load balancer is configured to distribute the load using the `roundrobin` algorithm. You can learn more about it here: [http://cbonte.github.io/haproxy-dconv/1.9/configuration.html#balance](http://cbonte.github.io/haproxy-dconv/1.9/configuration.html#balance).

### Turning it up to 11

If your app is designed to scale horizontally, then you should be able to scale it infinitely. Large Hadoop clusters, a system designed to scale horizontally, have thousands of nodes. The problem, of course, is writing an app to scale horizontally is non a trivial task. The biggest issue around making data available to each node participating on the back end and dealing with updates to that data.

1. Good thing our app is simple enough to scale in this fashion, so let's scale the app to 11!  
`PS ist346-labs\lab-G> docker-compose scale webapp=11` 
1. Go back to your **Statistics Report** and click the `Refresh Now` link several times. Observe how each of the 11 nodes on the backend participate in the process (look at the Session Total column).

### What comes up, must come down.

Besides balancing the load, load balancers act as as a proxy between the browser and your application. As such you can scale backend app up without ever denying a user service. We scaled from 1 node on the backend to 3 nodes, to 11 without ever bringing the **Sample Web Application** down. When you scale down, its not so easy as a client may have been using a node you're decommissioning.

1. Let's scale it down to 3:  
`PS ist346-labs\lab-G> docker-compose scale webapp=3`
1. Then to be sure that HA Proxy is restarted to flush the other nodes:  
`PS ist346-labs\lab-G> docker-compose restart haproxy`
1. Let's check on  **Sample Web Application** on `http://localhost`. It might still be running, it might have a page error, in which a manual refresh will fix things. If it has a page error, then it requested a node as it was brought down.

## Different Load Balancing Algorithms

As we explained in the previous section the default load balancing algorithm uses **round robin**. Let's explore two other load balancing algorithms `leastconn` and `uri`.

### leastconn

The leastconn algorithm selects the instance with the least number of connections. If a node is busy serving a client, the next request will not use that node but instead select another available node. Let's demonstrate this.

1. First bring down your existing environment, type:  
`PS ist346-labs\lab-G> docker-compose down`
1. Next we need to edit the `docker-compose.yml` file to adjust the algorithm that `haproxy` uses. Open the file in the `notepad` text editor:  
`PS ist346-labs\lab-G> notepad docker-compose.yml`
1. When the text editor opens, find the line that says:  
`- BALANCE=roundrobin`  
and change it to:  
`- BALANCE=leastconn`  
Then close the window and save your changes.
1. Let's bring up the environment with a scale of 3:  
`PS ist346-labs\lab-G> docker-compose up -d --scale webapp=3`
1. Now let's return to the browser where we have **Sample Web Application** running on `http://localhost` t  first glance, it seems to work the same as before, rotating evenly through each instance.
1. Let's put that to the test. We are going to access another url which keeps the instance busy so that the other browser cannot use that connection. Open another browser in a new window (`CTRL` + `n` does this). And arrange the windows so you can see both at the same time. In the new window let's request the **Sample Web Application** but instead use this url: `http://localhost/slow/10` 
1. You'll see the browser accessing `http://localhost` now only uses TWO of the three instances. The other instance is busy fulfilling the `http://localhost/slow/10` request! When that request finishes, the other browser will once again use all three instances to fulfill requests.

## uri

The `uri` algorithm selects an instance based on a hash of the Uri (Uniform Resource Indicator). This algorithm differs from the `leastconn` or `roundrobin`, in that a given Uri will always map to the same instance. Let's see a demo

1. First bring down your existing environment, type:  
`PS ist346-labs\lab-G> docker-compose down`
1. Next we need to edit the `docker-compose.yml` file to adjust the algorithm that `haproxy` uses. Open the file in the `notepad` text editor:  
`PS ist346-labs\lab-G> notepad docker-compose.yml`
1. When the text editor opens, find the line that says:  
`- BALANCE=leastconn`  
and change it to:  
`- BALANCE=uri`  
Then close the window and save your changes.
1. Let's bring up the environment with a scale of 3:  
`PS ist346-labs\lab-G> docker-compose up -d --scale webapp=3`
1. Now let's return to the browser where we have **Sample Web Application** running on `http://localhost` notice how with every page load we get the same instance. This is because that Uri maps to the instance you see.
1. Let's try another Uri in the browser, type `http://localhost/a` you will get a different instance. If you re-load the page (you must do this manually) you will still get the same instance for this `uri`.
1. Let's try a 3rd Uri in the browser, type `http://localhost/b` you will get yet another different instance. If you re-load the page (you must do this manually) you will still get the same instance for this `uri`.
1. Going back to either `http://localhost`, `http://localhost/a`, or `http://localhost/b` will always yield a response from the same instance. That's how uri mapping works!

There are other algorithms which in this manner. Consider their applications. Imagine distributing load based on geographical location, browser type, operating system, user attributes, etc... This offers a greater degree of flexibility for how we balance load.

## Tear down

This concludes our lab. Time for a tear down!

1. From the PowerShell prompt, type:
`PS ist346-labs\lab-G> docker-compose down`  
to tear down the docker containers used in this lab.

## Questions

1. How is horizontal scalability as demonstrated in this lab different from the vertical scalability of the previous lab?
1. What do we call multiple copies of the service we scale horizontally?
1. What are the two things HA Proxy does as explained in this lab?
1. Why is scaling out to more nodes/instances easier than scaling back to fewer nodes/instances?
1. Type the command to scale the service `db` to `7` instances.
1. Type the command bring up an environment and scale service `foo` to `8` instances.
1. How does the `leastconn` algorithm differ from the `roundrobin` algorithm? How are they similar?
1. What are some potential uses of the `uri` load balancer algorithm?
1. In spite of being load balanced, does our environment still have a single point of failure? If so, what is it? How can this be remedied?