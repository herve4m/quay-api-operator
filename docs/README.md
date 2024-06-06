# User Guide

The API Operator for Quay manages Quay Container Registry deployments by using Kubernetes resources.


## Description

The Operator provides Kubernetes custom resources to create, configure, and delete Quay objects.
For example, the Operator can manage Quay organizations, repositories, teams, and user and robot accounts.
It can also configure organization quotas, repository mirroring, and proxy caches.

The operator does not install Quay, which you can install in Kubernetes by using the [Quay Operator](https://operatorhub.io/operator/project-quay), or outside Kubernetes as a standalone deployment.

The [Custom Resources](crds/) section lists the Custom Resource Definitions (CDRs) that the Operator provides, and gives usage examples.

For more details on Quay installation and usage, refer to the [Project Quay Documentation](https://docs.projectquay.io/welcome.html).


## Usage

After you install the Operator in your Kubernetes or Openshift cluster, you can use the CRDs that the Operator installs to manage your Quay instances.

### Connection Parameters

All the Operator's custom resources rely on a Secret resource to provide the connection parameters to the Quay instance to manage.
This Secret resource must include the following data:

* `host`: URL for accessing the Quay API, such as ``https://quay.example.com:8443``.
* `validateCerts`: Whether to allow insecure connections to the API.
  By default, insecure connections are refused.
* `token`: OAuth access token for authenticating against the API.
  To create such a token see the [Creating an OAuth Access Token](https://access.redhat.com/documentation/en-us/red_hat_quay/3/html-single/red_hat_quay_api_guide/index#creating-oauth-access-token) documentation.
  You can also use the [ApiToken](crds/ApiToken.md) custom resource to create this token.
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

You refer to this secret in your Quay custom resources by using the `connSecretRef` property:

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

!!! warning
    Do not delete the Secret resource if a Quay custom resource still references it.
    If you delete the Secret resource, then the Operator cannot connect to the Quay API anymore, and cannot synchronize the Quay custom resource with its corresponding object in Quay.
    In addition, deleting the Quay custom resource does not complete because the Operator cannot delete the corresponding object in Quay.

    If you face this issue, then edit the custom resource (`kubectl edit`), and set the [.spec.preserveInQuayOnDeletion](#deleting-resources) property to `true`.
    Alternatively, you can remove the `.metadata.finalizers` section.
    In both case, you must manually delete the corresponding object in Quay.


### Deleting Resources

By default, when you delete a resource from your Kubernetes cluster, the Operator automatically removes the corresponding object in Quay.
This behavior is controlled by the `preserveInQuayOnDeletion` property of the custom resources.
By setting the property to `true`, you instruct the Operator to keep the object in Quay.

The `Robot` custom resource in the following example manages the `production+robotprod` Quay robot account.
The `preserveInQuayOnDeletion` property is set to `true`, therefore, if you delete the Kubernetes resource, then the Operator does not delete the robot account in Quay.

```yaml
---
apiVersion: quay.herve4m.github.io/v1alpha1
kind: Robot
metadata:
  name: robot-sample
spec:
  connSecretRef:
    name: quay-credentials-secret
    namespace: mynamespace

  # Preserve the robot account in Quay when you delete the resource.
  preserveInQuayOnDeletion: true

  name: production+robotprod
  description: Robot account for production
  retSecretRef:
    name: robot-account-credentials
    namespace: mynamespace
```
