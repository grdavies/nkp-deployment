# Nutanix NBD Operator Demo

## Step 1 - Install the NDB operator

1. Open a terminal session.
1. export `KUBECONFIG=PATH_TO_YOUR_KUBECONFIG_FILE`
1. Install the Nutanix NDB Kubernetes Operator `helm install ndb-operator nutanix/ndb-operator -n ndb-operator --create-namespace`.

## Step 2 - Configure your namespace to use the operator

1. Open a terminal session.
1. export `KUBECONFIG=PATH_TO_YOUR_KUBECONFIG_FILE`
1. Set the NDB password in `ndb-secrets.yaml` by setting the `NDB_PASSWORD` placeholder value.
1. Run `kubectl create -f ndb-secrets.yaml --namespace TARGET_NAMESPACE` to create the secret named `ndb-secret-name`.
1. Set the NDB server IP address in `ndb-server.yaml` by setting the `ERA_IP_ADDRESS` placeholder value.
1. Run `kubectl create -f ndb-server.yaml --namespace TARGET_NAMESPACE` to create the NDB server record named `ndb`.

## Step 3 - Deploy a Database via NDB

1. Open a terminal session.
1. export KUBECONFIG=PATH_TO_YOUR_KUBECONFIG_FILE
1. Update the following values in `ndb-db-deploy.yaml`.
    1. Update `YOUR_DATABASE_PASSWORD` with the password for your database.
    1. Update `YOUR_SSH_KEY` with the ssk key to access the database server OS.
1. Run `kubectl create -f ndb-db-deploy.yaml --namespace TARGET_NAMESPACE` to begin the database deployment.
1. You should be able to log into the NDB user interface and see a database deployment begin.

## Notes

See [https://artifacthub.io/packages/helm/nutanix/ndb-operator](https://artifacthub.io/packages/helm/nutanix/ndb-operator) for more details on options that can be used for the different database engines.