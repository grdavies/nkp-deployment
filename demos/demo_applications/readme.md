# Application demos

## Online Boutique

1. Open a terminal session.
1. export `KUBECONFIG=PATH_TO_YOUR_KUBECONFIG_FILE`
1. Run `kubectl create -f online_boutique/ --namespace TARGET_NAMESPACE` to deploy the online_boutique demo application.
1. Run `kubectl get services --namespace TARGET_NAMESPACE` to get the external IP for the frontend service.

## Sockshop

**_NOTE:_** This demo is a work in progress.

1. Open a terminal session.
1. export `KUBECONFIG=PATH_TO_YOUR_KUBECONFIG_FILE`
1. Install the Nutanix NDB Kubernetes Operator `helm install ndb-operator nutanix/ndb-operator -n ndb-operator --create-namespace`.
1. Run `kubectl create -f ndb/ --namespace TARGET_NAMESPACE` to configure the workspace NDB configuration for the namespace.
1. Run `kubectl create -f app/ --namespace TARGET_NAMESPACE` to deploy the sockshop application.
1. Run `kubectl get ser -f app/ --namespace TARGET_NAMESPACE` to deploy the sockshop application.
1. Run `kubectl get services --namespace TARGET_NAMESPACE` to get the external IP for the frontend service.
