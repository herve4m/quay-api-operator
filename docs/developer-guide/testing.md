# Validating and Testing the Operator

Before submitting your work, validate and test the Operator.


## Validating the Operator

Some of the following validation tests require a Kubernetes cluster to run.
All these tests are automatically performed by GitHub Actions when committing your work.
See the `.github/workflows/tests.yaml` file.

The tests use the `bundle/` directory that you build by running the `make bundle` command:

```sh
make bundle
```

Validating the bundle manifests by using the `operator-sdk bundle validate` command:

```sh
operator-sdk bundle validate ./bundle --select-optional suite=operatorframework
```

!!! note
    If the `operator-sdk` command is not available on your system, then the `make bundle` command that you used previously downloads and installs it in the `./bin` directory in the current directory.
    In that case, specify the path to the `operator-sdk` command: `./bin/operator-sdk bundle validate ...`

Validating the bundle manifests by using the `k8s-community-bundle-validator` command (see the [tools](tools.md) section to install this command).
A 0 return code indicates success:

```sh
k8s-community-bundle-validator ./bundle
echo $?
```

Validate the bundle manifests by using Scorecard.
Scorecard uses pods to run the tests, and therefore requires a Kubernetes cluster.
Ensure that you are logged in to your cluster before running Scorecard.
A 0 return code indicates success:

```sh
operator-sdk scorecard ./bundle
echo $?
```

The command returns a lot of output.
Each test reports its status in the `State` field:

```text
--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.34.2
Entrypoint: [scorecard-test olm-bundle-validation]
Labels:
	"suite":"olm"
	"test":"olm-bundle-validation-test"
Results:
	Name: olm-bundle-validation
	State: pass

	Log:
		time="2024-05-31T16:43:27Z" level=debug msg="Found manifests directory" name=bundle-test
		time="2024-05-31T16:43:27Z" level=debug msg="Found metadata directory" name=bundle-test
		time="2024-05-31T16:43:27Z" level=debug msg="Getting mediaType info from manifests directory" name=bundle-test
		time="2024-05-31T16:43:27Z" level=debug msg="Found annotations file" name=bundle-test
		time="2024-05-31T16:43:27Z" level=debug msg="Could not find optional dependencies file" name=bundle-test


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.34.2
Entrypoint: [scorecard-test basic-check-spec]
Labels:
	"suite":"basic"
	"test":"basic-check-spec-test"
Results:
	Name: basic-check-spec
	State: pass


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.34.2
Entrypoint: [scorecard-test olm-crds-have-resources]
Labels:
	"suite":"olm"
	"test":"olm-crds-have-resources-test"
Results:
	Name: olm-crds-have-resources
	State: pass

	Log:
		Loaded ClusterServiceVersion: quay-api-operator.v0.1.0
...
```


## Testing the Operator

Before you can test the Operator, you need to prepare your cluster:

