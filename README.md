
# Nutanix NKP Quickstart guide
This is a quickstart guide to get up & running with NKP in a non-airgapped environment.

## Pre-Requisites

- Internet connectivity
- Access to the Nutanix Support portal
- Existing Nutanix NCI / NCM deployment

## Table of Contents

- [Step 1: Create Linux Jump Host](#step-1-create-linux-jump-host)
- [Step 2: Deploy NKP](#step-2-deploy-nkp)
- [Appendix 1: NKP CLI v2.12.1 CLI Flags](#appendix-1-nkp-cli-v2121-flags-for-nutanix-deployment)
- [Appendix 2: Example Export & Installation Script](#appendix-2-example-nkp-cli-export--execution)

## Step 1: Create Linux Jump Host

### Step 1a: Upload Rocky image from Nutanix support portal

From the NKP product page on the [Nutanix Download Site](https://portal.nutanix.com/page/downloads?product=nkp), download the latest version of the pre-packaged image file `NKP Node OS Image (Rocky Linux) for AHV`. Deploy this image to your Nutanix infrastructure following [this guide](https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_10:wc-image-configure-acropolis-wc-t.html) from the Nutanix support portal.

### Step 1b: Build Jump VM

1. Once the image has uploaded, create a new VM using the image.
    - Set the vCPU count to 4.
    - Set RAM to 8GB.
    - Choose to "Clone from Image Service" and select the `NKP Node OS Image (Rocky Linux) for AHV` image uploaded previously and resize the new disk to an appropriate size (I chose 200GB).
    - Select to customize using a "custom script" and use the `cloud-init` script below. Replace the password defined on line 14 with one of your own choosing.

    **_NOTE:_** If you do not know how to create a VM consult the documentation for either [Prism Central](https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2024_2:mul-vm-create-acropolis-pc-t.html) or [AHV](https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_10:wc-vm-create-acropolis-wc-t.html).


    ```yaml
    #cloud-config
    users:
      - name: nutanix
        shell: /bin/bash
        sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_pwauth: True
    package_upgrade: true
    packages:
      - bind-utils
    chpasswd:
      expire: False
      users:
      - name: nutanix
        password: nutanix/4u # Recommended to change the password or update the script to use SSH keys
        type: text 
    bootcmd:
      - mkdir -p /etc/docker
    runcmd:
      - '[ ! -f "/etc/yum.repos.d/nutanix_rocky9.repo" ] || mv -f /etc/yum.repos.d/nutanix_rocky9.repo /etc/yum.repos.d/nutanix_rocky9.repo.disabled'
      - dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
      - dnf -y install docker-ce docker-ce-cli containerd.io
      - systemctl --now enable docker
      - usermod -aG docker nutanix
      - 'curl -Lo /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
      - chmod +x /usr/local/bin/kubectl
      - 'curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash'
      - eject
      - 'wall "If you are seeing this message, please reconnect your SSH session. Otherwise, the NKP CLI installation process may fail."'
    final_message: "The machine is ready after $UPTIME seconds. Go ahead and install the NKP CLI using: $ curl -sL https://raw.githubusercontent.com/nutanixdev/nkp-quickstart/main/scripts/get-nkp-cli | bash"
    growpart:
      mode: auto
      devices: ['/']
      ignore_growroot_disabled: false
    output:
      init:
        output: "> /var/log/cloud-init.out"
        error: "> /var/log/cloud-init.err"
      config: "tee -a /var/log/cloud-config.log"
      final:
        - ">> /var/log/cloud-final.out"
        - "/var/log/cloud-final.err"
    ```

3. Open an SSH session to the newly created VM using the `nutanix` user account and the password set in line 14 of the `cloud-init` script. 

    **_NOTE:_** After 10-15 minutes the VM should be ready for use. 
        If you logged onto the VM console you should see a message like this `If you are seeing this message, please reconnect your SSH session.` If not, check `/var/log/cloud-final.out` for this message to continue.

4. From the [Nutanix Download Site](https://portal.nutanix.com/page/downloads/list) and find the `NKP binary for Linux OS` download. Click on the three dots to the right of the `Download` button and choose to `Copy Download Link`. Save this link for use with the script below.

    **_NOTE:_** The copied link is only valid for 10 hours. 

    Install the NKP CLI by running the following command. When prompted provide the previously copied link.

    ```bash
    curl -fsSL https://raw.githubusercontent.com/nutanixdev/nkp-quickstart/main/scripts/get-nkp-cli | bash
    ```


## Step 2: Deploy NKP

During this step you will provide all the variables necessary for the deployment of your NKP management cluster and initiate the deployment. There are two options which you can use for the installation; [Prompt-Based Installation](#step-2a-prompt-based-installation) or [CLI Installation](#step-2b-cli-installation).

- The Prompt-Based Installation is a wizard driven installation method. This method is simplier and should be used when the Internet connection for the NKP cluster isn’t shared with more users. 
- The CLI Installation is a little more involved and allows for more customization. If advanced NCI features like `Network Virtualization` are in use this method is required. It should also be selected when the Internet connection for the NKP cluster is shared between many users.

For what its worth, its beneficial getting familiar with the [CLI Installation](#step-2b-cli-installation) method as its significantly more repeatable and flexible. 

### Step 2a: Prompt-Based Installation

this installation method give less control over the cluster configuration. You cannot change the cluster VM count, VM size (vCPU/RAM) nor can you select advanced components such as `Overlay Networks`.

Run the following command to load the wizard. Provide all inputs when prompted.

```bash
nkp create cluster nutanix
```

### Step 2b: CLI Installation

This installation method lets you fully customize your cluster configuration.

1. To get a full list of all available options you can run the following command. I have included the list of available options in the appendix [here](#appendix-1-nkp-cli-v2121-options-for-nutanix-deployment):

    ```bash
    nkp create cluster nutanix -h
    ```

2. In a text editor create a new document that will become a script containing a list of exports to hold the data for each flag that you wish to set. I have included an example [here](#appendix-2-export-script).

    ```bash
    export YOUR_NAME_FOR_FLAG=your_setting_for_flag
    ```

    **_NOTE:_** You may have to encapsulate your settings with apostrophes `''` if they contain special characters - for example your passwords likely meet this requirement.

3. In a text editor create a new document that will become the installation script for NKP. This will contain the execution of the NKP CLI using the variables that were previously set. I have included an example [here](#appendix-2-installation-script).

    - If your Nutanix Prism Central is not using trusted certificates be sure to provide the `--insecure` flag.
    - You must provide either `--dry-run` or `--self-managed`.
      - `--dry-run` will run through the deployment process but will not make any changes.
      - `--self-managed` will deploy your first NKP cluster.
    - `--control-plane-endpoint-ip` will be the IP address that can be used to access the NKP UI after installation.
    - The default Kubernetes Pod Network CIDR is 192.168.0.0/16. Provide a new range using the flag `--kubernetes-pod-network-cidr` if that will cause a conflict on your network.
    - If you wish to place your NKP cluster VMs into a Prism Central Project provide the `--control-plane-pc-project` and/or the `--worker-pc-project` flags.
    - You can control both the number of VMs deployed for the control plane and worker node pool and their sizes for this NKP cluster. Be careful changing these values as there both unexpected behavior and system performance can occur. **_NOTE:_** Do not change these values without guidance from Nutanix for a production NKP deployment.
      - `--control-plane-replicas` sets the number of control plane nodes (default 3)
      - `--control-plane-vcpus` sets the number of vCPUs to use in a control plane machine (default 4)
      - `--control-plane-memory` sets the size of memory (in GiB) of a control plane machine (default 16)
      - `--worker-replicas` sets the number of workers (default 4)
      - `--worker-vcpus` sets the number of vCPUs to use in a worker machine (default 8)
      - `--worker-memory` sets the size of memory (in GiB) of a worker machine (default 32)

4. Once you have prepared both the list of exports and the installation script it is time to execute them :).
    1. Copy the export script, paste it into your SSH session and execute it on your jump host. Use the `env` command to ensure that they have been set correctly.
    2. Run `nkp create bootstrap` to build the local bootstrap environment.
    3. Copy the installation script, paste it into your SSH session and execute it. If you want to test the process be sure to set `--dry-run` instead of `--self-managed`.
    4. Process of the deployment will be output to the screen. Once the deployment has completed a kubectl conf file will be created in the directory that the installation command was executed from. Run `nkp get dashboard --kubeconfig="/home/nutanix/YOUR-CONF-FILE-NAME.conf"` to get the initial administrator username and password. 
    5. The UI should now be available at the IP address specified in the flag `--control-plane-endpoint-ip`. Open this IP address in a browser and logon using the initial administrator username and password.

Next steps:

- Apply licensing

## Appendix

### Appendix 1: NKP CLI v2.12.1 Flags for Nutanix Deployment

```bash
Create a Konvoy cluster in Nutanix

Usage:
  nkp create cluster nutanix [flags]

Flags:
      --acme-email string                                  Email address the ACME server can use to contact you.
      --acme-server string                                 Address of the ACME service issuing the certificates (default: Let's encrypt). (default "https://acme-v02.api.letsencrypt.org/directory")
      --additional-trust-bundle string                     Additional CA trust bundle to use to validate the Prism Central server certificate.
      --airgapped                                          Enable airgapped mode.
      --allow-missing-template-keys                        If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to golang and jsonpath output formats. (defaultue)
      --aws-service-endpoints string                       Custom AWS service endpoints in a semi-colon separated format: ${SigningRegion1}:${ServiceID1}=${URL},${ServiceID2}=${URL};${SigningRegion2}...
      --certificate-renew-interval int                     The interval number of days Kubernetes managed PKI certificates are renewed. For example, an Interval value of 30 means the certificates will be refreshevery 30 days. A value of 0 disables the feature. (default 0)
      --cluster-class string                               The name of the ClusterClass to create a cluster from (default "nkp-nutanix")
      --cluster-hostname string                            Hostname that is used for accessing the cluster's ingresses.
  -c, --cluster-name name                                  Name used to prefix the cluster and all the created resources.
      --control-plane-cores-per-vcpu int32                 The number of cores per vCPU(equivalent to CPU cores) to use in a control plane machine (default 1)
      --control-plane-disk-size int32                      The size of the primary disk (in GiB) of a control plane machine (default 80)
      --control-plane-endpoint-ip ip                       The control plane endpoint address. To use an external load balancer, set to its IP or hostname. To use the built-in virtual IP, set to a static IPv4 adss in the Layer 2 network of the control plane machines. [Not for production use: To use a single-machine control plane, set to the IP or hostname of the machine.]
      --control-plane-endpoint-port int32                  The control plane endpoint port. To use an external load balancer, set to its listening port. (default 6443)
      --control-plane-memory int32                         The size of memory (in GiB) of a control plane machine (default 16)
      --control-plane-pc-categories strings                Names of Prism Central categories to associate with control plane resources (VMs, VGs, etc). Example: key1=value1,key1=value2,key2=value2 (default [])
      --control-plane-pc-project string                    Name of Prism Central project to associate with control plane resources (VMs, VGs, etc).
      --control-plane-prism-element-cluster string         Name of the Prism Element cluster to use to create a control plane machine
      --control-plane-replicas int                         Number of control plane nodes (default 3)
      --control-plane-subnets strings                      Names of Prism Central subnets to use for control plane machines. Example: subnet1,subnet2,subnet3 (default [])
      --control-plane-vcpus int32                          The number of vCPUs(equivalent to CPU sockets) to use in a control plane machine (default 4)
      --control-plane-vm-image string                      Name of OS image to use for control plane machines.
      --csi-file-system string                             File system to use for CSI volumes. Allowed values ["ext4" "xfs"]. (default "ext4")
      --csi-flash-mode                                     If true, will enable flash mode for CSI volumes.
      --csi-hypervisor-attached-volumes                    If true, will enable the hypervisor attached feature for CSI volumes which allows disks to attach directly to the host without using iSCSI. (default tru
      --csi-reclaim-policy string                          Reclaim policy for CSI volumes. Allowed values ["Delete" "Retain"]. (default "Delete")
      --csi-storage-container string                       Name of the Prism Central storage container to associate with the storage class created on the cluster.
      --dry-run                                            Only print the objects that would be created, without creating them.
      --endpoint url                                       Prism Central URL. Accepted formats: host, host:port, http[s]://host[:port]. Accepted host formats: IP, FQDN.
      --etcd-image-repository string                       The image repository to use for pulling the etcd image
      --etcd-version string                                The version of etcd to use.
      --extra-sans strings                                 A comma separated list of additional Subject Alternative Names for the API Server signing cert (default [])
  -h, --help                                               Help for nutanix
      --http-proxy string                                  HTTP proxy for all nodes in the cluster
      --https-proxy string                                 HTTPS proxy for all nodes in the cluster
      --ingress-ca file                                    Path to file containing the certificate's CA bundle.
      --ingress-certificate file                           Path to file containing certificates for configuring Ingress.
      --ingress-private-key file                           Path to file containing the certificate's private key (PEM).
      --insecure                                           If true, the Prism Central server certificate will not be validated.
      --kind-cluster-image string                          Kind node image for the bootstrap cluster (default "mesosphere/konvoy-bootstrap:v2.12.0")
      --kubeconfig string                                  Path to the kubeconfig for the management cluster. If unspecified, default discovery rules apply. This flag is ignored if used with the --self-managed f.
      --kubernetes-image-repository string                 The image repository to use for pulling kubernetes images
      --kubernetes-pod-network-cidr cidr                   The Kubernetes Pod network CIDR to use in the cluster (default 192.168.0.0/16)
      --kubernetes-service-cidr cidr                       The Kubernetes Service CIDR to use in the cluster (default 10.96.0.0/12)
      --kubernetes-service-load-balancer-ip-range string   A hyphen separated IP range to configure the Kubernetes Service Load Balancer provider with. Example: 10.0.0.0-10.0.0.10
      --kubernetes-version string                          Kubernetes version (default "1.29.6")
  -n, --namespace string                                   If present, the namespace scope for this CLI request. (default "default")
      --no-proxy strings                                   No Proxy list for all nodes in the cluster (default [])
  -o, --output string                                      Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath, jsonpath-as-json, jsonpath-file).
      --output-directory string                            Used with --output=json|yaml. The directory where to output resources to files. The directory must already exist.
      --registry-cacert file                               Path to file containinng the CA certificate used to verify the registry server certificate
      --registry-mirror-cacert file                        Path to file containing the CA certificate used to verify the registry mirror server certificate
      --registry-mirror-password string                    Password used to authenticate with the registry mirror
      --registry-mirror-url url                            URL of a container registry used as a mirror
      --registry-mirror-username string                    Username used to authenticate with the registry mirror
      --registry-password string                           Password used to authenticate with the registry
      --registry-url url                                   URL of a container registry
      --registry-username string                           Username used to authenticate with the registry
      --self-managed                                       When set to true, the required prerequisites are created before creating the cluster and the resulting cluster has all necessary components deployed onttself, so it can manage its own cluster lifecycle. When set to false, a management cluster is used. (default false)
      --show-managed-fields                                If true, keep the managedFields when printing objects in JSON or YAML format.
      --ssh-public-key-file string                         Path to the authorized SSH key for the user
      --ssh-username string                                Name of the user to create on the instance (default "konvoy")
      --template string                                    Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format is golang templates [http://golang.org/pkgxt/template/#pkg-overview].
      --timeout duration                                   The length of time to wait before giving up. Zero means wait forever (e.g. 300s, 30m, 3h). (default 30m0s)
      --vm-image string                                    Name of OS image to use for all machines.
      --wait                                               If true, wait for operations to complete before returning. This flag is ignored and will always be 'true' if used with the --self-managed flag. (defaultue)
      --with-aws-bootstrap-credentials                     Set true to use AWS bootstrap credentials from your environment. When false, the instance profile of the EC2 instance where the CAPA controller is schedd on will be used instead.
      --with-gcp-bootstrap-credentials                     Set true to use GCP bootstrap credentials from your environment. When false, the service account of the VM instance where the CAPG controller is schedulon will be used instead.
      --worker-cores-per-vcpu int32                        The number of cores per vCPU(equivalent to CPU cores) to use in a worker machine (default 1)
      --worker-disk-size int32                             The size of the primary disk (in GiB) of a worker machine (default 80)
      --worker-memory int32                                The size of memory (in GiB) of a worker machine (default 32)
      --worker-pc-categories strings                       Names of Prism Central categories to associate with worker resources (VMs, VGs, etc). Example: key1=value1,key1=value2,key2=value2 (default [])
      --worker-pc-project string                           Name of Prism Central project to associate with worker resources (VMs, VGs, etc).
      --worker-prism-element-cluster string                Name of the Prism Element cluster to use to create a worker machine
      --worker-replicas int                                Number of workers (default 4)
      --worker-subnets strings                             Names of Prism Central subnets to use for worker machines. Example: subnet1,subnet2,subnet3 (default [])
      --worker-vcpus int32                                 The number of vCPUs(equivalent to CPU sockets) to use in a worker machine (default 8)
      --worker-vm-image string                             Name of OS image to use for worker machines.

Global Flags:
  -v, --verbose int   Output verbosity
```

### Appendix 2: Example NKP CLI export & execution

#### Appendix 2: Export Script

```bash
export NKP_VERSION=2.12.1                                                                     # NKP version to install
export CONTROL_PLANE_REPLICAS=3                                                               # Number of control plane VMs
export WORKER_REPLICAS=4                                                                      # Number of worker VMs
export CLUSTER_NAME=nkp                                                                       # NKP cluster name. When using NKP Pro/Ultimate, this name is used to generate the license key
export NUTANIX_USER=admin                                                                     # UPDATE with Prism Central user username
export NUTANIX_PASSWORD='password'                                                            # UPDATE with Prism Central user's password. Keep the password enclosed between single quotes - Ex: 'password'
export NUTANIX_ENDPOINT=10.54.80.39                                                           # UPDATE with Prism Central IP address
export NUTANIX_PORT=9440                                                                      # Prism Central port (default: 9440)
export LB_IP_RANGE=10.54.80.11-10.54.80.11                                                    # UPDATE with the External Load balancer IP range. Must not be in AHV IPAM. Ex: 10.42.236.204-10.42.236.204
export CONTROL_PLANE_ENDPOINT_IP=10.54.80.10                                                  # UPDATE with the Kubernetes VIP. Must be in the same subnet as the VMs. Must not be in AHV IPAM. - Ex: 10.42.236.203
export NUTANIX_MACHINE_TEMPLATE_IMAGE_NAME=nkp-rocky-9.4-release-1.29.6-20240816215147.qcow2  # UPDATE with the NKP Rocky image name
export NUTANIX_PRISM_ELEMENT_CLUSTER_NAME=CLUSTER1234                                         # UPDATE with Prism Element cluster name - Ex: PHX-POC207
export NUTANIX_SUBNET_NAME=primary                                                            # UPDATE with the subnet name on which to deploy the NKP cluster
export NUTANIX_STORAGE_CONTAINER_NAME=SelfServiceContainer                                    # UPDATE with the Prism storage container on which to deploy the NKP cluster
export REGISTRY_URL=[https://registry-1.docker.io](https://registry-1.docker.io/)
export REGISTRY_USERNAME=dockerhub_user                                                       # UPDATE with your dockerhub username
export REGISTRY_PASSWORD='dockerhub_password'                                                 # UPDATE with your dockerhub password
```

#### Appendix 2: Dry Run Script

```bash
nkp create cluster nutanix -c $CLUSTER_NAME \
    --endpoint https://$NUTANIX_ENDPOINT:$NUTANIX_PORT \
    --insecure \
    --cluster-name $CLUSTER_NAME \
    --kubernetes-service-load-balancer-ip-range $LB_IP_RANGE \
    --control-plane-endpoint-ip $CONTROL_PLANE_ENDPOINT_IP \
    --control-plane-vm-image $NUTANIX_MACHINE_TEMPLATE_IMAGE_NAME \
    --control-plane-prism-element-cluster $NUTANIX_PRISM_ELEMENT_CLUSTER_NAME \
    --control-plane-subnets $NUTANIX_SUBNET_NAME \
    --control-plane-replicas $CONTROL_PLANE_REPLICAS \
    --worker-vm-image $NUTANIX_MACHINE_TEMPLATE_IMAGE_NAME \
    --worker-prism-element-cluster $NUTANIX_PRISM_ELEMENT_CLUSTER_NAME \
    --worker-subnets $NUTANIX_SUBNET_NAME \
    --worker-replicas $WORKER_REPLICAS \
    --csi-storage-container $NUTANIX_STORAGE_CONTAINER_NAME \
    --registry-url $REGISTRY_URL \
    --registry-username $REGISTRY_USERNAME \
    --registry-password $REGISTRY_PASSWORD \
    --dry-run
```

#### Appendix 2: Installation Script

```bash
nkp create cluster nutanix -c $CLUSTER_NAME \
    --endpoint https://$NUTANIX_ENDPOINT:$NUTANIX_PORT \
    --insecure \
    --cluster-name $CLUSTER_NAME \
    --kubernetes-service-load-balancer-ip-range $LB_IP_RANGE \
    --control-plane-endpoint-ip $CONTROL_PLANE_ENDPOINT_IP \
    --control-plane-vm-image $NUTANIX_MACHINE_TEMPLATE_IMAGE_NAME \
    --control-plane-prism-element-cluster $NUTANIX_PRISM_ELEMENT_CLUSTER_NAME \
    --control-plane-subnets $NUTANIX_SUBNET_NAME \
    --control-plane-replicas $CONTROL_PLANE_REPLICAS \
    --worker-vm-image $NUTANIX_MACHINE_TEMPLATE_IMAGE_NAME \
    --worker-prism-element-cluster $NUTANIX_PRISM_ELEMENT_CLUSTER_NAME \
    --worker-subnets $NUTANIX_SUBNET_NAME \
    --worker-replicas $WORKER_REPLICAS \
    --csi-storage-container $NUTANIX_STORAGE_CONTAINER_NAME \
    --registry-url $REGISTRY_URL \
    --registry-username $REGISTRY_USERNAME \
    --registry-password $REGISTRY_PASSWORD \
    --self-managed
```