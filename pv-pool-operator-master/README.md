# PV Pool Operator

## Description
This operator is managing a pool of [pods](https://kubernetes.io/docs/concepts/workloads/pods/), each running a storage agent that can accept reads and writes from\to a persistent volume([PV](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)). In this exercise the pods are running a simulated agent that is "connected" to a central storage system which uses the pool as a storage backing store.  
The PV pool is defined using a `PvPool` CRD.

## Building and Deploying
* All build and deploy commands are defined in the `Makefile`.
* Deploy the operator - this will run an operator [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) in your cluster
    ```
    make deploy IMG=kfirpayne/pv-pool-operator:v1.0.0
    ```

### The operator 
When a creation or update of a PV pool CR is recognized, the pv-pool operator reconcile loop is triggered.

The operator deploys these resources for each pv-pool
* [Service](https://kubernetes.io/docs/concepts/services-networking/service/) - A `ClusterIP` service to expose the storage agents pods
* [Statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) - Manages the storage-agents pods. each pod is mounted with a [PV](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (using a [PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)).)

On each run of the reconcile loop the operator is updating the `Status` of the pv-pool CR (you can look at the CR description with `kubectl get pvpool pvpool-1 -o yaml`)

Once the pv pool is reconciled the Status is set to `Ready`

## Features

1. You can view the used storage percentage of the pool in the `PvPoolStatus`
2. Management of scaling up and scaling down of the pods in the cluster, when scaling down, a decommissioning process is initiated on the appropriate pods, when the decommissioning process finished the pods will terminate.
Scaling down and up cannot happen simultaneously, thus the operator will finish one operation of scaling before stating the second one.
