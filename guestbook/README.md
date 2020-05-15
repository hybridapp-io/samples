# GuestBook

GUestBook is example provided in [Kubernetes community](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/). This [github repository](https://github.com/kubernetes/examples/tree/master/guestbook) contains all the required files for this example. 

## Overview

This sample directory is organized as below:

- raw: kubernetes files copied from original github repo with additional label for discovery
- discovery: kubernetes resources for hybrid application model tools to discover guestbook resources and assemble application
- model: Hybrid Application Model resource to manage guestbook

## Raw Resources

There are 3 deployments and 3 services in original guestbook example, We added a label `ham-sample-app: guestbook` to all those 6 resource to facilitate the demo.

After you cloned this repository, you should be able to deploy those original example files directly as below:

```shell
% kubectl apply -f raw                                       
deployment.apps/frontend created
service/frontend created
deployment.apps/redis-master created
service/redis-master created
deployment.apps/redis-slave created
service/redis-slave created
% kubectl get deployments,service -l ham-sample-app=guestbook
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend       3/3     3            3           7s
deployment.apps/redis-master   1/1     1            1           6s
deployment.apps/redis-slave    2/2     2            2           6s

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/frontend       NodePort    10.106.146.25    <none>        80:30694/TCP   6s
service/redis-master   ClusterIP   10.110.76.134    <none>        6379/TCP       6s
service/redis-slave    ClusterIP   10.108.119.130   <none>        6379/TCP       5s
```

In order to better demo the functions, it is recommended to apply frontend.yaml and redis-slave.yaml to a cluster, and deploy redis-master.yaml to another cluster

## Discovery Resources

## Model Resources
