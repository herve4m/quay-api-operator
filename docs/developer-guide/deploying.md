# Deploying the Operator for Testing

There are several ways to run the Operator in your cluster.
Probably the easiest approach is to use a Deployment resource.
Another approach is to use the Operator Lifecycle Manager (OLM).

Both methods require that you first [build and publish](building.md) the container image for the Operator.


## Log in to Your Cluster as a Cluster Administrator

Both methods use the `kubectl` command to interact with your Kubernetes cluster from your workstation.
Therefore, you need to configure the `~/.kube/config` file to grant you a cluster administrator access to the cluster.
With an OpenShift cluster, you can use the `oc login` command.


## Run the Operator by Using a Deployment Resource

Set the `IMG` environment variable to the name of the image that you built and pushed to the container registry.
The Operator manager container uses that image.

```sh
IMG=quay.io/<some-organization>/quay-api-operator:<tag>
export IMG
make deploy
```

The `make deploy` command installs the Custom Resource Definitions (CRDs), creates the `quay-api-operator-system` Kubernetes namespace, and then creates the Deployment resource that deploys the manager pod.

You can verify that the pod is running by using the `kubectl get pods` command:

```sh
kubectl get pods -n quay-api-operator-system
```

If the pod fails to start, then review its logs by running the `kubectl logs <podname> -n quay-api-operator-system` command.
These logs are also useful later on, for accessing the Ansible messages, and troubleshooting Quay resources.

For more details, review the [Run as a Deployment Inside the Cluster][] section of the [Run the Operator][] guide.

### Undeploy the Operator

To remove the Operator from the cluster, use the `undeploy` target of the `Makefile`:

```sh
IMG=quay.io/<some-organization>/quay-api-operator:<tag>
export IMG
make undeploy
```

The command deletes the manager Deployment resource, the CRDs, and the `quay-api-operator-system`  namespace.


## Run the Operator by Using OLM

OLM consumes the Operator as a bundle container image that you must prepare.

### Bundle the Operator

You create the bundle manifests, and then build and push the image by using the `make` targets.

In the `Makefile`, the `BUNDLE_IMG` variable specifies the name of the bundle image, which is set to `quay.io/herve4m/quay-api-operator-bundle:v$(VERSION)` by default.
Because you do not have write access to that image repository, you must redefine the variable as an environment variable before building the bundle image.
Select a repository for which you have write access.

```sh
BUNDLE_IMG=quay.io/<some-organization>/quay-api-operator-bundle:<tag>
export BUNDLE_IMG
# Remove a previous local build of the bundle image, if it exists
podman rmi ${BUNDLE_IMG}
# Log in to Quay with your credentials
podman login --username <yourname> quay.io
# Build and then push the image
make bundle bundle-build bundle-push
```

### Install OLM

If you use an OpenShift cluster, the OLM is already installed.
Otherwise, use the `operator-sdk` command to install it.

```sh
operator-sdk olm install
```

!!! note
    If the `operator-sdk` command is not available on your system, then the `make bundle` command that you used previously downloads and installs it in the `./bin` directory in the current directory.
    In that case, specify the path to the `operator-sdk` command: `./bin/operator-sdk olm install`

### Deploy the Operator

Create the `quay-api-operator` namespace to host the Operator:

```sh
kubectl create namespace quay-api-operator
```

Use the `operator-sdk run bundle` command to install the Operator in your cluster:

```sh
operator-sdk run bundle -n quay-api-operator ${BUNDLE_IMG}
```

The command create the `CatalogSource`, `OperatorGroup`, `Subscription`, and `ClusterServiceVersion` resources, and then wait for the Operator to complete its deployment.

### Undeploy the Operator

Undeploy the Operator and delete the OLM resources:

```sh
kubectl get Subscription -n quay-api-operator | grep quay-api-operator
kubectl delete Subscription -n quay-api-operator quay-api-operator-<version>

kubectl get ClusterServiceVersion | grep quay-api-operator
kubectl delete ClusterServiceVersion quay-api-operator.<version>

kubectl delete OperatorGroup -n quay-api-operator operator-sdk-og
kubectl delete CatalogSource -n quay-api-operator quay-api-operator-catalog

kubectl delete namespace quay-api-operator
```

For more details, review the [OLM Integration Bundle Tutorial][] and the [Deploy Your Operator with OLM][] section of the [Run the Operator][] guide.

[Run the Operator]: https://sdk.operatorframework.io/docs/building-operators/ansible/tutorial/#run-the-operator
[Run as a Deployment Inside the Cluster]: https://sdk.operatorframework.io/docs/building-operators/ansible/tutorial/#2-run-as-a-deployment-inside-the-cluster
[Deploy Your Operator with OLM]: https://sdk.operatorframework.io/docs/building-operators/ansible/tutorial/#3-deploy-your-operator-with-olm
[OLM Integration Bundle Tutorial]: https://sdk.operatorframework.io/docs/olm-integration/tutorial-bundle/
