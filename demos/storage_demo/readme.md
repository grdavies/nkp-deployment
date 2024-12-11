# Storage Demos

## Volumes

1. The Volumes storage class is enabled by default in a NKP deployment. 
1. You can create a Volume claim using the example `csi-nutanix-volumes-claim-test.yaml`.

**_NOTE:_** For more detail refer to the [CSI Volume Driver 3.1
Documentation](https://portal.nutanix.com/page/documents/details?targetId=CSI-Volume-Driver-v3_1:CSI-Volume-Driver-v3_1).

## NFS

1. Update the following values in `csi-nutanix-nfs.yaml`.
    1. `key` `PC_IP_ADDRESS` should be set to the Prism Central IP address
    1. `key` `PC_ADMIN_PASSWORD` should be set to the Prism Central IP admin user account password.
    1. `files-key` `FILE_SERVER_FQDN` should be set to FQDN of the Files server in the format. Eg. `files1.domain.local`.
    1. `files-key` `PC_ADMIN_PASSWORD` should be set to the Prism Central IP admin user account password.
    1. `nfsServerName` `FILE_SERVER_NAME` should be set to the name of the Files server instance. Eg. `Files1`.
1. Run `kubectl create -f csi-nutanix-nfs.yaml --namespace ntnx-system` to create the NFS dynamic storage class `nutanix-dynamicfile`.
1. Now you can create dynamic NFS shares like that contained in the example `csi-nutanix-nfs-claim-test.yaml`.

**_NOTE:_** For more detail refer to the [CSI Volume Driver 3.1
Documentation](https://portal.nutanix.com/page/documents/details?targetId=CSI-Volume-Driver-v3_1:CSI-Volume-Driver-v3_1).

## Objects

1. Update the following variables in `demos/storage_demo/cosi/charts/values.yaml`.
    1. `endpoint` `OBJECT_ENDPOINT_IP` should be set to the Prism Central IP address.
    1. In the objects UI click on `Access Keys`, then `+ Add People`. Choose `Add people not in a directory service` and enter an email and name, then click `Next` then `Generate Keys`.
    1. `access_key` `OBJECT_ENDPOINT_ACCESS_KEY` set to the user access key generated in step ii.
    1. `secret_key` `OBJECT_ENDPOINT_SECRET_KEY` set to the user secret key generated in step ii.
    1. `pc_ip` `PC_IP_ADDRESS` should be set to the Prism Central IP address.
    1. `pc_password` `PC_ADMIN_PASSWORD` should be set to the Prism Central IP admin account password.
1. Change to the `demos/storage_demo/cosi/charts` directory.
1. Run `helm install cosi-driver -n cosi-driver-nutanix -f values.yaml .` to install the COSI driver for Nutanix Objects.
1. Run `kubectl create -f demos/storage_demo/cosi/project/examples/bucketclass.yaml --namespace ntnx-system` to create the Nutanix Objects storage class `sample-bucketclass`. Update any names/values necessary.
1. Use `demos/storage_demo/cosi/project/examples/bucketclaim.yaml` to create a s3 claim.

**_NOTE:_** For more detail refer to [https://github.com/nutanix-cloud-native](https://github.com/nutanix-cloud-native).