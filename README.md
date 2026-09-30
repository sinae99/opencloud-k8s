# OpenCloud FileServer

Kubernetes deployment for a standalone OpenCloud server.

- One OpenCloud pod: `opencloudeu/opencloud:7.2.4`
- `opencloud-init` Job
- `local-path` PVC for config and local state
- ClusterIP Service on port `9200`
- Traefik Ingress
- ConfigMap for non-sensitive settings
- Secret for S3 credentials and the admin password


## Conf

Edit `k8s/configmap.yaml`

- `OC_URL`
- `STORAGE_USERS_DECOMPOSEDS3_ENDPOINT`
- `STORAGE_USERS_DECOMPOSEDS3_REGION`
- `STORAGE_USERS_DECOMPOSEDS3_BUCKET`

Replace all placeholders in `k8s/secret.yaml`:

- `STORAGE_USERS_DECOMPOSEDS3_ACCESS_KEY`
- `STORAGE_USERS_DECOMPOSEDS3_SECRET_KEY`
- `IDM_ADMIN_PASSWORD`

Set `OC_URL` to the public URL and use the same hostname in `k8s/ingress.yaml`.

OCM is disabled with `OC_ENABLE_OCM=false` and `OC_EXCLUDE_RUN_SERVICES=ocm`.

## Init

The `opencloud-init` Job runs `opencloud init` only if the configuration file is missing. The Deployment waits for that file, then runs `opencloud server`; restarts do not reinitialize existing state.


```sh
kubectl apply -k k8s/
```

