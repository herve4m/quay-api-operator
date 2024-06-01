# Releasing the Operator to OperatorHub.io

Prepare the Operator container image and the bundle manifests:

1. Edit the `Makefile`, and update the `VERSION` variable.
2. Edit the `config/manifests/bases/quay-api-operator.clusterserviceversion.yaml` file, and update the tag part of the image in `.metadata.annotations.containerImage`.
3. [Build and push](building.md) the Operator container image.
4. Create the bundle manifests by running the `make bundle` command.
5. Review the [Packaging your Operator for OperatorHub][] documentation.
6. Follow the steps in the [Contributing] documentation.

[Packaging your Operator for OperatorHub]: https://k8s-operatorhub.github.io/community-operators/packaging-operator/
[Contributing]: https://k8s-operatorhub.github.io/community-operators/contributing-prerequisites/
