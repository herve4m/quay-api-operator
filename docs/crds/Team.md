<!---
DO NOT EDIT THIS FILE, BECAUSE IT IS AUTOMATICALLY GENERATED FROM A TEMPLATE.
Instead, edit the scripts/templates/docs/api.md.jinja template, and then run
the scripts/crd2markdown script.
--->

# Team - Manage Quay Container Registry teams

The Team custom resource relies on a Secret resource to provide the connection parameters to the Quay instance.
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

You refer to this secret in your Team custom resource by using the `connSecretRef` property.
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
kind: Team
metadata:
  name: team-sample
spec:
  # Connection parameters in a Secret resource
  connSecretRef:
    name: quay-credentials
    # By default, the operator looks for the secret in the same namespace as
    # the team resource, but you can specify a different namespace.
    # namespace: mynamespace

  # Whether to preserve the corresponding Quay object when you
  # delete the Team resource.
  preserveInQuayOnDeletion: false

  name: operators
  organization: production
  description: |
    # Operation Team

    * Operators can create repositories
    * Operators can store their images in those repositories
  role: creator
  members:
    - dwilde
    - production+robotprod1
  append: false

```


## Properties


### append

If `true`, then add the users specified in `members` to the team. If `false`, then the resource sets the team members to users specified in `members`, removing all others users from the team.

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

Namespace of the secret resource. By default, the secret resource is retrieved from the same namespace as the current Team resource.


__Type__: string

__Required__: False

__Default value__: None

### description

Text in Markdown format that describes the team.

__Type__: string

__Required__: False

__Default value__: None

### members

List of the user or robot accounts in the team. Use the syntax `organization`+`robotshortname` for robot accounts. If the team is synchronized with an LDAP group (see the TeamLdap resource), then you can only add or remove robot accounts.

__Type__: array

__Required__: False

__Default value__: None

### name

Name of the team to create, remove, or modify. The name must be in lowercase, must not contain white spaces, must not start by a digit, and must be at least two characters long.

__Type__: string

__Required__: True

__Default value__: None

### organization

Name of the organization for the team. This organization must exist.

__Type__: string

__Required__: True

__Default value__: None

### preserveInQuayOnDeletion

Whether to preserve the corresponding Quay object when you delete the Team resource. When set to `false` (the default), the object is deleted from Quay.


__Type__: boolean

__Required__: False

__Default value__: False

### role

Role of the team within the organization. If not set, then the new team has the `member` role.

__Type__: string

__Required__: False

__Default value__: None


## Listing the Team Resources

You can retrieve the list of the Team custom resources in a namespace by using the `kubectl get` command:

```sh
kubectl get teams.quay.herve4m.github.io -n <namespace>
```
