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
