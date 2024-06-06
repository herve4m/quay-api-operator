# Installing the Tools for Development

The Operator building and testing processes require that you install some tools in your system:

* `make`
* `awk`
* `podman` or `docker`
* `kubectl`.
  See the [Install Tools](https://kubernetes.io/docs/tasks/tools/) documentation.
* The Kubernetes Community Bundle Validator, `k8s-community-bundle-validator`.
  You can download the command from the project [releases](https://github.com/k8s-operatorhub/bundle-validator/releases/) page.

The `Makefile` tries to automatically install the `kustomize` and the `operator-sdk` commands in the `./bin` directory, if these commands are not already available on your system.

!!! note
    The `.gitignore` file lists this `bin` directory to prevent Git from tracking the directory.
