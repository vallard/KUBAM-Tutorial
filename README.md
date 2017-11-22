# KUBAM dCloud Tutorial

This repository contains the instructions for deploying KUBAM.  Perhaps you've never heard of KUBAM?  Let's go to that right now: 

### KUBAM

[KUBAM](https://kubam.io) is an [open source tool](https://github.com/CiscoUcs/KUBaM) for automatically deploying solutions on top of Cisco Unified Compute System (UCS).  

#### UCS
UCS is a product suite from Cisco that includes Intel Servers in blade and rack-mount form factor that are managed through a pair of Fabric Interconnects.  The advantages include simplification of management, future-proofing investment, and less money spent on network infrastructure. 
To learn more about Cisco Unified Computing System, please [go here](www.cisco.com/go/ucs)

## This Lab

Since the UCS APIs are open, KUBAM uses them to create custom solutions.  In this lab we will walk over how to create a Kubernetes cluster from scratch on UCS.  It will probably be boring because most of the time you'll just be twiddling your thumbs waiting for the system to come on line.  You'll do a few quick tasks and then wait for Kubernetes to come up. Then you'll do some fun kubernetes work.  At the end you should learn:

* How KUBAM simplifies OS installation on UCS including Service Profile creation, OS installation without PXEboot, and automated solution creation. 
* Kubernetes concepts
* Cisco Contiv

Ready to start?  Let's go!

## Lab Modules

1. [dCloud](docs/dcloud.md) - Here you will access your UCS resources to play with and try out. The rest of the labs assume that you are using this dCloud environment.
* [KUBAM Installation](docs/install.md) - In this lab we will show the super simple way to install KUBAM.  So boring!
* [Install Kubernetes with KUBAM](docs/menus.md) - In this lab we'll walk through the menus of the KUBAM web interface and provision Kubernetes on UCS.  
* [Trobleshooting KUBAM](docs/trouble.md) - In this lab we'll talk about things that could go wrong, how to troubleshoot, and where to look for help. 
* [Kubernetes](docs/kubernetes.md) - In this lab we'll kick the tires of our Kubernetes cluster, run some services, deployments, and pods and have some fun. 
* [Contiv](docs/contiv.md) - In this lab we'll explore the features of Contiv and see how it helps our Kubernetes cluster.