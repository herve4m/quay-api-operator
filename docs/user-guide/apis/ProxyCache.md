<!---
DO NOT EDIT THIS FILE, BECAUSE IT IS AUTOMATICALLY GENERATED FROM A TEMPLATE.
Instead, edit the scripts/templates/docs/api.md.jinja template, and then run
the scripts/crd2markdown script.
--->

# ProxyCache - Manage Quay Container Registry proxy cache configurations

The ProxyCache custom resource relies on a Secret resource to provide the connection parameters to the Quay instance.
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

You refer to this secret in your ProxyCache custom resources by the using the `connSecretRef` property:

```yaml
---
apiVersion: quay.herve4m.github.io/v1alpha1
kind: ProxyCache
metadata:
  name: ProxyCache-sample
spec:
  # Connection parameters in a Secret resource
  connSecretRef:
    name: quay-credentials
    # By default, the operator looks for the secret in the same namespace as
    # the ProxyCache resource, but you can specify a different namespace.
    # namespace: mynamespace
...
```


## Usage Example

```yaml
---
apiVersion: quay.herve4m.github.io/v1alpha1
kind: ProxyCache
metadata:
  name: proxycache-sample
spec:
  # Connection parameters in a Secret resource
  connSecretRef:
    name: quay-credentials-secret
    # By default, the operator looks for the secret in the same namespace as the
    # proxycache resource, but you can specify a different namespace.
    # namespace: mynamespace

  # Whether to preserve the corresponding configuration in Quay when you
  # delete the resource.
  preserveInQuayOnDeletion: false

  organization: production
  registry: quay.io/prodimgs
  username: cwade
  password: My53cr3Tpa55
  expiration: 172800

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

Namespace of the secret resource. By default, the secret resource is retrieved from the same namespace as the current ProxyCache resource.


__Type__: string

__Required__: False

__Default value__: None

### expiration

Tag expiration in seconds for cached images. 86400 (one day) by default.

__Type__: integer

__Required__: False

__Default value__: 86400

### insecure

Whether to allow insecure connections to the remote registry. If `true`, then the resource does not validate SSL certificates.

__Type__: boolean

__Required__: False

__Default value__: None

### organization

Name of the organization in which to create the proxy cache configuration. That organization must exist.

__Type__: string

__Required__: True

__Default value__: None

### password

User's password as a clear string. Do not set a password for a public access to the remote registry.

__Type__: string

__Required__: False

__Default value__: None

### preserveInQuayOnDeletion

Whether to preserve the corresponding Quay object when you delete the ProxyCache resource. When set to `false` (the default), the object is deleted from Quay.


__Type__: boolean

__Required__: False

__Default value__: False

### registry

Name of the remote registry. Add a namespace to the remote registry to restrict caching images from that namespace.

__Type__: string

__Required__: False

__Default value__: quay.io

### username

Name of the user account to use for authenticating with the remote registry. Do not set a username for a public access to the remote registry.

__Type__: string

__Required__: False

__Default value__: None


## Listing the ProxyCache Resources

You can retrieve the list of the ProxyCache custom resources in a namespace by using the `kubectl get` command:

```sh
kubectl get proxycaches.quay.herve4m.github.io -n <namespace>
```