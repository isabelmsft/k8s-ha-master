# SONiC Kubernetes Testbed Design

This document describes the design to test Kubernetes features in SONiC. 

## Background

Each SONiC DUT is a worker node managed by a High Availability Kubernetes master in Starlab. 

By connecting each SONiC DUT to HA Kubernetes master, containers running in SONiC can be managed by the Kubernetes master. SONiC containers managed by the Kubernetes master are termed to be running in "Kubernetes mode" as opposed to the original "Local mode." 

In Kubernetes mode, SONiC container properties are based on specifications defined in the associated Kubernetes manifest. A Kubernetes manifest is a file in the Kubernetes master that defines the Kubernetes object and container configurations. In our case, we use Kubernetes Daemonset objects. The Kubernetes Daemonset object ensures that each worker node is running one container of the image specified in the Daemonset manifest file.  

For example, in order to run Analytic and Telemetry containers in Kubernetes mode, we must have two manifests that define two Kubernetes Daemonset objects- one for each container running in "Kubernetes mode." 

The following is a snippet of the Telemetry Daemonset manifest file that specifies the Kubernetes object type and container image:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: telemetry-ds
spec:
  template:
    metadata:
      labels:
        name: telemetry
    spec:
      hostname: sonic
      hostNetwork: true
      containers:
      - name: telemetry
        image: sonicanalytics.azurecr.io/sonic-dockers/any/docker-sonic-telemetry:20200531
        tty: true
            .
            .
            .
```


## Topology Overview

In order to connect each SONiC DUT to a High Availability Kubernetes master, we need to set up the following topology: 
![alt text](https://github.com/isabelmsft/k8s-ha-master/blob/master/k8s-testbed-diagram.PNG)

- Each high availability master setup requires four new Linux KVMs running on Starlab server(s) via bridged networking.
    - KVM Bridged networking in 10.64.246.0/23 subnet allows for connectivity from SONiC DUTs in Starlab. 
- Kubernetes testing requires 4 additional KVMs: 
    - 3 Linux KVMs to serve as 3-node high availability Kubernetes master
    - 1 Linux KVM to serve as HAProxy Load Balancer node
- For the initial set up of the high availability master, an Ansible agent is required to run the Ansible jobs to set up and configure the HA functionality through the four Linux KVMs mentioned above. 

## We aim to test the following functionalities: 
- Successful deployment of SONiC containers via manifests defined in master
- Automatic recreation of container after being (intentionally or accidentally) stopped
- Switching between Local and Kubernetes management mode

During each of the following states:
- When all masters are up and running
- When one master is down
- When two masters are down
- When all masters are down

Down: shut off, disconnected, or in the middle of reboot

In this setup, we do not consider:
- Load balancer performance. In our case, HAProxy is configured to perform vanilla round-robin load balancing with health checking.

In order to perform tests, the client must have the ability to request 
The preexisting way to do this is 
