# KUBAM Kubernetes Lab

At this point, if you completed the [last lab](./menus.md) your nodes should be up because KUBAM did everything for you because it is so rad.  If it's still not up, take heart as it takes about 20 minutes to complete. 

## 1.1 Test Kubernetes

The kubam server is ```198.18.134.242```, you are probably already logged in if you finished the [last lab](./menus.md).  If not login here: 

```
ssh root@198.18.134.242
```
Password is ```C1sco12345```. 

Let's now see if the nodes are up.  From the utility server (aka kubam server) SSH into the first node:

```
ssh kubam01
```
You should not need to enter a password as kubam used the public key.  The first time you log in you will get a challenge and you can type ```yes``` and log into the node. 

```
# ssh kubam01
The authenticity of host 'kubam01 (198.18.0.160)' can't be established.
ECDSA key fingerprint is 09:e6:01:e1:2f:1a:57:ec:58:7c:cf:a3:96:e5:76:93.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'kubam01,198.18.0.160' (ECDSA) to the list of known hosts.
[root@kubam01 ~]#
```

From here, let's see how our kubernetes cluster is doing:

```
kubectl get nodes
```
This should give you output like: 

```
# kubectl get nodes
NAME      STATUS    AGE       VERSION
kubam01   Ready     2h        v1.7.1
kubam02   Ready     2h        v1.7.1
kubam03   Ready     2h        v1.7.1
```

If you get all three nodes up you are ready to go.  If not, take a look at the [troubleshooting lab](./trouble.md) to see what might be wrong.  Usually opening the KVM console of a bad node could give you a good idea of what is happening.

#### What is the password to log into the nodes? 

What is the default password of the nodes installed by KUBAM?  There is none.  As developers we scrambled the key so even we don't know what it is.  If you want to change that you need to create your own kickstart file.  

## 1.2 Exploring Kubernetes Cluster

Let's see what kind of containers are running on this cluster? 

```
kubectl get pods
```

The results show that nothing is running.  But that's not entirely true.  The fact is KUBAM uses [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) to install Kubernetes.  It installs it in a way known as self-hosting, meaning the binaries that control how kubernetes works are actually running as pods in the kubernetes cluster.  Sort of egg and chicken in the same nest sort of idea.  

The kubernetes pods are running in a different namespace called ```kube-system```.  To see what's running, we can now do the same command with a few more flags:

```
kubectl get pods -o wide -w --all-namespaces
```
This command has some flags:

* ```-o wide``` tell it to print which nodes the pods are running on. 
* ```-w``` tells the command to watch it, sort of like a ```tail -f``` command would do. 
* ```--all-namespaces``` gets all the name spaces that are in the cluster.  We could have instead specified ```-n kube-system``` to just get a different namespace. 

### 1.2.1 Common Errors: kube-dns not working

Usually the cluster will come right up. But sometimes you'll have problems.  Look for the pod called ```kube-dns-xxxxxxxxxxx-xxxxx```.  Make sure that all 3/3 pods are up.  If it looks something like:

```
kube-system   kube-dns-2425271678-7p1vl         1/3       CrashLoopBackOff   74         3h        172.16.0.2     kubam03
```
It means there is a problem.  Let's fix it!  If yours says ```3/3``` and all are up, then no need to worry!  Go to the next section.

It turns out the ```kube-dns``` pod needs to run on the ```kube01``` master node to be with the contiv ```net-master```.  We can make this happen by creating some labels. 

```
kubectl label nodes kubam01 nettype=contiv-master
```

Now we can edit the ```kube-dns``` to run on this node.  We update the ```deployment``` of the node:

```
kubectl edit deployment kube-dns -n kube-system
```
This pulls up a VIM editor to edit the deployment.  Notice where we add the ```nodeSelector``` label.  Add yours around the same place.  

```
...
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                - amd64
      nodeSelector:
        nettype: contiv-master
      containers:
      - args:
        - --domain=cluster.local.
        - --dns-port=10053
        - --config-dir=/kube-dns-config
        - --v=2
        env:
        - name: PROMETHEUS_PORT
          value: "10055"
...
```

If you get errors while saving make sure you are putting it in the right place.  There are two ```spec``` fields in the definition, you want to put it right above the ```containers``` definition.

Running our command we should see the new pods get placed on the right node and work: 

