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

### Configure a docker-machine shell to the swarm manager

