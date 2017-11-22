# KUBAM Installation Lab

The purpose of this module is to show how quickly and efficiently you can install KUBAM. It’s so easy it will be something you’ll want to do on a Friday night just for fun.  
We first install [Docker](https://docker.com) and then [Docker Compose](https://docs.docker.com/compose/).  From there we only need two commands and KUBAM is up and ready for us to use.  So simple it’s scary.  Easiest install you’ll ever do.  For the bulk of this lab you will be just waiting and twiddling your thumbs.
 KUBAM creates the necessary components needed to install a Kubernetes cluster on UCS bare metal from scratch.  One of the huge advantages of UCS and KUBAM is that a PXE server is not required to do automated operating system (OS) installations.  No external agent, just KUBAM to do all the work.  
Additionally, the goal of this exercise is to also be exposed to some of the DevOps tools commonly used. In this this scenario we will use Docker and Docker Compose.

## 1.1 Log in to KUBAM server

After connecting to the VPN you should now use a terminal (or Putty for Windows ) to connect to the KUBAM server.  This server is called ```utility1``` and the IP address is ```198.18.134.242```.  The user is ```root``` and password is ```C1sco12345```

```
ssh root@198.18.134.242
[root@razor1 ~]#
```
## 1.2 Install Prerequisites

The prereqs for KUBAM are docker and docker compose.  Perform the following on the utility1 server.  First install Docker:```
yum install –y yum-utils yum-config-manager –-add-repo https://download.docker.com/linux/centos/docker-ce.repoyum install –y docker-cesystemctl start dockersystemctl enable docker```

Next install Docker Compose.  Docker Compose provides a simple way to bring up multiple containers at the same time.  Since KUBAM is based on two containers that work together we simplify the installation using Docker Compose.  Perform the following on the utility1 server:

```
curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-Linux-x86_64 \		  -o /usr/local/bin/docker-composechmod +x /usr/local/bin/docker-compose```

Notice the above are just two commands. This grabs the correct compose version from github and then makes it an executable on our system.  Now we have docker and docker compose.  Super simple.  What could go wrong?  Well, if you had a system that already had docker installed these instructions may not work.  Notice that we aren’t using the docker that comes default with the operating system.  So if that were installed you would uninstall it first.  Also if SELinux is enabled that is going to be a problem.  For now we ignore the problem by __making sure selinux is inactive__.  This server already has it disabled so we are good to go for the next step.  