1. [Build and publish](building.md) the container image.
2. [Deploy](deploying.md) the Operator.
3. Deploy a Quay instance for running the tests.
   You can use the [Quay Operator](https://github.com/quay/quay-operator), or use the following instructions to deploy a test Quay instance in OpenShift.

### Deploying Quay in OpenShift

The following instructions require an OpenShift cluster, with at least 3 GiB of memory available on a node.
The Quay instance uses ephemeral storage, and you should use it only for testing the Operator.

The resources in YAML format are available in the `scripts/Quay-in-OpenShift` directory.

1. Retrieve the DNS domain of your cluster.
   The following `oc` command might provide you with that information, otherwise review the domain of the existing route resources (`oc get routes -A`):

        oc get ingresscontroller/default -n openshift-ingress-operator -o jsonpath='{.spec.domain}'

2. Change to the `scripts/Quay-in-OpenShift` directory.

        cd ./scripts/Quay-in-OpenShift

3. Edit the `quay.yaml` file, and replace the `<CHANGE_ME>` placeholder in the `SERVER_HOSTNAME` variable with the DNS domain:

        ...
        SECURITY_SCANNER_V4_ENDPOINT: http://fake-clair
        SECURITY_SCANNER_V4_PSK: MmNiOTBoNWdnNzli
        # CHANGE_ME:
        # oc get ingresscontroller/default -n openshift-ingress-operator -o jsonpath='{.spec.domain}'
        SERVER_HOSTNAME: quay-quay.<DNS-domain>
        PREFERRED_URL_SCHEME: https
        EXTERNAL_TLS_TERMINATION: true
        ...

4. Create a new OpenShift project to host the Quay instance:

        oc new-project quay

5. Deploy the resources from the YAML files:

        oc apply -f postgresql.yaml
        oc apply -f redis.yaml
        oc apply -f fake-clair.yaml
        sleep 20   # Give time to the pods to be up and running
        oc apply -f quay.yaml
        sleep 60   # Give time to Quay to start (it might restart a few times)
        oc get pods  # All pods should be up and running

This Quay installation enables the first user creation feature (`FEATURE_USER_INITIALIZE` in `config.yaml`), which allows you to test the `FirstUser` Kubernetes resource to bootstrap Quay.

### Applying the Test Resources

The `config/samples` directory stores some sample resource files in YAML format that you can use to test the Operator.

The resources refer to Secrets to retrieve the Quay connection parameters.
However, you only have to create the Secret for the `FirstUser` resource, which in turn creates a Secret to store the temporary Quay credentials.
Then, the `ApiToken` resource uses that new secret to generate a permanent OAuth token that it stores in a third Secret resource, which in turn is used by all the other resources.


[quay-connection-secret] --> fisrtuser --> [quay-temp-credentials-secret] --> organization/application/apitoken --> [quay-credentials-secret] --> ...


Before applying the resources, create a project and the Secret for the `FirstUser` resource.
In the following command, replace the `<DNS-domain>` placeholder by the domain of your OpenShift cluster.

```sh
oc new-project test-operator
oc create secret generic quay-connection-secret --from-literal host=https://quay-quay.<DNS-domain> --from-literal validateCerts=false
```

Use the `oc apply` command to deploy all the resources:

```sh
oc apply -k config/samples
```

Most resources initially report an error state, because the Secret that provides the Quay connection parameters does not exist yet (the `FirstUser` and `ApiToken` resources must complete first).
It takes 30 minutes to reconcile.

You can review the status of the resources by using the following loop:

```sh
for i in apitoken.quay.herve4m.github.com/apitoken-sample \
  application.quay.herve4m.github.com/application-sample \
  defaultperm.quay.herve4m.github.com/defaultperm-sample \
  dockertoken.quay.herve4m.github.com/dockertoken-sample \
  firstuser.quay.herve4m.github.com/firstuser-sample \
  manifestlabel.quay.herve4m.github.com/manifestlabel-sample \
  message.quay.herve4m.github.com/message-sample \
  notification.quay.herve4m.github.com/notification-sample \
  organization.quay.herve4m.github.com/organization-sample \
  proxycache.quay.herve4m.github.com/proxycache-sample \
  quota.quay.herve4m.github.com/quota-sample \
  repository.quay.herve4m.github.com/repository-sample-1 \
  repository.quay.herve4m.github.com/repository-sample-2 \
  repositorymirror.quay.herve4m.github.com/repositorymirror-sample \
  robot.quay.herve4m.github.com/robot-sample \
  tag.quay.herve4m.github.com/tag-sample \
  team.quay.herve4m.github.com/team-sample \
  teamldap.quay.herve4m.github.com/teamldap-sample \
  teamoidc.quay.herve4m.github.com/teamoidc-sample \
  user.quay.herve4m.github.com/user-sample
do
  oc get --no-headers $i
done
```

!!! note
    The `manifestlabel-sample`, `teamldap-sample`, `teamoidc-sample`, and `tag-sample` resources stay in the error state.
    This state is expected because these resources depend on Quay features or container images that are not available in this test environment.

### Deleting the Test Resources

Delete the resources by using the `oc delete` command:

```sh
oc delete -k config/samples
oc delete project test-operator
```

You can delete the Quay installation by deleting the `quay` project:

```sh
oc delete project quay
```
