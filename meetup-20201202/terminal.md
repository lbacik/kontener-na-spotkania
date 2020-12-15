
# Imperative way 

### step 1
Create container and install some additional, example software:

    $ docker run --name container01 -ti debian bash
    (container01)# ps
    (container01)# apt update
    (container01)# apt install procps
    (container01)# exit


### step 2
Run it again:

    $ docker run --name container01 -ti debian bash
    
    !!! ERROR

### step 3
You have to use `start`:

    $ docker start -ai container01
    # ps

(`ps` should be already there - we installed it in _step 1_ )


### step 4

    $ docker container diff container01
    $ docker container commit container01 debian:local


    $ docker rm container01

    $ docker run --rm -ti debian:local bash


# Declarative way (Dockerfile)

    --- Dockerfile 
    FROM debian
    RUN apt-get -y update && apt-get -y install \
        procps


    $ docker build -t debian:image01 .


    $ docker image ls

    $ docker run --rm -ti debian:image01 bash 
    (container)# ps


# Liner

Hadolint: https://github.com/hadolint/hadolint

    $ hadolint test/Dockerfile
    ...


    --- Dockerfile (meetup-20201202/test)
    FROM debian:buster

    # hadolint ignore=DL3008
    RUN apt-get update && apt-get -y install --no-install-recommends \
        procps \
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*
    

# Sharing configuration/images

### Dockerfile in repositories

Probably the easiest way to share the docker image configuration - required the image to be built on the 
client-side.

    $ git push ... ;)


### docker image repository

It is very easy to publish the already built imgaes through the pubic repositories like the dockerhub.com 

    $ docker tag debian:image01 lbacik/test-image01
    $ docker push lbacik/test-image01


https://ropenscilabs.github.io/r-docker-tutorial/04-Dockerhub.html

