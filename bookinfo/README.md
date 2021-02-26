<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [BookInfo](#bookinfo)
  - [Overview](#overview)
  - [Raw Resources](#raw-resources)
  - [Discovery Resources](#discovery-resources)
  - [Model Resources](#model-resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# BookInfo

BookInfo is example provided by [Istio community](https://istio.io/docs/examples/virtual-machines/bookinfo/). This [github repository](https://github.com/istio/istio/tree/master/samples/bookinfo) contains all the required files for this example. 

## Overview

This sample directory is organized as below:

- raw: kubernetes files copied from original github repo with additional label for discovery
- discovery: kubernetes resources for hybrid application model tools to discover bookinfo resources and assemble application
- model: Hybrid Application Model resource to manage bookinfo

## Raw Resources

There are 4 deployments, 4 services and 4 serviceaccounts in original bookinfo example, We added a label `ham-sample-app: bookinfo` to all those 12 resource to facilitate the demo.

After you cloned this repository, you should be able to deploy those original example files directly as below:

```shell
% kubectl apply -f raw
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
% kubectl get deploy,svc,sa -l ham-sample-app=bookinfo
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/details-v1       1/1     1            1           37s
deployment.apps/productpage-v1   0/1     1            0           37s
deployment.apps/ratings-v1       1/1     1            1           36s
deployment.apps/reviews-v1       1/1     1            1           35s
deployment.apps/reviews-v2       1/1     1            1           35s
deployment.apps/reviews-v3       1/1     1            1           35s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/details       ClusterIP   10.102.18.27    <none>        9080/TCP   39s
service/productpage   ClusterIP   10.108.69.159   <none>        9080/TCP   38s
service/ratings       ClusterIP   10.100.236.21   <none>        9080/TCP   37s
service/reviews       ClusterIP   10.97.9.132     <none>        9080/TCP   37s

NAME                                  SECRETS   AGE
serviceaccount/bookinfo-details       1         38s
serviceaccount/bookinfo-productpage   1         38s
serviceaccount/bookinfo-ratings       1         37s
serviceaccount/bookinfo-reviews       1         36s
```

## Discovery

- To discover resources from a specific cluster, uncomment the `tools.hybridapp.io/hybrid-discovery-target: "{\"name\":\"toronto\"}"` annotation. The value of the annotation is a json representing a serialized kubernetes object reference (usually indicated by a cluster name). If this annotation is not defined, the discovery process will run across all managed clusters. 

- Apply Application for discovery tool

``` shell
% kubectl apply -f bookinfo/discovery/model.yaml
application.app.k8s.io/bookinfo created
% kubectl get deployables.apps.open-cluster-management.io -n toronto
NAME                                                         TEMPLATE-KIND   TEMPLATE-APIVERSION          AGE    STATUS
deployment-bookinfo-details-v1-lp2qh                         Deployment       apps/v1                              64m     Deployed
deployment-bookinfo-productpage-v1-w874n                     Deployment       apps/v1                              64m     Deployed
deployment-bookinfo-ratings-v1-szqk6                         Deployment       apps/v1                              64m     Deployed
deployment-bookinfo-reviews-v1-x526q                         Deployment       apps/v1                              64m     Deployed
deployment-bookinfo-reviews-v2-wmfpd                         Deployment       apps/v1                              64m     Deployed
deployment-bookinfo-reviews-v3-2k5mx                         Deployment       apps/v1                              64m     Deployed
service-bookinfo-details-6z9mv                               Service          v1                                   64m     Deployed
service-bookinfo-productpage-t49lq                           Service          v1                                   64m     Deployed
service-bookinfo-ratings-z574w                               Service          v1                                   64m     Deployed
service-bookinfo-reviews-9zwcx                               Service          v1                                   64m     Deployed
serviceaccount-bookinfo-bookinfo-details-grp4s               ServiceAccount   v1                                   63m     Deployed
serviceaccount-bookinfo-bookinfo-productpage-xxbqp           ServiceAccount   v1                                   63m     Deployed
serviceaccount-bookinfo-bookinfo-ratings-mx77l               ServiceAccount   v1                                   63m     Deployed
serviceaccount-bookinfo-bookinfo-reviews-pzgjz               ServiceAccount   v1                                   63m     Deployed
```

- Trigger resource generation and assembly, uncomment the `#    tools.hybridapp.io/hybrid-discovery-create-assembler: "true"`

```shell
% kubectl apply -f bookinfo/discovery/model.yaml                 
application.app.k8s.io/bookinfo configured
% kubectl get appasm
bookinfo   64m
% kubectl get appasm bookinfo -o yaml
apiVersion: tools.hybridapp.io/v1alpha1
kind: ApplicationAssembler
metadata:
  creationTimestamp: "2020-11-27T14:38:04Z"
  generation: 1
  name: bookinfo
  namespace: bookinfo
  resourceVersion: "60445063"
  selfLink: /apis/tools.hybridapp.io/v1alpha1/namespaces/bookinfo/applicationassemblers/bookinfo
  uid: ff159187-5efe-422b-9b0c-24e4a615de37
spec:
  managedClustersComponents:
  - cluster: toronto
    components:
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: service-bookinfo-details-6z9mv
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: service-bookinfo-reviews-9zwcx
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: service-bookinfo-productpage-t49lq
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: service-bookinfo-ratings-z574w
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-bookinfo-details-v1-lp2qh
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-bookinfo-reviews-v3-2k5mx
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-bookinfo-ratings-v1-szqk6
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-bookinfo-reviews-v1-x526q
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-bookinfo-productpage-v1-w874n
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: deployment-bookinfo-reviews-v2-wmfpd
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: serviceaccount-bookinfo-bookinfo-ratings-mx77l
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: serviceaccount-bookinfo-bookinfo-reviews-pzgjz
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: serviceaccount-bookinfo-bookinfo-productpage-xxbqp
      namespace: toronto
    - apiVersion: apps.open-cluster-management.io/v1
      kind: Deployable
      name: serviceaccount-bookinfo-bookinfo-details-grp4s
      namespace: toronto
status:
  lastUpdateTime: "2020-11-27T14:38:08Z"
  phase: ""
% kubectl get hdpl,hpr
NAME                                                                            AGE
deployable.core.hybridapp.io/toronto-deployment-bookinfo-details-v1                 65m
deployable.core.hybridapp.io/toronto-deployment-bookinfo-productpage-v1             65m
deployable.core.hybridapp.io/toronto-deployment-bookinfo-ratings-v1                 65m
deployable.core.hybridapp.io/toronto-deployment-bookinfo-reviews-v1                 65m
deployable.core.hybridapp.io/toronto-deployment-bookinfo-reviews-v2                 65m
deployable.core.hybridapp.io/toronto-deployment-bookinfo-reviews-v3                 65m
deployable.core.hybridapp.io/toronto-service-bookinfo-details                       65m
deployable.core.hybridapp.io/toronto-service-bookinfo-productpage                   65m
deployable.core.hybridapp.io/toronto-service-bookinfo-ratings                       65m
deployable.core.hybridapp.io/toronto-service-bookinfo-reviews                       65m
deployable.core.hybridapp.io/toronto-serviceaccount-bookinfo-bookinfo-details       65m
deployable.core.hybridapp.io/toronto-serviceaccount-bookinfo-bookinfo-productpage   65m
deployable.core.hybridapp.io/toronto-serviceaccount-bookinfo-bookinfo-ratings       65m
deployable.core.hybridapp.io/toronto-serviceaccount-bookinfo-bookinfo-reviews       65m

NAME                                                           AGE
placementrule.core.hybridapp.io/toronto-service-bookinfo-details   65m
```


## Model Resources

Apply resources in this folder gives the application model directly.

```shell
% kubectl apply -f bookinfo/model/all-in-one.yaml
application.app.k8s.io/bookinfo created
placementrule.core.hybridapp.io/ham-sample-bookinfo created
deployable.core.hybridapp.io/service-bookinfo-details created
deployable.core.hybridapp.io/serviceaccount-bookinfo-details created
deployable.core.hybridapp.io/deployment-bookinfo-details-v1 created
deployable.core.hybridapp.io/service-bookinfo-productpage created
deployable.core.hybridapp.io/serviceaccount-bookinfo-bookinfo-productpage created
deployable.core.hybridapp.io/deployment-bookinfo-productpage-v1 created
deployable.core.hybridapp.io/service-bookinfo-ratings created
deployable.core.hybridapp.io/serviceaccount-bookinfo-bookinfo-ratings created
deployable.core.hybridapp.io/deployment-bookinfo-ratings-v1 created
deployable.core.hybridapp.io/service-bookinfo-reviews created
deployable.core.hybridapp.io/serviceaccount-bookinfo-bookinfo-reviews created
deployable.core.hybridapp.io/deployment-bookinfo-reviews-v1 created
deployable.core.hybridapp.io/deployment-bookinfo-reviews-v2 created
deployable.core.hybridapp.io/deployment-bookinfo-reviews-v3 created

% kubectl get deployables.apps.open-cluster-management.io -n toronto
NAME                                  TEMPLATE-KIND   TEMPLATE-APIVERSION   AGE   STATUS
deployment-bookinfo-model-details-v1-gqmdv                   Deployment       apps/v1                        18m     Deployed
deployment-bookinfo-model-productpage-v1-fp6xg               Deployment       apps/v1                        18m     Deployed
deployment-bookinfo-model-ratings-v1-2d2dx                   Deployment       apps/v1                        18m     Deployed
deployment-bookinfo-model-reviews-v1-hc28l                   Deployment       apps/v1                        18m     Deployed
deployment-bookinfo-model-reviews-v2-7lnx2                   Deployment       apps/v1                        18m     Deployed
deployment-bookinfo-model-reviews-v3-4kztz                   Deployment       apps/v1                        18m     Deployed
service-bookinfo-model-details-dxr6d                         Service          v1                             18m     Deployed
service-bookinfo-model-productpage-7xbqp                     Service          v1                             18m     Deployed
service-bookinfo-model-ratings-59msk                         Service          v1                             18m     Deployed
service-bookinfo-model-reviews-sz2kb                         Service          v1                             18m     Deployed
serviceaccount-bookinfo-model-bookinfo-details-kfxh7         ServiceAccount   v1                             18m     Deployed
serviceaccount-bookinfo-model-bookinfo-productpage-pdtzz     ServiceAccount   v1                             18m     Deployed
serviceaccount-bookinfo-model-bookinfo-ratings-v9cfp         ServiceAccount   v1                             18m     Deployed
serviceaccount-bookinfo-model-bookinfo-reviews-m9v4b         ServiceAccount   v1                             18m     Deployed
```
