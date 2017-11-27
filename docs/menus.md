# KUBAM Configuration Lab

The purpose of this module is to walk through the different menus of KUBAM and configure our turn key Kubernetes environment.  The screens may change over time but the ideas of the parameters will be the same. 

Please make sure you [completed the previous lab](./install.md) before proceeding to this lab. 

## 1.1 Authenticate UCS with KUBAM

Now that we know how to get into UCS, we can go back to the KUBAM interface and login using the same credentials we did for the UCS web interface.  This will let KUBAM know about how to connect to UCS. Enter the UCS credentials in a slightly different way:

* User: __ucs-dCloud\demouser__
* Password: __C1sco12345__
* UCS: __198.18.0.10__
 
![login screen](../images/kubam02.png)

Once the authentication has succeeded you will see that KUBAM is logged in. 

![logged in screen](../images/kubam03.png)

KUBAM logins persist.  All the data that you enter is stored in ```~/kubam/kubam.yaml``` file on the KUBAM server.  You can go back to your terminal and see this file now.  You will see that as you enter fields into the GUI it is saved in the ```kubam.yaml``` file in YAML form
