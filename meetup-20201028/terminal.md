
# Basics

All running containers

    $ docker ps 


The same command (but 'new' approach)

    $ docker container ls 
 
 
### The power of autocomplete - try it out! 

Alternatively use the "--help" option instead.

    $ docker ps <TAB><TAB> 
    $ docker container <TAB><TAB> 
 

### some more or less useful information

Notice that no matter which OS you are using - OS type always will be "Linux", e.g:

    $ docker info 
    ...
    Kernel Version: 4.19.121-linuxkit
    Operating System: Docker Desktop
    OSType: linux
    ...


# Running containers 
 
    $ docker run --rm -ti debian bash 
    $ docker run -ti debian bash 
    $ docker run -ti --name NAME debian bash 
    
### removing container
 
    $ docker ps -a | grep NAME  
    $ docker rm NAME
     
 
# Stateless 
 
### test 1

    $ docker run --rm -ti debian bash 
    (container)# ps 
    (error - no such file) 
    (container)# apt update && apt install procps 
    (container)# ps 
    (container)# exit 
     
And once again - notice that `ps` is not there anymore:

    $ docker run --rm -ti debian bash 
    (container)# ps 

### test 2
  
    $ docker run --name container01 -ti debian bash 
    (container01)# ps 
    (container01)# apt update && apt install procps 
    (container01)# exit 
      
    $ docker run --name container01 -ti debian bash 
    (error - such a container already exists!) 

To start (again) stopped container use:
  
    $ docker start container01
    (container01)# ps 
     
 
# Overlay fs 
 
    $ sudo mount -t overlay overlay -o  lowerdir=lower,upperdir=diff,workdir=work merged 
 
