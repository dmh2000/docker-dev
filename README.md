# Build the base image

One time build of the base image. Takes a while, has to download Ubuntu

## Setup

Do the following in a clean new directory

- cd new-directory
- add the Dockerfile to the directory
- run the build, export and compress

### Dockerfile

NOTE: tag names such as username/ubuntu-base are arbitrary

```yaml
FROM ubuntu:20.04

# working directory
WORKDIR /app

# get lastest repositories
RUN apt-get update -y

# install c++ support
RUN apt-get install -y build-essential g++

# verify g++ is installed
CMD ["g++","--help"]
```

## build,export and compress

This produces the docker image with Ubuntu and g++ and then saves it to a tar
file. For best results go ahead and compress the tar vile

```bash
# build the base
docker build -t username/ubuntu-gpp

# export it to a tar file and save it somewhere
docker save -o gpp-base.tar username/ubuntu-gpp

# zip it up (output is gpp-base.tar.gz)
gzip gpp-base.tar

# store it somewhere with versioning
```

# Restore the compressed image

If you have removed the base image and later you want to restore it, or distribute
it to another system, so the following.
If you didn't 'rm' the base image you can skip this step.

## import the compressed image

```bash

# import it
docker load -i gpp-base.tar.gz

# check that its there
docker images -a

# should show something like this
# REPOSITORY           TAG       IMAGE ID       CREATED       SIZE
# username/ubuntu-gpp   latest    ebf49abca245   3 hours ago   344MB
```

# Build the dev image

## dockerfile

```yaml
FROM username/ubuntu-base

# working directory
WORKDIR /app

# copy the build script to /app
COPY build-dev.sh .

# make it executable in the container
RUN chmod +x build-dev.sh

# create the output directory
RUN mkdir output

# run the build script (only happens when running the container)
CMD ["./build-dev.sh"]
```

## build script that is stored in the image

```bash
#!/bin/sh
pwd
# for debug
ls -l output
# build the application from the output directory
# which is a bind-mount when run. this directory
# will be mapped to the host directory
g++ -o output/hello output/hello.cpp
# confirm
ls -l output
exit
```

```bash
cd build
# confirm that the base image is available
docker images -a

# build the dev image
docker build -t username/build-dev
```

# Run the dev image to build the application

CD into your development project directory

```bash
cd build
# confirm the
# create the output directory to be mounted to the container
mkdir output

# copy your application source to the output directory
# and whatever is needed such as Makefile, cmake files et
cp hello.cpp output

# run the build-dev image
# -ti makes it interactive
# --rm removes the container after the run
# -v binds the local directory to the container directory
docker run -ti --rm -v $PWD/output:/app/output username/build-dev

# confirm the build worked
ls -l output
output/hello
```

# Cleanup commands

## Clean all containers

docker ps -aq | xargs docker rm

## Clean all images (carefule, this will remove all of your images)

docker images -aq | xargs docker rmi
