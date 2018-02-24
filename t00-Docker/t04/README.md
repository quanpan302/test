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

