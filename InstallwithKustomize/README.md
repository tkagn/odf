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

- gp2 for AWS. [Deploying OpenShift Data Foundation using Amazon Web Services](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_openshift_data_foundation_using_amazon_web_services/deploy-using-dynamic-storage-devices-aws#creating-an-openshift-data-foundation-service_cloud-storage)
- localblock for baremetal using the local storage operator (LSO) [Deploying OpenShift Data Foundation using bare metal infrastructure](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_openshift_data_foundation_using_bare_metal_infrastructure/deploy-using-local-storage-devices-bm#creating-openshift-data-foundation-cluster-on-bare-metal_local-bare-metal)
- thin on VMware - [Deploying OpenShift Data Foundation on VMware vSphere](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_openshift_data_foundation_using_amazon_web_services/deploy-using-dynamic-storage-devices-aws#creating-an-openshift-data-foundation-service_cloud-storage)
- managed-premium for Azure [Deploying OpenShift Data Foundation using Microsoft Azure and Azure Red Hat OpenShift](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_openshift_data_foundation_using_microsoft_azure_and_azure_red_hat_openshift/deploying-openshift-data-foundation-on-microsoft-azure_azure#creating-an-openshift-data-foundation-service_azure)
- standard for Openstack [Deploying and managing OpenShift Data Foundation using Red Hat OpenStack Platform](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_and_managing_openshift_data_foundation_using_red_hat_openstack_platform/deploying_openshift_data_foundation_on_red_hat_openstack_platform_in_internal_mode#creating-an-openshift-data-foundation-service_internal-osp)
- hdd or new storage class for GCP - [Deploying and managing OpenShift Data Foundation using Google Cloud](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_and_managing_openshift_data_foundation_using_google_cloud/deploying_openshift_data_foundation_on_google_cloud#creating-an-openshift-data-foundation-service_gcp)
- localblock storage clase from the local storage operaotr  - [Deploying OpenShift Data Foundation using IBM Power Systems](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_openshift_data_foundation_using_ibm_power_systems/deploy-using-local-storage-devices-ibm-power#creating-openshift-data-foundation-cluster-on-ibm-power_local-ibm-power)
- storage class for RHV - [Deploying OpenShift Data Foundation using Red Hat Virtualization platform](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_openshift_data_foundation_using_red_hat_virtualization_platform/deploy-using-dynamic-storage-devices-rhv#creating-an-openshift-data-foundation-service_dynamicrhv-storage)
- hdd or new storage class for GCP - [Deploying OpenShift Data Foundation using IBM Cloud](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_openshift_data_foundation_using_ibm_cloud)
- localblock or new storage class for GCP - [Deploying OpenShift Data Foundation using IBM Z infrastructure](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9/html/deploying_openshift_data_foundation_using_ibm_z_infrastructure/deploy-using-local-storage-devices-ibmz#creating-openshift-data-foundation-cluster-on-ibmz_ibmz)

## Testing Manifest

The `oc` tool and `--dry-run` options can be used to verify the generated manifests of Kustomize yamls

```bash
oc apply -k . -o yaml --dry-run=client
```

Review output and verify what will be added to the cluster.

## Applying Manifest to Cluster

```bash
oc apply -k .
```

## References

- [Product Documentation for Red Hat OpenShift Data Foundation 4.9](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.9)
- [Kubernetes native configuration management](https://kustomize.io/)
