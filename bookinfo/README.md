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

## Discovery Resources

## Model Resources
