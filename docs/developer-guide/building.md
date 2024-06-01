# Building the Operator Container Image

The `Makefile` provides targets to build and push the Operator container image.
The `Makefile` defines some variables to compose the image name:

* `IMAGE_TAG_BASE` specifies the name of the container image, without the tag part.
  `IMAGE_TAG_BASE` is set to `quay.io/herve4m/quay-api-operator`
* `VERSION` specifies the version of the Operator, and is also used as the container image tag.
* `IMG` specifies the full name of the image, and is defined as `$(IMAGE_TAG_BASE):$(VERSION)`

To use a different name for the container image, you can overwrite these variables by using environment variables before running the `make` command.

The following commands build and then push the image to the location specified by the `IMG` environment variable:

```sh
IMG=quay.io/<some-organization>/quay-api-operator:<tag>
export IMG
# Remove a previous local build of the image, if it exists
podman rmi ${IMG}
# Log in to Quay with your credentials
podman login --username <yourname> quay.io
# Build and then push the image
make docker-build docker-push
```

!!! note
    You need to have a write access to the push the image to the container registry.
    To use that image for testing the Operator, you must ensure that your Kubernetes cluster can pull the image, by making the repository public, or by configuring a pull secret.

For mode details, refer to the [Configure the Operator's Image Registry][] section of the [Ansible Operator Tutorial][] guide.

[Configure the Operator's Image Registry]: https://sdk.operatorframework.io/docs/building-operators/ansible/tutorial/#configure-the-operators-image-registry
[Ansible Operator Tutorial]: https://sdk.operatorframework.io/docs/building-operators/ansible/tutorial/
