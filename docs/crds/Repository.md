<!---
DO NOT EDIT THIS FILE, BECAUSE IT IS AUTOMATICALLY GENERATED FROM A TEMPLATE.
Instead, edit the scripts/templates/docs/api.md.jinja template, and then run
the scripts/crd2markdown script.
--->

# Repository - Manage Quay Container Registry repositories

The Repository custom resource relies on a Secret resource to provide the connection parameters to the Quay instance.
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

You refer to this secret in your Repository custom resource by using the `connSecretRef` property.
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
kind: Repository
metadata:
  name: repository-sample-1
spec:
  # Connection parameters in a Secret resource
  connSecretRef:
    name: quay-credentials
    # By default, the operator looks for the secret in the same namespace as
    # the repository resource, but you can specify a different namespace.
    # namespace: mynamespace

  # Whether to preserve the corresponding Quay object when you
  # delete the resource.
  preserveInQuayOnDeletion: false

  name: production/smallimage
  visibility: public
  description: |
    # My first repository

    * smallimage is a small GNU/linux container image.
    * Use podman to start a container using that image.
  perms:
    - name: operators
      type: team
      role: read
    - name: dwilde
      type: user
      role: read
    - name: production+robotprod1
      type: user
      role: admin
  append: true
  autoPruneMethod: date
  autoPruneValue: 4w
  star: true
  repoState: NORMAL
---
apiVersion: quay.herve4m.github.io/v1alpha1
kind: Repository
metadata:
  name: repository-sample-2
spec:
  # Connection parameters in a Secret resource
  connSecretRef:
    name: quay-credentials
    # By default, the operator looks for the secret in the same namespace as the
    # repository resource, but you can specify a different namespace.
    # repository: mynamespace

  # Whether to preserve the corresponding Quay object when you
  # delete the resource.
  preserveInQuayOnDeletion: false

  name: production/ubi9
  repoState: MIRROR

```


## Properties


### append

If `true`, then add the permission defined in `perms` to the repository. If `false`, then the resource sets the permissions specified in `perms`, removing all others permissions from the repository.

__Type__: boolean

__Required__: False

__Default value__: True

### autoPruneMethod

Method to use for the auto-pruning tags policy. If `none`, then the resource ensures that no policy is in place. The tags are not pruned. If `tags`, then the policy keeps only the number of tags that you specify in `autoPruneValue`. If `date`, then the policy deletes the tags older than the time period that you specify in `autoPruneValue`. `autoPruneValue` is required when `autoPruneMethod` is `tags` or `date`.

__Type__: string

__Required__: False

__Default value__: None

### autoPruneValue

Number of tags to keep when `autoPruneValue` is `tags`. The value must be 1 or more. Period of time when `autoPruneValue` is `date`. The value must be 1 or more, and must be followed by a suffix; s (for second), m (for minute), h (for hour), d (for day), or w (for week). `autoPruneMethod` is required when `autoPruneValue` is set.

__Type__: string

__Required__: False

__Default value__: None

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

Namespace of the secret resource. By default, the secret resource is retrieved from the same namespace as the current Repository resource.


__Type__: string

__Required__: False

__Default value__: None

### description

Text in Markdown format that describes the repository.

__Type__: string

__Required__: False

__Default value__: None

### name

Name of the repository to create, remove, or modify. The format for the name is `namespace`/`shortname`. The namespace can be an organization or a personal namespace. The name must be in lowercase and must not contain white spaces. If you omit the namespace part in the name, then the resource uses your personal namespace.

__Type__: string

__Required__: True

__Default value__: None

### perms

User, robot, and team permissions to associate with the repository.

__Type__: array

__Required__: False

__Default value__: None

### preserveInQuayOnDeletion

Whether to preserve the corresponding Quay object when you delete the Repository resource. When set to `false` (the default), the object is deleted from Quay.


__Type__: boolean

__Required__: False

__Default value__: False

### repoState

If `NORMAL`, then the repository is in the default state (read/write). If `READ_ONLY`, then the repository is read-only. If `MIRROR`, then the repository is a mirror and you can configure it by using the RepositoryMirror resource. You must enable the mirroring capability of your Quay installation to use this `repoState` parameter.

__Type__: string

__Required__: False

__Default value__: None

### star

If `true`, then add a star to the repository. If `false`, then remove the star. To star or unstar a repository you must provide the `quayToken` parameter to authenticate. If you are not authenticated, then the resource ignores the `star` parameter.

__Type__: boolean

__Required__: False

__Default value__: None

### visibility

If `public`, then anyone can pull images from the repository. If `private`, then nobody can access the repository and you need to explicitly grant access to users, robots, and teams. If you do not set the parameter when you create a repository, then it defaults to `private`.

__Type__: string

__Required__: False

__Default value__: None


## Listing the Repository Resources

You can retrieve the list of the Repository custom resources in a namespace by using the `kubectl get` command:

```sh
kubectl get repositories.quay.herve4m.github.io -n <namespace>
```
You can also use the short version for the resource type:

```sh
kubectl get repo -n <namespace>
```