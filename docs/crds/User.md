<!---
DO NOT EDIT THIS FILE, BECAUSE IT IS AUTOMATICALLY GENERATED FROM A TEMPLATE.
Instead, edit the scripts/templates/docs/api.md.jinja template, and then run
the scripts/crd2markdown script.
--->

# User - Manage Quay Container Registry users

The User custom resource relies on a Secret resource to provide the connection parameters to the Quay instance.
This Secret resource must include the following data:

* `host`: URL for accessing the Quay API, such as ``https://quay.example.com:8443`` for example.
* `validateCerts`: Whether to allow insecure connections to the API.
  By default, insecure connections are refused.
* `token`: OAuth access token for authenticating against the API.
  To create such a token see the [Creating an OAuth Access Token](https://access.redhat.com/documentation/en-us/red_hat_quay/3/html-single/red_hat_quay_api_guide/index#creating-oauth-access-token) documentation.
  You can also use the [ApiToken](ApiToken.md) custom resource to create this token.
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

You refer to this secret in your User custom resource by using the `connSecretRef` property.
See the [usage example](#usage-example).

!!! warning
    Do not delete the Secret resource if a Quay custom resource still references it.
    If you delete the Secret resource, then the Operator cannot connect to the Quay API anymore, and cannot synchronize the Quay custom resource with its corresponding object in Quay.
    In addition, deleting the Quay custom resource does not complete because the Operator cannot delete the corresponding object in Quay.

    If you face this issue, then edit the custom resource (`kubectl edit`), and set the [.spec.preserveInQuayOnDeletion](#preserveinquayondeletion) property to `true`.
    Alternatively, you can remove the `.metadata.finalizers` section.
    In both case, you must manually delete the corresponding object in Quay.


## Usage Example

```yaml
---
apiVersion: quay.herve4m.github.io/v1alpha1
kind: User
metadata:
  name: user-sample
spec:
  # Connection parameters in a Secret resource
  connSecretRef:
    name: quay-credentials
    # By default, the operator looks for the secret in the same namespace as
    # the user resource, but you can specify a different namespace.
    # namespace: mynamespace

  # Whether to preserve the corresponding Quay object when you
  # delete the User resource.
  preserveInQuayOnDeletion: false

  username: dwilde
  password: vs9mrD55NP
  email: dwilde@example.com
  enabled: true

```


## Properties


### connSecretRef

Reference to the secret resource that stores the connection parameters to the Quay Container Registry API.
The secret must include the `host`, `token` (or `username` and `password`), and optionally the `validateCerts` keys.


__Type__: object (see the following properties)

__Required__: True

__Default value__: None

### connSecretRef.name

Name of the secret resource.

__Type__: string

__Required__: True

__Default value__: None

### connSecretRef.namespace

Namespace of the secret resource. By default, the secret resource is retrieved from the same namespace as the current User resource.


__Type__: string

__Required__: False

__Default value__: None

### email

User`s email address. If your Quay administrator has enabled the mailing capability of your Quay installation (`FEATURE_MAILING` to `true` in `config.yaml`), then this `email' parameter is mandatory.

__Type__: string

__Required__: False

__Default value__: None

### enabled

Enable (`true`) or disable (`false`) the user account. When their account is disabled, the user cannot log in to the web UI and cannot push or pull container images.

__Type__: boolean

__Required__: False

__Default value__: None

### password

User's password as a clear string. The password must be at least eight characters long and must not contain white spaces.

__Type__: string

__Required__: False

__Default value__: None

### preserveInQuayOnDeletion

Whether to preserve the corresponding Quay object when you delete the User resource. When set to `false` (the default), the object is deleted from Quay.


__Type__: boolean

__Required__: False

__Default value__: False

### superuser

Grant superuser permissions to the user. Granting superuser privileges to a user is not immediate and usually requires a restart of the Quay Container Registry service. You cannot revoke superuser permissions.

__Type__: boolean

__Required__: False

__Default value__: None

### username

Name of the user account to create, remove, or modify.

__Type__: string

__Required__: True

__Default value__: None


## Listing the User Resources

You can retrieve the list of the User custom resources in a namespace by using the `kubectl get` command:

```sh
kubectl get users.quay.herve4m.github.io -n <namespace>
```
