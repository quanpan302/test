# Get Started, Part 5

[Stacks](https://docs.docker.com/get-started/part5/)

**Understanding Stack**

A stack is a group of interrelated services that share dependencies, and can be orchestrated and scaled together.

A single stack is capable of defining and coordinating the functionality of an entire application (though very complex applications may want to use multiple stacks).

## Add a new service and redeploy

It’s easy to add services to our docker-compose.yml file.
First, let’s add a free visualizer service that lets us look at how our swarm is scheduling containers.

1. Open up `docker-compose.yml` in an editor and replace its contents with the following.
   Be sure to replace `username/repo:tag` with your image details.
   
   ```
   version: "3"
   services:
     web:
       # replace username/repo:tag with your name and image details
       image: quanpan302/test:t01
       deploy:
         replicas: 5
         restart_policy:
           condition: on-failure
         resources:
           limits:
             cpus: "0.1"
             memory: 50M
       ports:
         - "80:80"
       networks:
         - webnet
     visualizer:
       image: dockersamples/visualizer:stable
       ports:
         - "8080:8080"
       volumes:
         - "/var/run/docker.sock:/var/run/docker.sock"
       deploy:
         placement:
           constraints: [node.role == manager]
       networks:
         - webnet
   networks:
     webnet:
   ```
2. Make sure your shell is configured to talk to `myvm1` (full examples are here).
   
   ```
   eval $(docker-machine env myvm1)
   ```
3. Re-run the docker stack deploy command on the manager, and whatever services need updating are updated.
   
   ```
   $ docker stack deploy -c docker-compose.yml getstartedlab
   
   > Updating service getstartedlab_web (id: mxxeto7tgbxtlc19xgd3c4qk7)
   > Creating service getstartedlab_visualizer
   ```
4. Take a look at the visualizer.

   ```
   docker stack ps getstartedlab
   
   > ID                  NAME                         IMAGE                             NODE                DESIRED STATE       CURRENT STATE               ERROR               PORTS
   > mbdgexn3vtxr        getstartedlab_web.1          quanpan302/test:t01               myvm2               Running             Running about an hour ago                       
   > rpkapyuit4kr        getstartedlab_visualizer.1   dockersamples/visualizer:stable   myvm1               Running             Running about an hour ago                       
   > yvn1jaml57a1        getstartedlab_web.2          quanpan302/test:t01               myvm2               Running             Running about an hour ago                       
   > pq26i07nxn6u        getstartedlab_web.3          quanpan302/test:t01               myvm1               Running             Running about an hour ago                       
   > etf0iwb7b3ox        getstartedlab_web.4          quanpan302/test:t01               myvm2               Running             Running about an hour ago                       
   > vd11rzep4f7p        getstartedlab_web.5          quanpan302/test:t01               myvm1               Running             Running about an hour ago                       
   ```

## Persist the data

1. Save this new `docker-compose.yml` file, which finally adds a Redis service.

   ```
   version: "3"
   services:
     web:
     # replace username/repo:tag with your name and image details
       image: quanpan302/test:t01
       deploy:
         replicas: 5
         restart_policy:
           condition: on-failure
         resources:
           limits:
             cpus: "0.1"
             memory: 50M
       ports:
         - "80:80"
       networks:
         - webnet
     visualizer:
       image: dockersamples/visualizer:stable
       ports:
         - "8080:8080"
       volumes:
         - "/var/run/docker.sock:/var/run/docker.sock"
       deploy:
         placement:
           constraints: [node.role == manager]
       networks:
         - webnet
     redis:
       image: redis
       ports:
         - "6379:6379"
       volumes:
         - "/home/docker/data:/data"
       deploy:
         placement:
           constraints: [node.role == manager]
       command: redis-server --appendonly yes
       networks:
         - webnet
   networks:
     webnet:
   ```
   
   Redis has an official image in the Docker library and has been granted the short image name of just `redis`,
   so no `username/repo` notation here.
   
   The Redis port, `6379`, has been _pre-configured_ by Redis to be exposed from the container to the host,
   and here in our Compose file we expose it from the host to the world.
   
   So you can actually enter the IP for any of your nodes into Redis Desktop Manager and manage this Redis instance, if you so choose.
   
   Most importantly, there are a couple of things in the `redis` specification that make data persist between deployments of this stack
   
   * `redis` always runs on the **manager**, so it’s always using the same filesystem.
   * `redis` accesses an arbitrary directory in the **host’s file system** as `/data` inside the container, which is where Redis stores data.
   
   Together, this is creating a “**source of truth**” in your host’s physical filesystem for the Redis data.
   
   Without this, Redis would store its data in `/data` inside the container’s filesystem, which would get wiped out if that container were ever redeployed.
   
   This **source of truth** has two components
   
   * The placement constraint you put on the Redis service, ensuring that it always uses the same host.
   * The volume you created that lets the container access `./data` (on the **host**) as `/data` (inside the **Redis container**).
     _While containers come and go, the files stored on `./data` on the **specified host persists**, enabling continuity._

2. Create a `./data` directory on the manager
   
   ```
   docker-machine ssh myvm1 "mkdir ./data"
   
   docker-machine ssh myvm1 "pwd"
   > /home/docker
   
   docker-machine ssh myvm1 "ls ./"
   > data
   
   docker-machine ssh myvm2 "pwd"
   > /home/docker
   ```

3. Make sure your shell is configured to talk to `myvm1`.
   
   ```
   eval $(docker-machine env myvm1)
   ```
   
4. Run `docker stack deploy` one more time.
   
   ```
   docker stack deploy -c docker-compose.yml getstartedlab
   
   > Creating service getstartedlab_redis
   > Updating service getstartedlab_web (id: sfvb2z150btmh42ldxa792vfj)
   > Updating service getstartedlab_visualizer (id: n9i6hqx89cxdar7z62inawt7u)
   ```

5. Run `docker service ls` to verify that the three services are running as expected.
   
   ```
   docker service ls
   
   > ID                  NAME                       MODE                REPLICAS            IMAGE                             PORTS
   > goz997c65y60        getstartedlab_redis        replicated          1/1                 redis:latest                      *:6379->6379/tcp
   > n9i6hqx89cxd        getstartedlab_visualizer   replicated          1/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
   > sfvb2z150btm        getstartedlab_web          replicated          5/5                 quanpan302/test:t01               *:80->80/tcp
   ```

6. Check the web page at one of your nodes, such as `http://192.168.99.101`,
   and take a look at the results of the visitor counter,
   which is now live and storing information on Redis.
   
   ```
   docker-machine ssh myvm1 "ls data"
   
   > appendonly.aof
   ```
   
   ```
   docker container ls
   
   > CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS              PORTS               NAMES
   > f832f679970c        redis:latest                      "docker-entrypoint..."   12 minutes ago      Up 11 minutes       6379/tcp            getstartedlab_redis.1.ypkcnx1wwnpopow5amjadjt0c
   > 6361fb3ef3a9        quanpan302/test:t01               "python app.py"          2 hours ago         Up 2 hours          80/tcp              getstartedlab_web.3.pq26i07nxn6un1ly3k69yhn7y
   > eb17bde62593        quanpan302/test:t01               "python app.py"          2 hours ago         Up 2 hours          80/tcp              getstartedlab_web.5.vd11rzep4f7p2tnagk2smgn00
   > e4dcb7b6800f        dockersamples/visualizer:stable   "npm start"              2 hours ago         Up 2 hours          8080/tcp            getstartedlab_visualizer.1.rpkapyuit4kr8luga75lzjpz2
   ```
   
   ```
   docker exec -it f832f679970c pwd
   
   > /data

   docker exec -it f832f679970c ls /
   
   > bin  boot  data  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
   ```

_Here are some commands you might like to run to interact with your swarm and your VMs a bit_

```
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1                  # Win10
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows command to connect shell to myvm1
==============
docker-machine create --driver virtualbox myvm1                                           # Create a VM myvm1 (Mac, Win7, Linux)
docker-machine create --driver virtualbox myvm2                                           # Create a VM myvm2 (Mac, Win7, Linux)
==============
docker-machine ls                                                                         # list VMs, asterisk shows which VM this shell is talking to
==============
docker-machine env myvm1                                                                  # View basic information about your node, show environment variables and command for myvm1
                                                                                          # myvm1 ACTIVE -
docker-machine ssh myvm1                                                                  # Open an SSH session with the VM; type "exit" to end
docker-machine ssh myvm1 "docker swarm init --advertise-addr <node manager ip> "          # Manager
docker-machine ssh myvm2 "docker swarm join --token <SWMTKN> <node manager ip>:<port>"    # Worker
docker-machine ssh myvm1 "docker node ls"                                                 # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"                                  # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"                              # View join token
==============
eval $(docker-machine env myvm1)                                                          # Mac command to connect shell to myvm1
                                                                                          # myvm1 ACTIVE *
docker node ls                                                                            # View nodes in swarm (while logged on to manager)
==============
eval $(docker-machine env -u)                                                             # Disconnect shell from VMs, use native docker
==============
docker-machine ssh myvm2 "docker swarm leave"                                             # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f"                                          # Make master leave, kill swarm
==============
docker-machine stop myvm1                                                                 # Stop a VM that is currently not running
docker-machine start myvm1                                                                # Start a VM that is currently not running
==============
docker-machine stop $(docker-machine ls -q)                                               # Stop all running VMs
docker-machine rm $(docker-machine ls -q)                                                 # Delete all VMs and their disk images
==============
docker stack deploy -c docker-compose.yml getstartedlab                                   # Deploy an app;
docker stack deploy -c <file> <app>                                                       # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker stack ps <app>                                                                     # List
docker stack rm <app>                                                                     # Remove
docker-machine scp docker-compose.yml myvm1:~                                             # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"                            # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
```
