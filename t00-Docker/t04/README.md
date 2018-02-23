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
	image: username/repo:tag
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