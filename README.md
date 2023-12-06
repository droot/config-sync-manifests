# config-sync-manifests
Kubernetes manifests for config-sync

This repo contains kubernetes manifests for a modified version of [config-sync](https://github.com/GoogleContainerTools/kpt-config-sync/pull/985)
that supports BYOID (Bring your own identity).

With this release, you can use a new authType `gcpksa` for fetching OCI artifacts from GCP AR.

Steps to install:

```sh
KUBECONFIG=<path/to/kubeconfig> kubectl apply -f https://raw.githubusercontent.com/droot/config-sync-manifests/main/config-sync-manifest.yaml
```

An example of RootSync that supports new auth type:

```yaml
apiVersion: configsync.gke.io/v1beta1
kind: RootSync
metadata:
  name: oci-test
  namespace: config-management-system
spec:
  sourceFormat: unstructured
  sourceType: oci
  oci:
    image: us-central1-docker.pkg.dev/<GCP_PROJECT>/pkg-samples/test-namespace:v1
    dir: .
    auth: gcpksa
```

## Steps to verify that workload identity is working correctly and using the `BYOID` setup:


```sh
# log in to the OCI pod to examine the auth token
KUBECONFIG=<path/to/kubeconfig> kubectl exec -it root-reconciler-oci-test-7945d4c9c6-77zsj -c oci-sync -n config-management-system -- /bin/bash
```

```sh
curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/email
```
This should print the workload identity pool. `PROJECT.svc.id.goog`

```sh
curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
```

should print token. BYOID token starts with "ya29.d......".


## Steps used to prepare the config-sync release

```sh

# instructions for generating the CRDs
make configsync-crds-in-docker

# building images
make build-manifests REGISTRY=ghcr.io/droot IMAGE_TAG=gcp-ksa-1.0.1

# push images
make push-images REGISTRY=ghcr.io/droot IMAGE_TAG=gcp-ksa-1.0.1

# build config-sync bundle
make config-sync-manifest REGISTRY=ghcr.io/droot IMAGE_TAG=gcp-ksa-1.0.1

```
