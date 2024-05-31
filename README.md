# API Operator for Quay

Manage Quay Container Registry deployments by using Kubernetes resources.

## Description

The API Operator for Quay manages Quay Container Registry components.
The operator does not install Quay, which you can install in Kubernetes by using the [Quay Operator](https://operatorhub.io/operator/project-quay), or outside Kubernetes as a standalone deployment.

The operator provides Kubernetes custom resources to create, configure, and delete Quay objects.
For example, the operator can manage Quay organizations, repositories, teams, and user and robot accounts.
It can also configure organization quotas, repository mirroring, and proxy caches.

### Usage

All the operator's custom resources rely on a Secret resource to provide the connection parameters to the Quay instance.

This secret resource must include the following data:

* `host`: URL for accessing the Quay API, such as ``https://quay.example.com:8443`` for example.
* `validateCerts`: Whether to allow insecure connections to the API.
  By default, insecure connections are refused.
* `token`: OAuth access token for authenticating against the API.
  To create such a token see the [Creating an OAuth Access Token](https://access.redhat.com/documentation/en-us/red_hat_quay/3/html-single/red_hat_quay_api_guide/index#creating-oauth-access-token) documentation.
* `username`: The username to use for authenticating against the API.
  If `token` is set, then `username` is ignored.
* `password`: The password to use for authenticating against the API.
  If `token` is set, then `password` is ignored.

You can create the secret by using the `kubectl create secret` command:

```sh
kubectl create secret generic quay-credentials --from-literal host=https://quay.example.com:8443 --from-literal validateCerts=false --from-literal token=vFYyU2D0fHYXvcA3Y5TYfMrIMyVIH9YmxoVLsmku
```

Or you can create the secret from a resource file:

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: quay-credentials
stringData:
  host: https://quay.example.com:8443
  validateCerts: "false"
  token: vFYyU2D0fHYXvcA3Y5TYfMrIMyVIH9YmxoVLsmku
```

You refer to this secret in your Quay custom resources by the using the `connSecretRef` option:

```yaml
---
apiVersion: quay.herve4m.github.io/v1alpha1
kind: Organization
metadata:
  name: organization-sample
spec:
  # Connection parameters in a Secret resource
  connSecretRef:
    name: quay-credentials
    # By default, the operator looks for the secret in the same namespace as
    # the organization resource, but you can specify a different namespace.
    # namespace: mynamespace

  name: production
  email: prodlist@example.com
  timeMachineExpiration: 7d
  autoPruneMethod: tags
  autoPruneValue: "20"
  preserveInQuayOnDeletion: false
```

More examples are available on [GitHub](https://github.com/herve4m/quay-api-operator/tree/main/config/samples).


## Getting Started

### Prerequisites

- podman version 4.9.4+.
- kubectl version v1.27.0+.
- Access to a Kubernetes v1.27.0+ cluster.

### Build and Push the Controller Container Image

Build and push your image to the location specified by the `IMG` environment variable:

```sh
IMG=quay.io/<some-registry>/quay-api-operator:<tag>
export IMG
# Remove a previous local build of the image
podman rmi ${IMG}
# Log in to Quay with your credentials
podman login --username <yourname> quay.io
# Build and then push the image
make docker-build docker-push
```

**NOTE:** This image ought to be published in the personal registry you specified.
And it is required to have access to pull the image from the working environment.
Make sure you have the proper permission to the registry if the above commands don’t work.

### Deploy the Operator on the Cluster

Deploy the Manager to the cluster with the image specified by the `IMG` environment variable:

```sh
IMG=quay.io/<some-registry>/quay-api-operator:<tag>
export IMG
make deploy
```

> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin privileges or be logged in as admin.

The `make deploy` command creates the `quay-api-operator-system` Kubernetes namespace, and then deploys the manager pod in this namespace.
You can verify that the pod is running by using the `kubectl get pods` command:

```sh
kubectl get pods -n quay-api-operator-system
```

You can access the logs of the pod by using the `kubectl logs <podname> -n quay-api-operator-system` command.
These logs are useful for reviewing the Ansible messages when a custom resource fails.


**Create instances of your solution**
You can apply the samples (examples) from the config/sample:

```sh
kubectl apply -k config/samples/
```

>**NOTE**: Ensure that the samples has default values to test it out.

### To Uninstall
**Delete the instances (CRs) from the cluster:**

```sh
kubectl delete -k config/samples/
```

**Delete the APIs(CRDs) from the cluster:**

```sh
make uninstall
```

**UnDeploy the controller from the cluster:**

```sh
make undeploy
```

**NOTE:** Run `make help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)


## Contributing

We welcome community contributions to this operator.
If you find problems, then please open an [issue](https://github.com/herve4m/quay-api-operator/issues) or create a [pull request](https://github.com/herve4m/quay-api-operator/pulls).

More information about contributing can be found in the [Contribution Guidelines](https://github.com/herve4m/quay-api-operator/blob/main/CONTRIBUTING.md).


## License

Copyright 2024.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
