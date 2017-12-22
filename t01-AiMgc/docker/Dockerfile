FROM gcc

LABEL maintainer=@quanpan

RUN apt-get update && \
  apt-get install -y \
  gsl-bin \
  libgsl0-dbg \
  libgsl0-dev \
  libgsl2
  
RUN gcc \
  -I /usr/include/ \
  -L /usr/lib/ -lgsl -lgslcblas \
  /home/src/bessel.c
  -o /home/build/bessel
  
