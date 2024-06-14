# Testing the API Operator for Quay with Molecule

The Molecule scenario tests the API Operator resources.
The scenario creates a [Kind](https://kind.sigs.k8s.io/) Kubernetes cluster, deploys a Quay Container Registry instance in this cluster, deploys the API Operator for Quay, and then uses test resources to verify that the Operator operates correctly.

The Molecule scenario configuration is stored in the `molecule/kind` directory.
Some files in this directory refer to files from the `molecule/directory` directory.

## Requirements

The Molecule scenario requires that you install the following applications in your system:

* [ansible-core](https://docs.ansible.com/ansible/latest/installation_guide/)
* [Molecule](https://ansible.readthedocs.io/projects/molecule/installation/)
* The Kubernetes Python package
* [Docker](https://docs.docker.com/engine/install/)
* [Kind](https://kind.sigs.k8s.io/docs/user/quick-start)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/)

## Running the Molecule Scenario

From the root directory of the Operator GitHub repository, run the following command:

```sh
molecule test --scenario-name kind
```

The command creates a Kind Kubernetes cluster, and then deploys a Quay Container Registry instance from the Kubernetes resources in the `molecule/default/quay` directory.
The command deploys the Operator, and runs the tests from the Ansible tasks in the `molecule/default/tasks` directory.
These tasks use the Kubernetes resources in the `config/samples` directory to test the Operator.

After the tests, the scenario deletes the Kind cluster.
