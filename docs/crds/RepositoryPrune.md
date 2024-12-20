<!---
DO NOT EDIT THIS FILE, BECAUSE IT IS AUTOMATICALLY GENERATED FROM A TEMPLATE.
Instead, edit the scripts/templates/docs/api.md.jinja template, and then run
the scripts/crd2markdown script.
--->

# RepositoryPrune - Manage auto-pruning policies for repositories

The RepositoryPrune custom resource relies on a Secret resource to provide the connection parameters to the Quay instance.
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

You refer to this secret in your RepositoryPrune custom resource by using the `connSecretRef` property.
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
kind: RepositoryPrune
metadata:
  name: repositoryprune-sample
spec:
  # Connection parameters in a Secret resource
  connSecretRef:
    name: quay-credentials
    # By default, the operator looks for the secret in the same namespace as
    # the repositoryprune resource, but you can specify a different namespace.
    # namespace: mynamespace

  # Whether to preserve the corresponding Quay object when you
  # delete the RepositoryPrune resource.
  preserveInQuayOnDeletion: false

  repository: production/smallimage
  method: tags
  value: "5"
  tagPattern: nightly

```


## Properties


### append

If `true`, then add the auto-pruning policy to the existing policies. If `false`, then the resource deletes all the existing auto-pruning policies before adding the specified policy.

__Type__: boolean

__Required__: False

__Default value__: True

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

Namespace of the secret resource. By default, the secret resource is retrieved from the same namespace as the current RepositoryPrune resource.


__Type__: string

__Required__: False

__Default value__: None

### method

Method to use for the auto-pruning tags policy. If `tags`, then the policy keeps only the number of tags that you specify in `value`. If `date`, then the policy deletes the tags older than the time period that you specify in `value`.

__Type__: string

__Required__: True

__Default value__: None

### preserveInQuayOnDeletion

Whether to preserve the corresponding Quay object when you delete the RepositoryPrune resource. When set to `false` (the default), the object is deleted from Quay.


__Type__: boolean

__Required__: False

__Default value__: False

### repository

Name of the existing repository to configure. The format for the name is `namespace`/`shortname`. The namespace can be an organization or your personal namespace. If you omit the namespace part in the name, then the resource looks for the repository in your personal namespace. You can manage auto-pruning policies for repositories in your personal namespace, but not in the personal namespace of other users. The token you use in `quayToken` determines the user account you are using.

__Type__: string

__Required__: True

__Default value__: None

### retSecretRef

RetSecretRef is the secret resource that the RepositoryPrune resource creates. This secret will store the data that the resource generates:

- id - Internal identifier of the auto-pruning policy.


__Type__: object (see the following properties)

__Required__: False

__Default value__: None

### retSecretRef.name

Name of the secret resource.

__Type__: string

__Required__: True

__Default value__: None

### retSecretRef.namespace

Namespace of the secret resource. By default, the secret resource is created in the same namespace as the current RepositoryPrune resource.


__Type__: string

__Required__: False

__Default value__: None

### tagPattern

Regular expression to select the tags to process. If you do not set the parameter, then Quay processes all the tags.

__Type__: string

__Required__: False

__Default value__: None

### tagPatternMatches

If `true`, then Quay processes the tags matching the `tagPattern` parameter. If `false`, then Quay excludes the tags matching the `tagPattern` parameter. `true` by default.

__Type__: boolean

__Required__: False

__Default value__: True

### value

Number of tags to keep when `method` is `tags`. The value must be 1 or more. Period of time when `method` is `date`. The value must be 1 or more, and must be followed by a suffix; s (for second), m (for minute), h (for hour), d (for day), or w (for week).

__Type__: string

__Required__: True

__Default value__: None


## Listing the RepositoryPrune Resources

You can retrieve the list of the RepositoryPrune custom resources in a namespace by using the `kubectl get` command:

```sh
kubectl get repositoryprunes.quay.herve4m.github.io -n <namespace>
```
