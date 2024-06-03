# API Operator for Quay

Manage Quay Container Registry deployments by using Kubernetes resources.

## Description

The API Operator for Quay manages Quay Container Registry components.
The Operator does not install Quay, which you can install in Kubernetes by using the [Quay Operator](https://operatorhub.io/operator/project-quay), or outside Kubernetes as a standalone deployment.

The API Operator for Quay provides Kubernetes custom resources to create, configure, and delete Quay objects.
For example, the Operator can manage Quay organizations, repositories, teams, and user and robot accounts.
It can also configure organization quotas, repository mirroring, and proxy caches.

For usage instructions, refer to the [User Guide](https://herve4m.github.io/quay-api-operator/).
The [Developer Guide](https://herve4m.github.io/quay-api-operator/developer-guide) provides instructions for building, deploying, and testing the Operator.

## Contributing

We welcome community contributions to this operator.
If you find problems, then please open an [issue](https://github.com/herve4m/quay-api-operator/issues) or create a [pull request](https://github.com/herve4m/quay-api-operator/pulls).

More information about contributing can be found in the [Contribution Guidelines](https://github.com/herve4m/quay-api-operator/blob/main/CONTRIBUTING.md).


## License

Copyright 2024.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
