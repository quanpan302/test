# Get Started, Part 4

[Swarms](https://docs.docker.com/get-started/part4/)

**Understanding Swarm clusters**

A swarm is a group of machines that are running Docker and joined into a cluster.
After that has happened, you continue to run the Docker commands you’re used to,
but now they are executed on a cluster by a **swarm manager**.
The machines in a swarm can be physical or virtual.
After joining a swarm, they are referred to as **nodes**.

Swarm managers can use several strategies to run containers,
such as “emptiest node” – which fills the least utilized machines with containers.
Or “global”, which ensures that each machine gets exactly one instance of the specified container.
You instruct the swarm manager to use these strategies in the Compose file, just like the one you have already been using.

Swarm managers are the only machines in a swarm that can execute your commands,
or authorize other machines to join the swarm as **workers**.
Workers are just there to provide capacity and do not have the authority to tell any other machine what it can and cannot do.

*Up until now, you have been using Docker in a single-host mode on your local machine.*
But Docker also can be switched into **swarm mode**, and that’s what enables the use of swarms.
Enabling swarm mode instantly makes the current machine a swarm manager.
From then on, Docker runs the commands you execute on the swarm you’re managing, rather than just on the current machine.

## Set up your swarm

A swarm is made up of multiple nodes, which can be either **physical** or **virtual machines**.

The basic concept is simple enough

1. run `docker swarm init` to enable swarm mode and make your current machine a swarm manager.
2. run `docker swarm join` on other machines to have them join the swarm as workers.

_We use **VMs** to quickly create a two-machine cluster and turn it into a swarm._

### Create a cluster

**VMS ON YOUR LOCAL MACHINE (MAC, LINUX, WINDOWS 7 AND 8)**

You need a hypervisor that can create virtual machines (VMs), so install Oracle **VirtualBox** for your machine’s OS.

Now, create a couple of VMs using `docker-machine`, using the VirtualBox driver:

```
docker-machine create --driver virtualbox myvm1

> Creating CA: /Users/quanpan/.docker/machine/certs/ca.pem
> Creating client certificate: /Users/quanpan/.docker/machine/certs/cert.pem
> Running pre-create checks...
> (myvm1) Image cache directory does not exist, creating it at /Users/quanpan/.docker/machine/cache...
> (myvm1) No default Boot2Docker ISO found locally, downloading the latest release...
> (myvm1) Latest release for github.com/boot2docker/boot2docker is v18.02.0-ce
> (myvm1) Downloading /Users/quanpan/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.02.0-ce/boot2docker.iso...
> (myvm1) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
> Creating machine...
> (myvm1) Copying /Users/quanpan/.docker/machine/cache/boot2docker.iso to /Users/quanpan/.docker/machine/machines/myvm1/boot2docker.iso...
> (myvm1) Creating VirtualBox VM...
> (myvm1) Creating SSH key...
> (myvm1) Starting the VM...
> (myvm1) Check network to re-create if needed...
> (myvm1) Found a new host-only adapter: "vboxnet0"
> (myvm1) Waiting for an IP...
> Waiting for machine to be running, this may take a few minutes...
> Detecting operating system of created instance...
> Waiting for SSH to be available...
> Detecting the provisioner...
> Provisioning with boot2docker...
> Copying certs to the local machine directory...
> Copying certs to the remote machine...
> Setting Docker configuration on the remote daemon...
> Checking connection to Docker...
> Docker is up and running!
> To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env myvm1
```

```
docker-machine create --driver virtualbox myvm2

> Running pre-create checks...
> Creating machine...
> (myvm2) Copying /Users/quanpan/.docker/machine/cache/boot2docker.iso to /Users/quanpan/.docker/machine/machines/myvm2/boot2docker.iso...
> (myvm2) Creating VirtualBox VM...
> (myvm2) Creating SSH key...
> (myvm2) Starting the VM...
> (myvm2) Check network to re-create if needed...
> (myvm2) Waiting for an IP...
> Waiting for machine to be running, this may take a few minutes...
> Detecting operating system of created instance...
> Waiting for SSH to be available...
> Detecting the provisioner...
> Provisioning with boot2docker...
> Copying certs to the local machine directory...
> Copying certs to the remote machine...
> Setting Docker configuration on the remote daemon...
> Checking connection to Docker...
> Docker is up and running!
> To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env myvm2
```

**LIST THE VMS AND GET THEIR IP ADDRESSES**

You now have two VMs created, named `myvm1` and `myvm2`.

Use this command to list the machines and get their IP addresses.

```
docker-machine ls

> NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
> myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v18.02.0-ce   
> myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.02.0-ce   
```

> 1. **myvm1** to become a **swarm manager**.
> 2. **myvm2** to join swarm, become a **worker**.

**INITIALIZE THE SWARM**

The first machine acts as the **manager**,
which executes management commands and authenticates workers to join the swarm, and the second is a **worker**.

You can send commands to your VMs using `docker-machine ssh`.
Instruct **myvm1** to become a **swarm manager** with `docker swarm init` and look for output like this.

```
docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.99.100"

> Swarm initialized: current node (zsx6r0dx9ew8nx2ejjkpq80hk) is now a manager.
> 
> To add a worker to this swarm, run the following command:
> 
>     docker swarm join --token SWMTKN-1-50a1ogovr3uy9wj0uyk1y38t78acqstib8bry81t1zgzayu9ge-dagwgu5hsqm1lidl1td67qc40 192.168.99.100:2377
> 
> To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

> Ports `2377` and `2376`
> 
> Always run `docker swarm init` and `docker swarm join` with port 2377 (the swarm management port), or no port at all and let it take the default.
> 
> The machine IP addresses returned by `docker-machine ls` include port 2376, which is the Docker daemon port.
> **Do not use this port or you may experience errors.**

**ADD NODES TO SWARM**

As you can see, the response to `docker swarm init` contains a pre-configured `docker swarm join` command for you to run on any nodes you want to add.
Copy this command, and send it to `myvm2` via `docker-machine ssh` to have **myvm2** join your new swarm as a **worker**.

```
docker-machine ssh myvm2 "docker swarm join \
--token SWMTKN-1-50a1ogovr3uy9wj0uyk1y38t78acqstib8bry81t1zgzayu9ge-dagwgu5hsqm1lidl1td67qc40 \
192.168.99.100:2377"

> This node joined a swarm as a worker.
```

Run `docker node ls` on the manager to view the nodes in this swarm.

```
docker-machine ssh myvm1 "docker node ls"

> ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
> zsx6r0dx9ew8nx2ejjkpq80hk *   myvm1               Ready               Active              Leader
> jxml5ddka5hjqxasa20ned5ww     myvm2               Ready               Active              
```

**Leaving a swarm**

If you want to start over, you can run `docker swarm leave` from each node.

![vb-swarm](https://raw.githubusercontent.com/quanpan302/test/master/t00-Docker/t03/vb-swarm.png)

## Deploy your app on the swarm cluster

The hard part is over.

Now you just repeat the process you used in part 3 to deploy on your new swarm.

Just remember that only swarm managers like myvm1 execute Docker commands; workers are just for capacity.

### Configure a `docker-machine` shell to the swarm manager

So far, you’ve been wrapping Docker commands in `docker-machine ssh` to talk to the VMs.

Another option is to run `docker-machine env <machine>` to get and run a command that configures your current shell to talk to the Docker daemon on the VM.

This method works better for the next step because it allows you to use your local `docker-compose.yml` file to deploy the app “remotely” without having to copy it anywhere.

1. Type `docker-machine env myvm1`
2. copy-paste and run the command provided as the last line of the output
3. configure your shell to talk to **myvm1**, the **swarm manager**.

**DOCKER MACHINE SHELL ENVIRONMENT ON MAC OR LINUX**

Run `docker-machine env myvm1` to get the command to configure your shell to talk to `myvm1`.

```
docker-machine env myvm1

> export DOCKER_TLS_VERIFY="1"
> export DOCKER_HOST="tcp://192.168.99.100:2376"
> export DOCKER_CERT_PATH="/Users/quanpan/.docker/machine/machines/myvm1"
> export DOCKER_MACHINE_NAME="myvm1"
> # Run this command to configure your shell: 
> # eval $(docker-machine env myvm1)
```

Run the given command to configure your shell to talk to `myvm1`.

```
eval $(docker-machine env myvm1)

> 
```

Run `docker-machine ls` to verify that `myvm1` is now the active machine, as indicated by the _asterisk_ next to it.

```
docker-machine ls

> NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
> myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v18.02.0-ce   
> myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.02.0-ce   
```

### Deploy the app on the swarm manager

Now that you have myvm1,
you can use its powers as a swarm manager to deploy your app by using the same `docker stack deploy` command you used in part 3 to **myvm1**,
and your local copy of `docker-compose.yml.`. 

Just like before, run the following command to deploy the app on **myvm1**.

```
docker stack deploy -c docker-compose.yml getstartedlab

> Creating network getstartedlab_webnet
> Creating service getstartedlab_web
```

> And that’s it, the app is deployed on a swarm cluster!

Now you can use the same [docker commands you used in part 3](https://docs.docker.com/get-started/part3/#run-your-new-load-balanced-app).
Only this time notice that the services (and associated containers) **have been distributed between both** `myvm1` and `myvm2`.

```
docker stack ps getstartedlab

> ID                  NAME                  IMAGE                 NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
> us63h7l9826o        getstartedlab_web.1   quanpan302/test:t01   myvm2               Running             Running 8 minutes ago                       
> yokqfgcl8sma        getstartedlab_web.2   quanpan302/test:t01   myvm2               Running             Running 8 minutes ago                       
> rsdpfitjyx1o        getstartedlab_web.3   quanpan302/test:t01   myvm1               Running             Running 8 minutes ago                       
> mkyfbev6bya2        getstartedlab_web.4   quanpan302/test:t01   myvm2               Running             Running 8 minutes ago                       
> m2vr7sk0fejw        getstartedlab_web.5   quanpan302/test:t01   myvm1               Running             Running 8 minutes ago                       
```

> Connecting to VMs with `docker-machine env` and `docker-machine ssh`
> 
> * To set your shell to talk to a different machine like myvm2, simply re-run docker-machine env in the same or a different shell, then run the given command to point to myvm2. This is always specific to the current shell. If you change to an unconfigured shell or open a new one, you need to re-run the commands. Use docker-machine ls to list machines, see what state they are in, get IP addresses, and find out which one, if any, you are connected to. To learn more, see the Docker Machine getting started topics.
> * Alternatively, you can wrap Docker commands in the form of docker-machine ssh <machine> "<command>", which logs directly into the VM but doesn’t give you immediate access to files on your local host.
> * On Mac and Linux, you can use docker-machine scp <file> <machine>:~ to copy files across machines, but Windows users need a Linux terminal emulator like Git Bash for this to work.
> 
> This tutorial demos both docker-machine ssh and docker-machine env, since these are available on all platforms via the `docker-machine` CLI.

### Accessing your cluster

You can access your app from the `IP address` of **either** `myvm1` or `myvm2`.

```
docker-machine ls

> myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v18.02.0-ce   
> myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.02.0-ce   
```

![ingress-routing-mesh](https://raw.githubusercontent.com/quanpan302/test/master/t00-Docker/t03/ingress-routing-mesh.png)

### Cleanup and reboot

```
docker stack ps getstartedlab

> ID                  NAME                  IMAGE                 NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
> us63h7l9826o        getstartedlab_web.1   quanpan302/test:t01   myvm2               Running             Running 20 minutes ago                       
> yokqfgcl8sma        getstartedlab_web.2   quanpan302/test:t01   myvm2               Running             Running 20 minutes ago                       
> rsdpfitjyx1o        getstartedlab_web.3   quanpan302/test:t01   myvm1               Running             Running 20 minutes ago                       
> mkyfbev6bya2        getstartedlab_web.4   quanpan302/test:t01   myvm2               Running             Running 20 minutes ago                       
> m2vr7sk0fejw        getstartedlab_web.5   quanpan302/test:t01   myvm1               Running             Running 20 minutes ago                       
```

**Stacks and swarms**

```
docker stack rm getstartedlab

> Removing service getstartedlab_web
> Removing network getstartedlab_webnet
```

> **Keep the swarm or remove it?**
> 
> * `docker-machine ssh myvm2 "docker swarm leave"` on the **worker**
> * `docker-machine ssh myvm1 "docker swarm leave --force"` on the **manager**
> 

```
docker-machine ssh myvm2 "docker swarm leave"
docker-machine ssh myvm1 "docker swarm leave --force"
docker-machine ls

> NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
> myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v18.02.0-ce   
> myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.02.0-ce   
```

**Unsetting docker-machine shell variable settings**

```
eval $(docker-machine env -u)

docker-machine env -u

> unset DOCKER_TLS_VERIFY
> unset DOCKER_HOST
> unset DOCKER_CERT_PATH
> unset DOCKER_MACHINE_NAME
> # Run this command to configure your shell: 
> # eval $(docker-machine env -u)
```

**Stop Docker machines**

```
docker-machine stop myvm1

> Stopping "myvm1"...
> Machine "myvm1" was stopped.
```

```
docker-machine stop myvm2

> Stopping "myvm2"...
> Machine "myvm2" was stopped.
```

```
docker-machine ls

> NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
> myvm1   -        virtualbox   Stopped                 Unknown   
> myvm2   -        virtualbox   Stopped                 Unknown   
```

**Restarting Docker machines**


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
docker stack deploy -c <file> <app>                                                       # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker stack deploy -c docker-compose.yml getstartedlab                                   # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker stack ps <app>                                                                     # List
docker stack rm <app>                                                                     # Remove
docker-machine scp docker-compose.yml myvm1:~                                             # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"                            # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
```