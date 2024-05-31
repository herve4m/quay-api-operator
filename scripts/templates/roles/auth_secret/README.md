auth_secret
=========

This role retrieves the connection parameters to the Quay Container Registry REST API from a Kubernetes secret resource.
The parameters are then stored in Ansible facts so that other roles can use them.

The role retrieves the following fields from the secret resource:

* `host`: URL for accessing the Quay Container Registry API.
  The role sets the `quay_host` Ansible fact from this field.
* `token`: OAuth access token for authenticating with the API.
  If you also provide the `username` and `password` fields, then these two fields are ignored.
  The role sets the `quay_token` Ansible fact from this field.
* `username`: Username for authenticating with the API.
  You also need to define the `password` field.
  If you provide the `token` field, then `username` and `password` are ignored.
  The role sets the `quay_username` Ansible fact from this field.
* `password`: Password for authenticating with the API.
  If you provide the `token` field, then `username` and `password` are ignored.
  The role sets the `quay_password` Ansible fact from this field.
* `validateCerts`: Whether to allow insecure connections to the API.
  The role sets the `validate_certs` Ansible fact from this field.

Use the following command as an example to create a secret resource for authenticating to the Quay Container Registry REST API with an OAuth access token:

```
kubectl create secret generic quay-creds-token --from-literal host=https://quay.example.com:8443 --from-literal validateCerts=true --from-literal token=XYZ
```

Use the following command as an example to create a secret resource for authenticating to the Quay Container Registry REST API with a username and a password:

```
kubectl create secret generic quay-creds-user --from-literal host=https://quay.example.com:8443 --from-literal validateCerts=true --from-literal username=admin --from-literal password=s3cr3t
```


Requirements
------------

To access the API, you can use a username and a password, or an OAuth access token.
If you wish to use an OAuth access token, then you can create one as follows:

1. Log in to the Quay Container Registry web UI with a user account that has superuser permissions.
2. Use an existing organization or create a new one.
3. In the organization, create an application.
4. In the application, select the `Generate Token` menu.
5. Select the permissions to associate to the token.
6. Click `Generate Token`.
7. Use the token with the `token` field of the Kubernetes secret resource.


Role Variables
--------------

The role accepts the following variables:

* `conn_secret_ref['name']`: Name of the Kubernetes secret resource.
* `conn_secret_ref['namespace']`: Namespace that hosts the secret resource.
* `ansible_operator_meta['namespace']`: Namespace that hosts the secret resource if `conn_secret_ref['namespace']` is not set.

The Ansible Operator declares these variables from the custom resource.


Example Playbook
----------------

The role is meant to be used as a dependency for roles that need to retrieve to the Quay connection parameters.
These roles can specify the dependency in the `meta/main.yml` file:

```yaml
---
dependencies:
  - auth_secret
```


License
-------

GPL 3.0 or later.


Author Information
------------------

This role was created in 2024 by Hervé Quatremain
