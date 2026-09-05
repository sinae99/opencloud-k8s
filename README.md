# OpenCloud - FileServer

Kubernetes deployment for OpenCloud

- One `opencloudeu/opencloud:7.2.4` pod.
- One guarded `opencloud-init` Job for first-time initialization.
- One `local-path` PVC for OpenCloud configuration and local state.
- One ClusterIP Service on port `9200`.
- One Traefik Ingress.
- One ConfigMap containing non-sensitive OpenCloud configuration values and placeholders.
- One Secret containing storage credentials and the administrator password placeholders.
- OCM federation disabled because this is a standalone deployment.

OpenCloud runs as one container; the user/file data backend is the external S3-compatible service.

## Arch

```text
Internet
   |
   v
Traefik Ingress
   |
   v
OpenCloud ClusterIP Service :9200
   |
   v
OpenCloud Pod
   +-- local-path PVC: config and local state
   +-- external S3: user/file data via decomposeds3
```

## values

Edit `k8s/configmap.yaml` :

- `OC_URL`
- `STORAGE_USERS_DECOMPOSEDS3_ENDPOINT`
- `STORAGE_USERS_DECOMPOSEDS3_REGION`
- `STORAGE_USERS_DECOMPOSEDS3_BUCKET`

Edit `k8s/secret.yaml` and replace every placeholder:

- `STORAGE_USERS_DECOMPOSEDS3_ACCESS_KEY`
- `STORAGE_USERS_DECOMPOSEDS3_SECRET_KEY`
- `IDM_ADMIN_PASSWORD`

Set `OC_URL` to the public URL.
Replace the example `host` value in `k8s/ingress.yaml` with the same hostname.

OCM federation is disabled with `OC_ENABLE_OCM=false` and `OC_EXCLUDE_RUN_SERVICES=ocm`; the latter prevents the OCM service process from starting and avoids requiring an OCM provider domain and WebDAV federation metadata for this standalone deployment.

## Init

`opencloud-init` mounts the same PVC as the Deployment and runs `opencloud init` only when `${OC_CONFIG_DIR}/opencloud.yaml` does not exist. If initialization has already completed, the Job exits successfully without changing the existing configuration.

The Deployment does not run `opencloud init`. It waits for the initialized configuration file and then runs `opencloud server`. This prevents a pod restart from overwriting or reinitializing existing state.

The Job and Deployment use the documented production image tag `opencloudeu/opencloud:7.2.4`. Change the tag consistently in `k8s/job.yaml` and `k8s/deployment.yaml` only after checking the OpenCloud release lifecycle documentation.

## Deploy

```sh
kubectl apply -k k8s/
```

Because the Job and Deployment may be created at the same time, the Deployment waits for the Job-created configuration file. Do not delete the PVC unless you intentionally want to remove the local OpenCloud configuration and state.

## Access

After replacing the example Ingress host, creating the required DNS record, and configuring TLS on Traefik, open the value configured in `OC_URL` in a browser. The initial administrator password is the value of `IDM_ADMIN_PASSWORD` in `k8s/secret.yaml`.

