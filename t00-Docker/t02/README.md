# Get Started, Part 3

[Services](http://devdocs.io/docker~17/get-started/part3/index)

## About services

In a distributed application, different pieces of the app are called “services.”

For example, if you imagine a video sharing site, it probably includes

* a service for storing application data in a database,
* a service for video transcoding in the background after a user uploads something,
* a service for the front-end, and so on.

Services are really just “containers in production.”

**A service** only runs **one image**,
but it codifies the way that image runs—what ports it should use,
how many replicas of the container should run so the service has the capacity it needs, and so on.

Scaling a service changes the number of container instances running that piece of software,
assigning more computing resources to the service in the process.

## Your first `docker-compose.yml` file

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: quanpan302/test:t01
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```

This docker-compose.yml file tells Docker to do the following:

* Pull the image we uploaded in step 2 from the registry.
* Run 5 instances of that image as a service called web, limiting each one to use, at most, 10% of the CPU (across all cores), and 50MB of RAM.
* Immediately restart containers if one fails.
* Map port 80 on the host to web’s port 80.
* Instruct web’s containers to share port 80 via a load-balanced network called webnet. (Internally, the containers themselves will publish to web’s port 80 at an ephemeral port.)
* Define the webnet network with the default settings (which is a load-balanced overlay network).

> **Wondering about Compose file versions, names, and commands?**
> 
> Notice that we set the Compose file to `version: "3"`.
> 
> This essentially makes it [swarm mode](http://devdocs.io/docker~17/engine/swarm/index) compatible.
> 
> * We can make use of the deploy key (only available on Compose file formats version 3.x and up) and its sub-options to load balance and optimize performance for each service (e.g., `web`).
> * We can run the file with the `docker stack deploy` command (also only supported on Compose files version 3.x and up).
> * You could use `docker-compose up` to run version 3 files with non swarm configurations, but we are focusing on a stack deployment since we are building up to a swarm example.

*You can name the Compose file anything you want to make it logically meaningful to you; `docker-compose.yml` is simply a standard name. We could just as easily have called this file `docker-stack.yml` or something more specific to our project.*

## Run your new load-balanced app

Before we can use the `docker stack deploy` command we’ll first run:

```
docker swarm init

> Swarm initialized: current node (b9b291l98zad6r85nvcbn3fra) is now a manager.
> 
> To add a worker to this swarm, run the following command:
> 
>     docker swarm join --token SWMTKN-1-4o0i9ogmno6xo9qlpdhrq7keiphvtom1h3bqm95rk9cofrpnhl-819g17ckwtthfbvmq7svx57b9 192.168.65.2:2377
> 
> To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Now let’s run it. You have to give your app a name. Here, it is set to t02:

```
docker stack deploy -c docker-compose.yml getstartedlab

> Creating network getstartedlab_webnet
> Creating service getstartedlab_web
```

Our single service stack is running **5 container instances** of our deployed image on one host.
Let’s investigate.

Get the service ID for the one service in our application:

```
docker service ls

> ID                  NAME                MODE                REPLICAS            IMAGE                 PORTS
> ukuvi65iw7ht        getstartedlab_web   replicated          0/5                 quanpan302/test:t01   *:80->80/tcp
```

Docker swarms run tasks that spawn containers. Tasks have state and their own IDs:

```
docker service ps <service>

docker service ps getstartedlab_web

> ID                  NAME                  IMAGE                 NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
> m73z0czmm6ri        getstartedlab_web.1   quanpan302/test:t01   moby                Running             Running 2 minutes ago                       
> rpj5sc4jppxv        getstartedlab_web.2   quanpan302/test:t01   moby                Running             Running 2 minutes ago                       
> gdv8a2d9f5yi        getstartedlab_web.3   quanpan302/test:t01   moby                Running             Running 2 minutes ago                       
> waegr2sq9ipu        getstartedlab_web.4   quanpan302/test:t01   moby                Running             Running 2 minutes ago                       
> zh9eso27db5v        getstartedlab_web.5   quanpan302/test:t01   moby                Running             Running 2 minutes ago    
```

Now list all 5 containers:

```
docker container ls -q

> b61beede0eff
> 11fdcfbc107b
> 6635b9c6785e
> 36c1df60267b
> 5342aac81a46
```

You can run curl `http://localhost` several times in a row, or go to that URL in your browser and hit refresh a few times.
Either way, you’ll see the container ID change, demonstrating the **load-balancing**; with each request, one of the 5 replicas is chosen, in a round-robin fashion, to respond.

> Let’s inspect one task and limit the ouput to container ID:

```
docker inspect --format='{{.Status.ContainerStatus.ContainerID}}' <task>
```

> Vice versa, inspect the container ID, and extract the task ID:

```
docker inspect --format="{{index .Config.Labels \"com.docker.swarm.task.id\"}}" <container>
```

## Scale the app

You can scale the app by changing the `replicas` value in `docker-compose.yml`, saving the change, and re-running the `docker stack deploy` command:

```
docker stack deploy -c docker-compose.yml getstartedlab
```

Docker will do an in-place update, no need to tear the stack down first or kill any containers.

Now, re-run `docker container ls -q` to see the deployed instances **reconfigured**.
If you scaled up the replicas, more tasks, and hence, more containers, are started.

## Take down the app and the swarm

Take the app down with `docker stack rm`:

```
docker stack rm getstartedlab

> Removing service getstartedlab_web
> Removing network getstartedlab_webnet
```

Take down the swarm.

```
docker swarm leave --force

> Node left the swarm.


docker service ls

> Error response from daemon: This node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
```

This removes the app, but our one-node swarm is still up and running (as shown by `docker node ls`).
Take down the swarm with `docker swarm leave --force`.

It’s as easy as that to stand up and scale your app with Docker.
You’ve taken a huge step towards learning how to run containers in production.
Up next, you will learn how to run this app as a bonafide swarm on a cluster of Docker machines.

> **Note**: Compose files like this are used to define applications with Docker, 
> and can be uploaded to cloud providers using **Docker Cloud**,
> or on any hardware or cloud provider you choose with **Docker Enterprise Edition**.

## Recap and cheat sheet (optional)

_Here are some commands you might like to run to interact with your swarm and your VMs a bit_

```
docker stack ls                                 # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                               # List running services associated with an app
docker service ps <service>                     # List tasks associated with an app
docker inspect <task or container>              # Inspect task or container
docker container ls -q                          # List container IDs
docker stack rm <appname>                       # Tear down an application
```
