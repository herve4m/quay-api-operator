# Deploy a Testing Quay Instance in OpenShift

The following instructions require an OpenShift cluster.

1. Retrieve the DNS domain of your cluster.
   The following `oc` command might provide you with that information, otherwise review the domain of the existing route resources (`oc get routes -A`):

        oc get ingresscontroller/default -n openshift-ingress-operator -o jsonpath='{.spec.domain}'

2. Edit the `quay.yaml` file, and replace the `<CHANGE_ME>` placeholder in the `SERVER_HOSTNAME` variable with the DNS domain:

        ...
        SECURITY_SCANNER_V4_ENDPOINT: http://fake-clair
        SECURITY_SCANNER_V4_PSK: MmNiOTBoNWdnNzli
        # CHANGE_ME:
        # oc get ingresscontroller/default -n openshift-ingress-operator -o jsonpath='{.spec.domain}'
        SERVER_HOSTNAME: quay-quay.<DNS-domain>
        PREFERRED_URL_SCHEME: https
        EXTERNAL_TLS_TERMINATION: true
        ...

3. Create a new OpenShift project to host the Quay instance:

        oc new-project quay

4. Deploy the resources from the YAML files:

        oc apply -f postgresql.yaml
        oc apply -f redis.yaml
        oc apply -f fake-clair.yaml
        sleep 20   # Give time to the pods to be up and running
        oc apply -f quay.yaml
        sleep 60   # Give time to Quay to start (it might restart a few times)
        oc get pods  # All pods should be up and running
