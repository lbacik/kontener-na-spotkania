# Networking and netchat app

Netchat - the simple (and well know :)) example of an internet chat app build with netcat.

## build the netchat app image

	$ cd netchat
	$ docker build -t netchat:local .

## test 1 - server and client in the same container

### "Server"

	(terminal 1) $ docker run -ti --name netchat --rm netchat:local
	listening on [any] 4444 ...

### "Client"

	(terminal 2) $ docker exec -ti netchat nc localhost 4444

Now you should be able typing in both terminals. To send a message just press ENTER :)

## test 2 - two containers within the same docker network

### "Server"

	(terminal 1) $ docker run -ti --name netchat --rm netchat:local
	listening on [any] 4444 ...

### "Client"

To find server's ip address:

	$ docker inspect netchat | grep IPAddress
		"SecondaryIPAddresses": null,
	            "IPAddress": "172.17.0.2",
	                    "IPAddress": "172.17.0.2",

And now you can connect to the server (from different container) by:

	$ docker run --rm -ti netchat:local nc 172.17.0.2 4444

Check it by:

	➜  ~ docker ps
	CONTAINER ID   IMAGE           COMMAND                CREATED         STATUS         PORTS      NAMES
	9c5c552fd0b4   netchat:local   "nc 172.17.0.2 4444"   3 minutes ago   Up 3 minutes   4444/tcp   crazy_kare
	d727ae4f20e9   netchat:local   "nc -lv -p 4444"       5 minutes ago   Up 5 minutes   4444/tcp   netchat

### bridge (system) network

The problem: it is not possible to connect to the "server" using its (container) name:

	$ docker run --rm -ti netchat:local nc netchat 4444
	netchat: forward host lookup failed: Unknown host

### user-defined bridge networks

https://docs.docker.com/network/network-tutorial-standalone/

	$ docker network create test
	$ docker network ls
	NETWORK ID     NAME          DRIVER    SCOPE
	11cea57441b1   bridge        bridge    local
	d953159d763f   host          host      local
	e7c46a5e6d27   none          null      local
	a4d6e6293611   test          bridge    local

**"Server"**

	$ docker run -ti --name netchat --rm --net test netchat:local
	listening on [any] 4444 ...

Notice the "--net" argument!

**"Client"**

And the connection-by-name should works now:

	$ docker run --rm -ti --net test netchat:local nc netchat 4444

#### comparison of bridge networks

**bridge (system)**

	$ docker run --rm netchat:local ss -ln
	Netid   State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port
	nl      UNCONN   0        0                      0:730                  *
	nl      UNCONN   0        0                      0:0                    *
	nl      UNCONN   4352     0                      4:1                    *
	nl      UNCONN   768      0                      4:0                    *
	nl      UNCONN   0        0                      6:0                    *
	nl      UNCONN   0        0                      9:0                    *
	nl      UNCONN   0        0                     10:0                    *
	nl      UNCONN   0        0                     12:0                    *
	nl      UNCONN   0        0                     15:0                    *
	nl      UNCONN   0        0                     16:0                    *

**user-defined bridge network**

	$ docker run --rm --net test netchat:local ss -ln
	Netid   State    Recv-Q   Send-Q     Local Address:Port      Peer Address:Port
	nl      UNCONN   0        0                      0:730                   *
	nl      UNCONN   0        0                      0:0                     *
	nl      UNCONN   4352     0                      4:1                     *
	nl      UNCONN   768      0                      4:0                     *
	nl      UNCONN   0        0                      6:0                     *
	nl      UNCONN   0        0                      9:0                     *
	nl      UNCONN   0        0                     10:0                     *
	nl      UNCONN   0        0                     12:0                     *
	nl      UNCONN   0        0                     15:0                     *
	nl      UNCONN   0        0                     16:0                     *
	udp     UNCONN   0        0             127.0.0.11:42184          0.0.0.0:*
	tcp     LISTEN   0        4096          127.0.0.11:40815          0.0.0.0:*

Notice the last two lines!

The user-defined bridge networks use the "internal dns" which allows the container to resolve other containers' names :)

	$ docker run --rm --net test netchat:local cat /etc/resolv.conf
	search Home
	nameserver 127.0.0.11
	options edns0 trust-ad ndots:0

## test 3 - two containers, two networks

Let's assume we have two containers, in two separate (user-defined) networks:

	$ docker run --rm -ti --name netchat-server --net network_A netchat:local
	$ docker run --rm -ti --name netchat-client --net network_B netchat:local bash

Even if probably possible to set up some "router" connecting those two networks, the simplest solution is to add an additinal interface to one of the containers:

	$ docker network connect network_A netchat-client

It doesn't require any restart - the container is connected immediately, and you can run on it:

	(netchat-client) # nc netchat-server 4444

## test 4 - access the server (run in container) from the outside the docker environment

The parameter "port", "p" - allow to map any of the host ports to the container port (https://docs.docker.com/config/containers/container-networking/):

	$ docker run --rm -ti -p 4444:4444 netchat:local
