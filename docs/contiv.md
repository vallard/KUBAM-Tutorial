# KUBAM Contiv Lab

In this lab we'll take a look at Contiv to see what it provides for us.  

## 1.1 Log in

Log into the ```kubam01``` server, or master node that is used for your kubernetes cluster.  This is the node you used last time.

## 1.2 ```netctl```

```netctl``` is a utility to create, update, read and modify contiv objects.  It is a CLI wrapper on top of a REST interface.  On the ```kubam01``` server run the following:

```
netctl version
```

```
netctl ls
```
This will show that two networks are created

```
Tenant   Network      Nw Type  Encap type  Packet tag  Subnet         Gateway     IPv6Subnet  IPv6Gateway  Cfgd Tag
------   -------      -------  ----------  ----------  -------        ------      ----------  -----------  ---------
default  contivh1     infra    vxlan       0           132.1.1.0/24   132.1.1.1
default  default-net  data     vxlan       0           172.16.0.0/16  172.16.0.1
```
This is the default way contiv is installed.  The ```contivh1``` network is for the infrastructure.  The other network ```default-net``` contains the list of IP addresses that default containers will get for their IP addresses.  

## 1.3 Creating networks

Let's make a fresh container network

```
netctl net create --subnet=10.1.2.0/24 --gateway=10.1.2.1 contiv-net
```
Now let's inspect the network

```
netctl net inspect contiv-net
```
This shows us the characteristics of the new network

```
{
  "Config": {
    "key": "default:contiv-net",
    "encap": "vxlan",
    "gateway": "10.1.2.1",
    "networkName": "contiv-net",
    "nwType": "data",
    "subnet": "10.1.2.0/24",
    "tenantName": "default",
    "link-sets": {},
    "links": {
      "Tenant": {
        "type": "tenant",
        "key": "default"
      }
    }
  },
  "Oper": {
    "allocatedIPAddresses": "10.1.2.1",
    "availableIPAddresses": "10.1.2.2-10.1.2.254",
    "externalPktTag": 3,
    "networkTag": "contiv-net.default",
    "pktTag": 3
  }
}
```

## 1.4 Create a Pod using the Contiv Network

We can now spin a couple of pods and add them to the ```contiv-net``` network.  Let’s create the first pod defining it should run with the contiv-net and default tenant: 

```
cat <<EOF > contiv-c1.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    io.contiv.tenant: default
    io.contiv.network: contiv-net
    k8s-app: contiv-c1
  name: contiv-c1
spec: 
  containers: 
    - 
      image: contiv/alpine
      name: alpine
      command: 
      - sleep
      - "6000"
EOF

```
Then start it:

```
kubectl create -f contiv-c1.yaml
```


Now let's create another container on that network called ```contiv-c2```

```
cat <<EOF > contiv-c2.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    io.contiv.tenant: default
    io.contiv.network: contiv-net
    k8s-app: contiv-c2
  name: contiv-c2
spec: 
  containers: 
    - 
      image: contiv/alpine
      name: alpine
      command: 
      - sleep
      - "6000"
EOF
```
Then start it:

```
kubectl create -f contiv-c2.yaml
```

Now we should have two contiv containers:

```
kubectl get pods -o wide
```

Notice the different networks they have.

```
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
contiv-c1                1/1       Running   0          2m        10.1.2.2     kubam03
contiv-c2                1/1       Running   0          9s        10.1.2.3     kubam02
nginx-4217019353-2tsl5   1/1       Running   0          19h       172.16.0.3   kubam03
nginx-4217019353-8vqpf   1/1       Running   0          19h       172.16.0.5   kubam02
nginx-4217019353-8wvk6   1/1       Running   0          19h       172.16.0.6   kubam03
nginx-4217019353-jlqgj   1/1       Running   0          19h       172.16.0.4   kubam02
```

We can now log into ```contiv-c1``` and ping between the two containers:

```
kubectl exec -it contiv-c1 sh
```
On the container run:

```
# ping contiv-c2
```


## All Done!

If you made it this far you have completed the lab!  Great job!  Go [back to the beginning](../README.md) and do it all again or do something else more fun. 

Send feedback to [@vallard](https://twitter.com/vallard) 



