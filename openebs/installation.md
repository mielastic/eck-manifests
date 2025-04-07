OpenEBS Installation Guide: 
https://openebs.io/docs/user-guides/local-storage-user-guide/local-pv-hostpath/hostpath-installation

**Installation commands:**

```
# helm repo add openebs https://openebs.github.io/openebs
# helm repo update
# helm install openebs --namespace openebs openebs/openebs --set engines.replicated.mayastor.enabled=false --set engines.local.lvm.enabled=false --set engines.local.zfs.enabled=false --set localpv-provisioner.localpv.basePath=/mnt/fast-disks --create-namespace

```

**Verify installation:**

```
# helm ls -n openebs
# kubectl get pods -n openebs

# kubectl get sc

NAME               PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-hostpath   openebs.io/local        Delete          WaitForFirstConsumer   false                  6d2h

```

Once the storage class is available, we can use it in our ECK manifests for Elasticsearch to request volumes on the local disks

**Example manifest:**

```
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart-localdisk
spec:
  podDisruptionBudget: {}
  updateStrategy:
    changeBudget:
      maxUnavailable: 3
  version: 8.17.4
  nodeSets:
  - name: default
    count: 3
    config:
      node.store.allow_mmap: false
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data # Do not change this name unless you set up a volume mount for the data path.
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 50Gi
        storageClassName: openebs-hostpath
```
Applying above manifest will create a 3 node ES cluster, with each node having a 50Gi volume from the local disks
