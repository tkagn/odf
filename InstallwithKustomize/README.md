# Install ODF with Kustomize

This information is for Internal ODF deployments

## Create directory and files for Kustomize

Create `odf` directory

```bash
mkdir ./odf
cd ./odf
```

Create kustomization file

kustomization.yaml contents:

```yaml
---  
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - odf-operator.yaml
  - odf-storagecluster.yaml
```

Create manifest for creating `openshift-storage` namespace, creating operator group, and adding ODF subscription

odf-operator.yaml contents:

```yaml
# ODF Operator (Namespace + Operator Group + Subscription)
---
# Create openshift-storage namespace.
apiVersion: v1
kind: Namespace
metadata:
  labels:
    openshift.io/cluster-monitoring: "true"
  name: openshift-storage
spec: {}
---
# Create Operator Group for ODF Operator.
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-storage-operatorgroup
  namespace: openshift-storage
spec:
  targetNamespaces:
  - openshift-storage
---
# Subscribe to ODF Operator.
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: odf-subscription
  namespace: openshift-storage
spec:
  channel: "stable-4.9"
  installPlanApproval: Automatic
  name: odf-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

Create storagecluster CR to deploy the ODF cluster

odf-storagecluster.yaml contents:

```yaml
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: odf-storagecluster
  namespace: openshift-storage
spec:
  externalStorage: {}
  storageDeviceSets:
    - config: {}
      count: 1 
      dataPVCTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: "500Gi"
          storageClassName: localblock
          volumeMode: Block
      name: odf-deviceset
      placement: {}
      portable: true
      replica: 3
      resources: {}
```

`storageClassName` should be set the storage class to be used to provision PVCs for ODF volumes.

The typical storage classes are:

- gp2 for GCP
- localblock for baremetal using the local storage operator (LSO)
- managed-premium for Azure
- standard for Openstack




## References

