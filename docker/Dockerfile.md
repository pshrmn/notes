# Dockerfile

Defines the container environment.

## Commands

https://docs.docker.com/engine/reference/builder/

### FROM

https://docs.docker.com/engine/reference/builder/#from

A base image

> a valid Dockerfile must start with a `FROM` instruction

```bash
FROM ubuntu
```

### USER

https://docs.docker.com/engine/reference/builder/#user

Set the user (and possibly the group)

```bash
USER username
USER username:group
USER UID
USER UID:GID
```

### WORKDIR

https://docs.docker.com/engine/reference/builder/#workdir

Sets the working directory. This will be used by any subsequent `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD` commands.

Multiple `WORKDIR`s can be specified; relative `WORKDIR`s will be on the previous `WORKDIR` path.

```bash
WORKDIR /test
WORKDIT ing # /test/ing
```

### ENV

https://docs.docker.com/engine/reference/builder/#env

Set an environment variable

```bash
ENV key value # one variable
ENV key=value another=value # multiple variables, requires equals sign
```

### ADD

https://docs.docker.com/engine/reference/builder/#add

Copy files, directories, and URLs from source to destination.

If the source is a recognized archive format, `ADD` will unpack it.

```bash
ADD source destination # UID & GID = 0
ADD --chown=UID:GID source destination
ADD --chown=UID source destination # GID = UID
```

Source interpreted relative to build's source context.

Destination is either absolute or relative to the `WORKDIR`

If `--chown` is not provided, files/directories created with UID & GID of 0.

### COPY

https://docs.docker.com/engine/reference/builder/#copy

Copy files/directories from source to destination. `COPY` is like `ADD`, but doesn't support URLs and does not unpack archives.

```bash
COPY source destination
COPY ["source with spaces", "destination location"]
COPY --chown=user:group source destination
```

### CMD

https://docs.docker.com/engine/reference/builder/#cmd

Run a command, primarily used for provide defaults for executing container.

Only one `CMD` is allowed per Dockerfile. If there are multiple, the last one will be used.

```bash
CMD ["executable", "param", "another"]
CMD command param another
```

### RUN

https://docs.docker.com/engine/reference/builder/#run

Run a command and create a new image. Use case: install a package on top of the OS.

The command can either be provided in shell form or exec form.

```bash
RUN command # shell form
RUN ["sequence", "of", "values"] # exec form
```

### ARG

https://docs.docker.com/engine/reference/builder/#arg

A build-time variable, with optional default value.

```bash
ARG name
ARG other=default
```

Docker comes with some [built-in args](https://docs.docker.com/engine/reference/builder/#predefined-args).

### LABEL

https://docs.docker.com/engine/reference/builder/#label

Used for adding metadata to an image with key value pairs.

```bash
LABEL one="two"
LABEL three="four" five="six"
LABEL seven="eight" \
      nine="ten" \
      "eleven twelve"="thirteen"
```

An image's labels can be viewed using `docker inspect`.

### EXPOSE

https://docs.docker.com/engine/reference/builder/#expose

Tell docker to listen to specified port. TCP or UDP (TCP is default if not specified)

```bash
EXPOSE 80/tcp
EXPOSE 123 # tcp
EXPOSE 456/udp
```

These can be overriden at runtime with the `-p` flag.

```bash
docker run -p 80:80/tcp
```

### ENTRYPOINT

https://docs.docker.com/engine/reference/builder/#entrypoint

> configure a container that will run as an executable

```bash
ENTRYPOINT ["executable", "one", "two"] # exec form
ENTRYPOINT command one two # shell form
```

### VOLUME

https://docs.docker.com/engine/reference/builder/#volume

Create a mount point