# Get Started, Part 2

[Containers](http://devdocs.io/docker~17/get-started/part2/index)

## Define a container with Dockerfile

**Dockerfile**

Dockerfile defines what goes on in the environment inside your container.
Access to resources like networking interfaces and disk drives is virtualized inside this environment,

* which is isolated from the rest of your system,
* so you need to map ports to the outside world,
* and be specific about what files you want to “copy in” to that environment.

However, after doing that, you can expect that the build of your app defined in this Dockerfile behaves exactly the same wherever it runs.

```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

## Build the app

We are ready to build the app. Make sure you are still at the top level of your new directory. Here’s what `ls` should show:

Now run the build command. This creates a Docker image, which we’re going to tag using `-t` so it has a friendly name.

```
docker build -t friendlyhello .
```

Where is your built image? It’s in your machine’s local Docker image registry:

```
docker images

> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
> friendlyhello       latest              e9c8db7940a0        44 seconds ago      149MB
> python              2.7-slim            52ad41c7aea4        6 days ago          139MB
```

## Run the app

**interactive mode**

Run the app, mapping your machine’s port 4000 to the container’s published port 80 using `-p`:

```
docker run -p 4000:80 friendlyhello
```

You should see a notice that Python is serving your app at `http://0.0.0.0:80`.
But that message is coming from inside the container, which doesn’t know you mapped port 80 of that container to 4000,
making the correct URL `http://localhost:4000`.

Go to that URL `http://localhost:4000` in a web browser to see the display content served up on a web page,
including “Hello World” text, the container ID, and the **Redis error message**.


You can also use the curl command in a shell to view the same content.

```
curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> e9c8db7940a0<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

This port remapping of `4000:80` is to demonstrate the difference between what you `EXPOSE` within the Dockerfile, and what you publish using `docker run -p`.
In later steps, we just map port `80` on the host to port `80` in the container and use `http://localhost`.

*Close and web browser. Hit `CTRL+C` in your terminal to quit.*

**detached mode**

Now let’s run the app in the background, in detached mode:

```
docker run -d -p 4000:80 friendlyhello
```

You get the long container ID for your app and then are kicked back to your terminal.
Your container is running in the background.
You can also see the abbreviated container ID with docker `container ls` (and both work interchangeably when running commands):

```
docker container ls

> CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                  NAMES
> ffa318fd7e18        friendlyhello       "python app.py"     About a minute ago   Up About a minute   0.0.0.0:4000->80/tcp   gracious_mayer
```

You’ll see that `CONTAINER ID` matches what’s on `http://localhost:4000`.

Now use `docker stop` to end the process, using the `CONTAINER ID`, like so:

*Close and web browser.*
```
docker stop ffa318fd7e18
```

## Share your image

Log in with your Docker ID

```
docker login
```

Tag the image

```
docker tag image username/repository:tag
```

For example:

```
docker tag friendlyhello quanpan302/test:t01
```

```
$ docker image ls

> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
> friendlyhello       latest              e9c8db7940a0        3 minutes ago       149MB
> quanpan302/test     t01                 e9c8db7940a0        3 minutes ago       149MB
> python              2.7-slim            52ad41c7aea4        6 days ago          139MB
```

**Untagged**: quanpan302/test:t01

```
$ sudo docker rmi quanpan302/test:t01
```

## Publish the image

Upload your tagged image to the repository:

```
docker push username/repository:tag

docker push quanpan302/test:t01
```

## Recap and cheat sheet (optional)

_Here are some commands you might like to run to interact with your swarm and your VMs a bit_

```
docker build -t friendlyhello .                    # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello                # Run "friendlyhello" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello             # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a                             # List all containers, even those not running
docker container stop <hash>                       # Gracefully stop the specified container
docker container kill <hash>                       # Force shutdown of the specified container
docker container rm <hash>                         # Remove specified container from this machine
docker container rm $(docker container ls -a -q)   # Remove all containers
docker image ls -a                                 # List all images on this machine
docker image rm <image id>                         # Remove specified image from this machine
docker image rm $(docker image ls -a -q)           # Remove all images from this machine
docker login                                       # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag         # Tag <image> for upload to registry
docker push username/repository:tag                # Upload tagged image to registry
docker run username/repository:tag                 # Run image from a registry
```
