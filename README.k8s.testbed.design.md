# SONiC Kubernetes Testbed Design

This document describes the design to test Kubernetes features in SONiC. SONiC is a worker node managed by a High Availability Kubernetes master in Starlab.

## Topology Overview
![alt text](https://github.com/isabelmsft/k8s-ha-master/blob/master/k8s-testbed-diagram.PNG)

1. Each high availability master setup requires four new Linux KVMs running on Starlab server(s) via bridged networking.
    - KVM Bridged networking in 10.64.246.0/23 subnet allows for connectivity from SONiC DUTs in Starlab. 
2. Kubernetes testing requires 4 additional KVMs: 
    - 3 Linux KVMs to serve as 3-node high availability Kubernetes master
    - 1 Linux KVM to serve as HAProxy Load Balancer node
3. For the initial set up of the high availability master, an Ansible agent is required to run the Ansible jobs to set up and configure the HA functionality through the four Linux KVMs mentioned above. 

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
- Load balancer performance
