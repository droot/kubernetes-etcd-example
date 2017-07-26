# Single Node Etcd Cluster with Persistent Storage on K8s
This document describes the steps to run a single instance Etcd cluster backed
by persistent volume (Google Persistent disks) on Kubernetes. We use StatefulSet
primitive to run Etcd so that it is stable with persistence across Pod (re)scheduling.

## Prerequisites
The steps below assume that you have a running Kubernetes cluster and kubectl is
configured to connect to the cluster.

## Setup
Run following command to deploy etcd with GCP persistent volume.
```sh
kubectl create -f statefulset/
```

## Verification
Inspect etcd artifacts in Kubernetes cluster.
Run following command to verify if service object has been created.
```sh
kubectl get svc/etcd-svc statefulsets/etcd po/etcd-0 pvc/etcd-data-dir-etcd-0 -o wide
```

You should see an output like this
```
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE SELECTOR
svc/etcd-svc   ClusterIP   10.11.254.21   <none>        2379/TCP   4h app=etcd

NAME                DESIRED   CURRENT   AGE
statefulsets/etcd   1         1         4h        etcd quay.io/coreos/etcd:latest   app=etcd

NAME        READY     STATUS    RESTARTS   AGE       IP         NODE
po/etcd-0   1/1       Running   0          3h        10.8.2.7   gke-cluster-1-default-pool-835cace8-nvmn

NAME                       STATUS    VOLUME                                   CAPACITY   ACCESSMODES   STORAGECLASS   AGE
pvc/etcd-data-dir-etcd-0   Bound     pvc-8fd97ce7-7172-11e7-bf09-42010a800061 10Gi       RWO           standard       4h
```


## Testing
In order to verify the persistance stability across rescheduling of pods,
  * create some data in Etcd using etcdctl command
  * Force rescheduling of Pod to another node. You can use 'kubectl drain
    <node>' to do that.
  * Verify that data exist after Etcd pod is rescheduled and UP on another node.
