## Overview
Pods using a Local Persistent Volume are always scheduled to the same Node as the Local PV it uses (as opposed to if they were using a HostPath volume, for instance). When Nodes fail while having a Local PV attached to them, Pods using the Local PV become stuck since they can't be scheduled to a deleted Node. In other words, the Local PV and its corresponding PVC aren’t cleaned up when a Node becomes unavailable and so each of them (PV/PVC/Pods) becomes stuck. This results in a degraded or unavailable workload. This controller aims to clean up the stale Local PVs and their bound PVCs after Node deletion, allowing workloads to automatically recover.

## Local Volume Node Cleanup Controller
The local volume node cleanup controller removes PersistentVolumes and PersistentVolumeClaims that reference deleted Nodes.

## Usage
Please see the example deployment and rbac for deploying the controller.
