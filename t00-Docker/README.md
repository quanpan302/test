# test Docker

All Docker tests, tutorials and examples

:link:[docs.docker.com/get-started](https://docs.docker.com/get-started/part2/#build-the-app)

**Table of Contents**

- [Docker Overview](#docker-overview)
  - [Docker Engine](#docker-engine)
  - [Docker Architecture](#docker-architecture)
- [The Underlying Technology](#the-underlying-technology)
  - [Namespaces](#namespaces)
- [Dockerfile Reference](#dockerfile-reference-link)
- [Compose file version 3 reference](#compose-file-version-3-reference-link)
  - [Service configuration reference](#service-configuration-reference-link)
  - [Specifying durations](#specifying-durations-link)
  - [Specifying byte values](#specifying-byte-values-link)
  - [Volume configuration reference](#volume-configuration-reference-link)
  - [Network configuration reference](#network-configuration-reference-link)
  - [configs configuration reference](#configs-configuration-reference-link)
  - [secrets configuration reference](#secrets-configuration-reference-link)
  - [Variable substitution](#variable-substitution-link)
  - [Extension fields](#extension-fields-link)

# Docker Overview

## Docker Engine

Docker Engine is a client-server application with these major components:

* A server which is a type of long-running program called a daemon process (the dockerd command).
* A REST API which specifies interfaces that programs can use to talk to the daemon and instruct it what to do.
* A command line interface (CLI) client (the docker command).

![engine-components-flow](https://raw.githubusercontent.com/quanpan302/test/master/t00-Docker/engine-components-flow.png)

## Docker Architecture

Docker uses a client-server architecture.

The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers.

The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon.

The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface.

<img src="https://raw.githubusercontent.com/quanpan302/test/master/t00-Docker/architecture.svg?sanitize=true">

# The Underlying Technology

Docker is written in `Go` and takes advantage of several features of the Linux kernel to deliver its functionality.

## Namespaces

Docker uses a technology called namespaces to provide the isolated workspace called the container.
When you run a container, Docker creates a set of namespaces for that container.

Docker Engine uses namespaces such as the following on Linux:

* **The `pid` namespace**: Process isolation (PID: Process ID).
* **The `net` namespace**: Managing network interfaces (NET: Networking).
* **The `ipc` namespace**: Managing access to IPC resources (IPC: InterProcess Communication).
* **The `mnt` namespace**: Managing filesystem mount points (MNT: Mount).
* **The `uts` namespace**: Isolating kernel and version identifiers. (UTS: Unix Timesharing System).

# Dockerfile Reference [link](https://docs.docker.com/engine/reference/builder/)

These recommendations help you to write an efficient and maintainable `Dockerfile`.

1. **FROM** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#from)
   
   `FROM <image> [AS <name>]`
   
   `FROM <image>[:<tag>] [AS <name>]`
   
   `FROM <image>[@<digest>] [AS <name>]`
   
2. **RUN** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#run)
   
   `RUN <command>` _(shell form, the command is run in a shell, which by default is `/bin/sh -c` on Linux_ or _`cmd /S /C` on Windows)_
   
   `RUN ["executable", "param1", "param2"]` _(exec form)_
   
3. **CMD** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#cmd)
   
   `CMD ["executable","param1","param2"]` _(exec form, this is the preferred form)_
   
   `CMD ["param1","param2"]` _(as default parameters to ENTRYPOINT)_
   
   `CMD command param1 param2` _(shell form)_
   
4. **LABEL** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#label)
   
   `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
   
   ```
   LABEL "com.example.vendor"="ACME Incorporated"
   LABEL com.example.label-with-value="foo"
   LABEL version="1.0"
   LABEL description="This text illustrates \
   that label-values can span multiple lines."
   ```
   
5. **EXPOSE** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#expose)
   
   `EXPOSE <port> [<port>/<protocol>...]`
   
6. **ENV** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#env)
   
   ```
   ENV <key> <value>
   ENV <key>=<value> ...
   ```
   
7. **ADD** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#add)
   
   * `ADD [--chown=<user>:<group>] <src>... <dest>`
   * `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]` _(this form is required for paths containing whitespace)_
   
8. **COPY** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#copy)
   
   * `COPY [--chown=<user>:<group>] <src>... <dest>`
   * `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]` _(this form is required for paths containing whitespace)_
   
9. **ENTRYPOINT** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#entrypoint)
   
   * `ENTRYPOINT ["executable", "param1", "param2"]` _(exec form, preferred)_
   * `ENTRYPOINT command param1 param2` _(shell form)_
   
   |                              | No ENTRYPOINT                | ENTRYPOINT exec\_entry p1\_entry | ENTRYPOINT [“exec\_entry”, “p1\_entry”]            |
   |------------------------------|------------------------------|----------------------------------|----------------------------------------------------|
   | No CMD                       | error, not allowed           | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry                              |
   | CMD [“exec\_cmd”, “p1\_cmd”] | exec\_cmd p1\_cmd            | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry exec\_cmd p1\_cmd            |
   | CMD [“p1\_cmd”, “p2\_cmd”]   | p1\_cmd p2\_cmd              | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry p1\_cmd p2\_cmd              |
   | CMD exec\_cmd p1\_cmd        | /bin/sh -c exec\_cmd p1\_cmd | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry /bin/sh -c exec\_cmd p1\_cmd |
   
10. **VOLUME** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#volume)
   
   `VOLUME ["/data"]`
   
   For more information/examples and mounting instructions via the Docker client,
   refer to [Share Directories via Volumes](https://docs.docker.com/storage/volumes/) documentation.
   
11. **USER** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#user)
   
   ```
   USER <user>[:<group>]
   USER <UID>[:<GID>]
   ```
   
12. **WORKDIR** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#workdir)
   
   `WORKDIR /path/to/workdir`
   
   The `WORKDIR` instruction can resolve environment variables previously set using `ENV`.
   You can only use environment variables explicitly set in the `Dockerfile`. 
   
   ```
   ENV DIRPATH /path
   WORKDIR $DIRPATH/$DIRNAME
   RUN pwd
   ```
   
   The output of the final `pwd` command in this `Dockerfile` would be `/path/$DIRNAME`
   
13. **ARG** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#arg)
   
   `ARG <name>[=<default value>]`
   
   **Using ARG variables**
   
   _constant\_image\_version_
   
   ```
   FROM ubuntu
   ARG CONT_IMG_VER
   ENV CONT_IMG_VER ${CONT_IMG_VER:-v1.0.0}
   RUN echo $CONT_IMG_VER
   ```
   
   **--build-arg \<varname\>=\<value\>**
   
   `docker build --build-arg CONT_IMG_VER=v2.0.1 .`
   
   **Predefined ARGs**
   
   * `HTTP_PROXY`
   * `http_proxy`
   * `HTTPS_PROXY`
   * `https_proxy`
   * `FTP_PROXY`
   * `ftp_proxy`
   * `NO_PROXY`
   * `no_proxy`
   
14. **ONBUILD** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#onbuild)
   
   `ONBUILD [INSTRUCTION]`
   
15. **STOPSIGNAL** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#stopsignal)
   
   `STOPSIGNAL signal`
   
16. **HEALTHCHECK** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#healthcheck)
   
   * `HEALTHCHECK [OPTIONS] CMD command` _(check container health by running a command inside the container)_
   * `HEALTHCHECK NONE` _(disable any healthcheck inherited from the base image)_
   
   The options that can appear before CMD are:
   
   * `--interval=DURATION` (default: `30s`)
   * `--timeout=DURATION` (default: `30s`)
   * `--start-period=DURATION` (default: `0s`)
   * `--retries=N` (default: `3`)
   
   ```
   HEALTHCHECK --interval=5m --timeout=3s \
     CMD curl -f http://localhost/ || exit 1
   ```
     
17. **SHELL** [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#shell)
   
   `SHELL ["executable", "parameters"]`
   
   The default shell
   
   * on Linux is `["/bin/sh", "-c"]`
   * on Windows is `["cmd", "/S", "/C"]`

# Compose file version 3 reference [link](https://docs.docker.com/compose/compose-file/)

| Compose file format | Docker Engine release |
|---------------------|-----------------------|
| 3.6                 | 18.02.0+              |
| 3.5                 | 17.12.0+              |
| 3.4                 | 17.09.0+              |
| 3.3                 | 17.06.0+              |
| 3.2                 | 17.04.0+              |
| 3.1                 | 1.13.1+               |
| 3.0                 | 1.13.0+               |
| 2.3                 | 17.06.0+              |
| 2.2                 | 1.13.0+               |
| 2.1                 | 1.12.0+               |
| 2.0                 | 1.10.0+               |
| 1.0                 | 1.9.1.+               |

## Service configuration reference [link](https://docs.docker.com/compose/compose-file/#service-configuration-reference)

## Specifying durations [link](https://docs.docker.com/compose/compose-file/#specifying-durations)

## Specifying byte values [link](https://docs.docker.com/compose/compose-file/#specifying-byte-values)

## Volume configuration reference [link](https://docs.docker.com/compose/compose-file/#volume-configuration-reference)

## Network configuration reference [link](https://docs.docker.com/compose/compose-file/#network-configuration-reference)

## configs configuration reference [link](https://docs.docker.com/compose/compose-file/#configs-configuration-reference)

## secrets configuration reference [link](https://docs.docker.com/compose/compose-file/#secrets-configuration-reference)

## Variable substitution [link](https://docs.docker.com/compose/compose-file/#variable-substitution)

## Extension fields [link](https://docs.docker.com/compose/compose-file/#extension-fields)
