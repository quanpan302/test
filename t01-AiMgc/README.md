t01-AiMgc


Docker:
=======

$ docker build -t gsl docker
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


$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
gsl latest b3f3b5f49e4a 24 hours ago 1.52 GB
...
