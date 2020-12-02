<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [GuestBook](#guestbook)
  - [Overview](#overview)
  - [Raw Resources](#raw-resources)
  - [Application Discovery](#application-discovery)
    - [All-in-one](#all-in-one)
      - [Preparation](#preparation)
      - [Discovery](#discovery)
    - [Multicluster](#multicluster)
  - [Model Resources](#model-resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

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

## Application Discovery

This folder contains the required resource to demonstrate how to discover and generate hybrid application model from existing resources

### All-in-one 

All-in-one is a single cluster environment to simulate multicluster behavior for demo/poc purpose. The multicluser engine in this demo is [open-cluster-management](https://github.com/open-cluster-management)

#### Preparation

- Install Hybrid Application Model operator from operatorhub.io (https://operatorhub.io/operator/ham-deploy)

- Deploy raw resources in a dedicated namespace

```shell
% kubectl create namespace guestbook
namespace/guestbook created
```

- Apply raw resources in guestbook namespace

```shell
% kubectl apply -f guestbook/raw -n guestbook
deployment.apps/frontend created
service/frontend created
deployment.apps/redis-master created
service/redis-master created
deployment.apps/redis-slave created
service/redis-slave created
% kubectl get deploy,svc -n guestbook
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend       3/3     3            3           21s
deployment.apps/redis-master   1/1     1            1           21s
deployment.apps/redis-slave    2/2     2            2           19s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/frontend       NodePort    10.108.42.109   <none>        80:31631/TCP   21s
service/redis-master   ClusterIP   10.110.64.132   <none>        6379/TCP       20s
service/redis-slave    ClusterIP   10.99.140.251   <none>        6379/TCP       19s
```

- Create cluster, cluster namespaces and all-in-one ham operator

```shell
% kubectl apply -f guestbook/discovery/prerequisites.yaml
customresourcedefinition.apiextensions.k8s.io/clusters.clusterregistry.k8s.io created
namespace/toronto created
cluster.clusterregistry.k8s.io/toronto created
operator.deploy.hybridapp.io/ham-sample created
% kubectl get clusters --all-namespaces
NAMESPACE   NAME      AGE
toronto     toronto   1m
% kubectl get operator,pod
NAME                                      AGE
operator.deploy.hybridapp.io/ham-sample   125m

NAME                                          READY   STATUS    RESTARTS   AGE
pod/ham-sample-pod                            3/3     Running   0          3m55s
```

#### Discovery

- To discover resources from a specific cluster, uncomment the `tools.hybridapp.io/hybrid-discovery-target: "{\"namespace\":\"toronto\",\"name\":\"toronto\"}"` annotation. The value of the annotation is a json representing a serialized kubernetes object reference (usually indicated by a name and a namespace). If this annotation is not defined, the discovery process will run across all managed clusters. 

- Apply Application for discovery tool

``` shell
% kubectl apply -f guestbook/discovery/model.yaml
application.app.k8s.io/gbapp created
% kubectl get deployables.apps.open-cluster-management.io -n toronto
NAME                                                         TEMPLATE-KIND   TEMPLATE-APIVERSION          AGE    STATUS
deployable-default-deployment-guestbook-frontend-crgxc       Deployable      core.hybridapp.io/v1alpha1   5s     
deployable-default-deployment-guestbook-redis-master-22m5f   Deployable      core.hybridapp.io/v1alpha1   5s     
deployable-default-deployment-guestbook-redis-slave-gw6t8    Deployable      core.hybridapp.io/v1alpha1   4s     
deployable-default-service-guestbook-frontend-44xjn          Deployable      core.hybridapp.io/v1alpha1   4s     
deployable-default-service-guestbook-redis-master-hc75w      Deployable      core.hybridapp.io/v1alpha1   4s     
deployable-default-service-guestbook-redis-slave-5wtpz       Deployable      core.hybridapp.io/v1alpha1   4s     
gbapp-tjzfl                                                  Application     app.k8s.io/v1beta1           104s 
```

- Trigger resource generation and assembly, uncomment the `#    tools.hybridapp.io/hybrid-discovery-create-assembler: "true"`

```shell
% kubectl apply -f guestbook/discovery/model.yaml                 
application.app.k8s.io/gbapp configured
% kubectl get appasmNAME    AGE
gbapp   15m
% kubectl get appasm gbapp -o yaml
apiVersion: tools.hybridapp.io/v1alpha1
kind: ApplicationAssembler
metadata:
  creationTimestamp: "2020-05-18T00:16:26Z"
  generation: 1
  name: gbapp
  namespace: default
  resourceVersion: "8350614"
  selfLink: /apis/tools.hybridapp.io/v1alpha1/namespaces/default/applicationassemblers/gbapp
  uid: d025d9fe-989c-11ea-84ae-00000a10203b
spec:
  managedClustersComponents:
  - cluster: toronto/toronto
    components:
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: service-guestbook-frontend-f749l
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: service-guestbook-redis-master-kzltc
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: service-guestbook-redis-slave-xtdqs
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-guestbook-frontend-5hvtn
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-guestbook-redis-master-ppzz2
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-guestbook-redis-slave-xvpnz
      namespace: toronto
status:
  lastUpdateTime: "2020-05-18T00:16:28Z"
  phase: ""
% kubectl get hdpl,placementrule
NAME                                                                     AGE
deployable.core.hybridapp.io/toronto-deployment-guestbook-frontend       67s
deployable.core.hybridapp.io/toronto-deployment-guestbook-redis-master   67s
deployable.core.hybridapp.io/toronto-deployment-guestbook-redis-slave    66s
deployable.core.hybridapp.io/toronto-service-guestbook-frontend          67s
deployable.core.hybridapp.io/toronto-service-guestbook-redis-master      67s
deployable.core.hybridapp.io/toronto-service-guestbook-redis-slave       67s

NAME                                                                                   AGE
placementrule.apps.open-cluster-management.io/toronto-service-guestbook-redis-master   67s
```

### Multicluster

Assuming the multicluster environment is already set, the only difference from the all-in-one scenario above is, deploy the guestbook raw example in the managed cluster rather than the hub cluster.

## Model Resources

Apply resources in this folder gives the application model directly.

```shell
% kubectl apply -f guestbook/model/all-in-one.yaml
application.app.k8s.io/ham-sample-gbapp created
deployable.core.hybridapp.io/deployment-guestbook-frontend created
deployable.core.hybridapp.io/service-guestbook-frontend created
placementrule.apps.open-cluster-management.io/ham-sample-guestbook created
deployable.core.hybridapp.io/deployment-guestbook-redis-master created
deployable.core.hybridapp.io/service-guestbook-redis-master created
deployable.core.hybridapp.io/deployment-guestbook-redis-slave created
deployable.core.hybridapp.io/service-guestbook-redis-slave created

% kubectl get deployables.apps.open-cluster-management.io -n toronto
NAME                                  TEMPLATE-KIND   TEMPLATE-APIVERSION   AGE   STATUS
deployment--frontend-bsj95            Deployment      apps/v1               26s   
deployment--redis-master-f2j88        Deployment      apps/v1               26s   
deployment--redis-slave-7hsfj         Deployment      apps/v1               26s   
service--frontend-wpmkt               Service         v1                    26s   
service--redis-master-jbpz8           Service         v1                    26s   
service-guestbook-redis-slave-6zsjc   Service         v1                    26s   

```