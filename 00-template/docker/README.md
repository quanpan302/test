# Docker:

Management Commands:

```
  checkpoint  Manage checkpoints
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  volume      Manage volumes
```

Commands:

```
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  deploy      Deploy a new stack or update an existing stack
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes
```


## $ docker build -t gsl docker

```
Sending build context to Docker daemon 2.048 kB
Step 1/3 : FROM gcc
latest: Pulling from library/gcc
693502eb7dfb: Already exists
...
Status: Downloaded newer image for gcc:latest
---> 408d218617ca
Step 2/3 : MAINTAINER @joshuacook
---> Running in 43a89bcd4fae
---> 97faa5ea6f1e
Removing intermediate container 43a89bcd4fae
Step 3/3 : RUN apt-get update && apt-get install -y gsl-bin
libgsl0-dbg libgsl0-dev libgsl0ldbl
---> Running in 988614cb6d56
...
---> 568814736d4b
Removing intermediate container 988614cb6d56
Successfully built 568814736d4b
```

# Dockerfile

## gsl

```
FROM gcc

LABEL maintainer=@quanpan

RUN apt-get update && \
  apt-get install -y \
  gsl-bin \
  libgsl0-dbg \
  libgsl0-dev \
  libgsl2
```

## miniconda3

```
FROM debian

LABEL maintainer=@quanpan

ARG DEBIAN_FRONTEND=noninteractive

RUN /bin/bash -c 'source $HOME/.bashrc ; echo $HOME'

RUN apt-get update --fix-missing && \
  apt-get install -y \
  wget bzip2 ca-certificates \
  libglib2.0-0 libxext6 libsm6 libxrender1

RUN echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh && \
  wget --quiet \
  https://repo.continuum.io/miniconda/Miniconda3-4.3.11-Linux-x86_64.sh -O
  ~/miniconda.sh && \
  /bin/bash ~/miniconda.sh -b -p /opt/conda && \
  rm ~/miniconda.sh
  
RUN apt-get install -y curl grep sed dpkg && \
  TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest |
  grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
  curl -L "https://github.com/krallin/tini/releases/download/v${TINI_
  VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
  dpkg -i tini.deb && \
  rm tini.deb && \
  apt-get clean

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV PATH /opt/conda/bin:$PATH
```

## ipython

```
FROM miniconda3

LABEL maintainer=@quanpan

RUN conda update conda && \
  conda install --quiet --yes ipython && \
  conda clean -tipsy

CMD ["ipython"]
```
