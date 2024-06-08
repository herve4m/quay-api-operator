# Deploy a Testing Quay Instance in Kubernetes

The following instructions require a Kubernetes or OpenShift cluster.

1. Create the `quay` Kubernetes namespace to host the Quay instance.
   Do not use a different name, because the resources rely on this name.

        kubectl create namespace quay

2. Deploy the resources from the YAML files:

        kubectl apply -n quay -f postgresql.yaml
        kubectl apply -n quay -f redis.yaml
        kubectl apply -n quay -f fake-clair.yaml
        sleep 20   # Give time for the pods to be up and running
        kubectl apply -n quay -f quay.yaml
        sleep 60   # Give time for Quay to start (it might restart a few times)
        kubectl get pods -n quay # All pods should be up and running

The Quay instance is accessible only as a service, and is not accessible from outside the cluster.
The Quay instance is available at `https://quay.quay.svc.cluster.local` from inside the cluster.
