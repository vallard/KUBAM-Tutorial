# KUBAM Installation Lab

The purpose of this module is to show how quickly and efficiently you can install KUBAM. It’s so easy it will be something you’ll want to do on a Friday night just for fun.  

 


## 1.1 Log in to KUBAM server

After connecting to the VPN you should now use a terminal (or Putty for Windows ) to connect to the KUBAM server.  This server is called ```utility1``` and the IP address is ```198.18.134.242```.  The user is ```root``` and password is ```C1sco12345```

```
ssh root@198.18.134.242
[root@razor1 ~]#
```
## 1.2 Install Prerequisites

The prereqs for KUBAM are docker and docker compose.  Perform the following on the utility1 server.  First install Docker:
yum install –y yum-utils 

Next install Docker Compose.  Docker Compose provides a simple way to bring up multiple containers at the same time.  Since KUBAM is based on two containers that work together we simplify the installation using Docker Compose.  Perform the following on the utility1 server:

```
curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-Linux-x86_64 \

Notice the above are just two commands. This grabs the correct compose version from github and then makes it an executable on our system.  Now we have docker and docker compose.  Super simple.  

