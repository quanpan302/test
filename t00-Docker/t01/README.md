# Get Started, Part 2: [Containers](http://devdocs.io/docker~17/get-started/part2/index)

## Build the app

We are ready to build the app. Make sure you are still at the top level of your new directory. Here’s what `ls` should show:

Now run the build command. This creates a Docker image, which we’re going to tag using `-t` so it has a friendly name.

```
docker build -t t01 .
```

Where is your built image? It’s in your machine’s local Docker image registry:

```
$ docker images

REPOSITORY            TAG                 IMAGE ID
t01                   latest              326387cea398
```

## Run the app

Run the app, mapping your machine’s port 4000 to the container’s published port 80 using `-p`:

```
docker run -p 4000:80 t01
```

You should see a notice that Python is serving your app at `http://0.0.0.0:80`.
But that message is coming from inside the container, which doesn’t know you mapped port 80 of that container to 4000,
making the correct URL `http://localhost:4000`.

Go to that URL in a web browser to see the display content served up on a web page,
including “Hello World” text, the container ID, and the **Redis error message**.

Hit `CTRL+C` in your terminal to quit.

Now let’s run the app in the background, in *detached mode*:

```
docker run -d -p 4000:80 t01
```

You get the long container ID for your app and then are kicked back to your terminal.
Your container is running in the background.
You can also see the abbreviated container ID with docker `container ls` (and both work interchangeably when running commands):

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        t01                 "python app.py"     28 seconds ago
```

You’ll see that `CONTAINER ID` matches what’s on `http://localhost:4000`.

Now use `docker stop` to end the process, using the `CONTAINER ID`, like so:

```
docker stop 1fa4ab2cf395
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
docker tag t01 quanpan302/testdocker:t01
```

```
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
t01                      latest              d9e555c53008        3 minutes ago       195MB
quanpan302/testdocker    part1               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

## Publish the image

Upload your tagged image to the repository:

```
docker push username/repository:tag
```

## Recap and cheat sheet (optional)

Here is a list of the basic Docker commands from this page, and some related ones if you’d like to explore a bit before moving on.

```
docker build -t t01 .           # Create image using this directory's Dockerfile
docker run -p 4000:80 t01       # Run "t01" mapping port 4000 to 80
docker run -d -p 4000:80 t01    # Same thing, but in detached mode
docker container ls             # List all running containers
docker container ls -a          # List all containers, even those not running
docker container stop <hash>    # Gracefully stop the specified container
docker container kill <hash>    # Force shutdown of the specified container
docker container rm <hash>      # Remove specified container from this machine
docker container rm $(docker container ls -a -q)   # Remove all containers
docker image ls -a                                 # List all images on this machine
docker image rm <image id>                         # Remove specified image from this machine
docker image rm $(docker image ls -a -q)           # Remove all images from this machine
docker login                                       # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag         # Tag <image> for upload to registry
docker push username/repository:tag                # Upload tagged image to registry
docker run username/repository:tag                 # Run image from a registry
```
