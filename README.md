# Hazelcast running on K8s ARM7 RPi
## Introduction:
This work was derived from the [hazelcast-openshift-origin](https://github.com/hazelcast/hazelcast-openshift/tree/master/hazelcast-openshift-origin) repo. It has been modified to run on K8s (Kubernetes) on ARM7 Raspberry Pi hardware. It's assumed that you have a working K8s install running on a RPi cluster.

**In Scope:**

* Building Hazelcast containers compatible with the Pi hardware (ARM7) and that use the K8s discovery module to auto-join Hazelcast cluster members. The image can be pulled from [DockerHub](https://hub.docker.com/r/dxjohnson/hazelcast-k8s-rpi/).
* The resource definitions specific to K8s are provided. There are subtle (and sometimes not so subtle) difference in how resources are defined in OpenShift vs. K8s.
* Basic instructions are provided to setup a Hazelcast cluster K8s.

**Out of Scope:**

* If you don't want to use the K8s discovery module, it can be removed from the hazelcast.xml configuration file and the container rebuilt using the Dockerfile.
* This work does not cover the building of a Pi Cluster or installing and configuring the K8s cluster on the Pi. Perhaps at some future date I'll provide some links on how to do all of that.
* SPI modules for Hazelcast. It's worth reading the [Hazelcast Blog](https://blog.hazelcast.com/openshift/) for details about its auto-discovery capability on Kubernetes/OpenShift. There are also  [discovery plug-ins](https://hazelcast.org/plugins/?type=cloud-discovery) for other platforms, or you could [write your own](https://blog.hazelcast.com/hazelcast-discovery-spi/).
* K8s tools and configuration. Consult the [K8s documentation](https://kubernetes.io/docs/home/) for additional details.


## Tested on:

* 4-node Raspberry Pi B "cluster" - 1.2Ghz 1GB
* HypriotOS-v7+ armv7l GNU/Linux
* Kubernetes v1.8.0+


## Usage

Assuming a 4-node Pi cluster (one K8s Master and three compute nodes) and that the Pi cluster has access to the Internet, login to the K8s Master node (no root needed) and execute the following:

```bash
cd ~
git clone https://github.com/davidxjohnson/hazelcast-k8s-rpi
cd hazelcast-k8s-rpi
kubectl create -f hazelcast-service.json    # creates the load-balance service
kubectl create -f hazelcast-replicator.json # creates a replication controller and 3 pods
kubectl get all
```

It may take a while for the pods to stand up as the Docker image is downloaded to each K8s compute node.

```bash
HypriotOS/armv7: pirate@rpi-white in ~/hazelcast-k8s-rpi on master
$ kubectl get all
NAME                      READY     STATUS    RESTARTS   AGE
po/hazelcast-node-br7rj   1/1       Running   0          3h
po/hazelcast-node-lwzt6   1/1       Running   0          3h
po/hazelcast-node-tmz88   1/1       Running   0          3h

NAME                DESIRED   CURRENT   READY     AGE
rc/hazelcast-node   3         3         3         18h

NAME                    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
svc/hazelcast-cluster   ClusterIP   None         <none>        5701/TCP   1d
```

To verify that the Hazelcast cluster is formed, look at the logs for each pod:

```bash
HypriotOS/armv7: pirate@rpi-white in ~/hazelcast-k8s-rpi on master
$ for pod in $(kubectl get pods -o name); do
    echo "----- $pod ----"
    kubectl logs $pod | grep Member;
  done

----- po/hazelcast-node-br7rj ----
Members [3] {
        Member [10.40.0.1]:5701 - d3bbf8bd-09bc-4e18-ae53-aa21e9785e69
        Member [10.32.0.5]:5701 - 74ad2a5d-9aca-44c5-a6a4-c2ee14547465 this
        Member [10.38.0.5]:5701 - 6b1e2af2-5160-463a-9a5b-a130962f65d0
```

## What's next:
* Exposing the cluster IP external to K8s network.  Might have to use a HTTP tunnel to alow memcached or native protocol to the cluster.
* Testing: Load, LRU eviction, scale up/down etc.
* Backing store, perhaps Cassandra.