```
kubectl get pods --all-namespaces -o wide -w
```

And eventually you should see:

```
kube-system   kube-dns-1954560882-gv1t9   3/3       Running   0         20s       172.16.0.2   kubam01
```

## 1.3 Kubernetes Deployments

Let's run a web deployment to see how easy this really is: 

```
kubectl run nginx --image=nginx
```
Now see if its up:

```
kubectl get pods
```
Notice that this time it is running in our own default namespace. 

The ```run``` command creates a [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) super simple without us having to write any yaml files.  We can now modify this to tell it we want 3 nginx pods running:

```
kubectl edit deployment nginx
```

Change the replica count from ```1``` to ```3```.  

```diff
-   replicas: 1
+   replicas: 3
```
Saving the file, then notice how many we have: 

```
kubect get pods -o wide -w
```
You'll see 3 pods come on line now.  Let's kill one of the pods and see what happens: 

```
kubect delete pod nginx-xxxxxxxxxx-xxxxx
```
(note replace the xxx's with the number of one of your outputs.  It doesn't matter which pod you delete.)

Now see how many pods you have: 

```
kubect get pods -o wide -w
```
You still have 3!  What's going on?  Kubernetes knows you want three, so you have to scale it.  You can make it even larger without editing the file:

```
kubectl scale deployment nginx --replicas=4
```

## 1.4 Kubernetes Service

But how can we access this pod? One option is to ping the IP address of the pod.  The 

```
kubectl get pods -o wide
```

Will give us the IP addresses of the running pods.  Yours might be something like: 

```
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-4217019353-2tsl5   1/1       Running   0          8m        172.16.0.3   kubam03
nginx-4217019353-8vqpf   1/1       Running   0          4m        172.16.0.5   kubam02
nginx-4217019353-8wvk6   1/1       Running   0          1m        172.16.0.6   kubam03
nginx-4217019353-jlqgj   1/1       Running   0          5m        172.16.0.4   kubam02
```
We can connect to the IP address of any of these pods and get the web page:

```
curl 172.16.0.3
```
(use one of the IP addresses shown in your own output)

But these containers come and go and the IP addresses change.  What if instead we had a virtual IP address that could connect to any of these servers?  

That's where a [kubernetes service](https://kubernetes.io/docs/concepts/services-networking/service/) comes in handy.  Let's create one:

```
kubectl expose deployment nginx --port=80
```

This creates a service.  Let's have a look:

```
kubectl get svc
```
The output looks like:

```
NAME         CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes   10.96.0.1     <none>        443/TCP   3h
nginx        10.96.38.90   <none>        80/TCP    33s
```
Here we see that the IP address for the service for the ```nginx``` service is ```10.96.38.90```.  We can now curl this IP address: 

```
curl 10.96.38.90
```
This gives us the same webpage!  It routes to one of the four pods that are running in our cluster.  Now if one of them changes or goes up the service will route to whatever is still available in the cluster. 

## 1.5 Kube DNS

Kubernetes services are helpful when we have some pods that require the use of other pods.  Let's show an example:

```
kubectl run bb --image=busybox -- sleep 3000
```
This creates a deployment that is a busybox container that runs the ```sleep 3000``` command when it boots up.  It basically is up and doesn't do anything but sleep.  

Let's attach to this pod and run some commands:

```
kubectl get pods
```

You'll see the pod named ```bb-xxxxxxxxxx-xxxxx```.  Copy this name and lets attach to it running:

```
kubectl exec -it bb-xxxxxxxxxx-xxxxx /bin/sh
```
This will put you on the command line of the running container.  

Let's look up the nginx service:

```
nslookup nginx.default.svc.cluster.local
```
You'll see that ```kube-dns``` at IP address ```10.96.0.10``` resolved the name of the service to ```10.96.38.90```.  In this way if an application were running in this container it could contact the nginx service by using either:

* ```nginx```
* ```nginx.default```
* ```nginx.default.svc```
* ...

Most likely you'd just put ```nginx``` or whatever the service name is to keep it simple. 

Exit out of the container and delete the deployment:

```
/ # exit
[root@kubam01 ~]# kubectl delete deployment bb
```

# Go to Next Lab

If you made it this far, great job!  Now let's go to the [next lab](./contiv.md) and take a look at Contiv.  

Or you can [go back to the beginning](../README.md)





 

