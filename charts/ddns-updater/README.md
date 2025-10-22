# ddns-updater chart

A Helm chart for deploying a ddns-updater server to a Kubernetes cluster

### Installation via Helm

1. Add the Helm chart repo

```bash
helm repo add ddns-updater https://raw.githubusercontent.com/zbroad84/ddns-updater/gh-pages
```

2. Inspect & modify the default values (optional)

```bash
helm show values zbroad84/ddns-updater > values.yaml
```

3. Install the chart

```bash
helm upgrade --install ddns-updater zbroad84/ddns-updater --values values.yaml
```

## Values

| Key | Default | Description |
| :--- | :--- | :--- |
| **App Configuration** | | |
| `config.PERIOD` | `5m` | The default period for checking the public IP address. |
| `config.PUBLICIP_FETCHERS` | `all` | Comma-separated list of fetcher types (`http`, `dns`) to obtain the public IP address. |
| `config.UPDATE_COOLDOWN_PERIOD` | `5m` | Duration to wait between updates for each record to avoid rate limiting. |
| `config.SERVER_ENABLED` | `yes` | Enables the built-in web server and web UI. |
| `config.LISTENING_ADDRESS` | `:8000` | Internal TCP listening port for the web UI. |
| `config.ROOT_URL` | `/` | URL path to append to all paths for the webUI (useful for proxies). |
| `config.DATADIR` | `/updater/data` | Directory to read and write data files from internally. |
| `config.RESOLVER_ADDRESS` | `1.1.1.1:53` | A plaintext DNS address to use to resolve domain names defined in settings (useful for split DNS). |
| `config.LOG_LEVEL` | `info` | Level of logging (`debug`, `info`, `warning`, or `error`). |
| `config.TZ` | `America/New_York` | Timezone for accurate times. |
| `config.SHOUTRRR_ADDRESSES` | `[]` | (optional) Comma-separated list of Shoutrrr addresses (notification services). |
| **Image** | | |
| `replicaCount` | `1` | The number of deployment replicas. |
| `image.repository` | `qmcgaw/ddns-updater` | The Docker image repository. |
| `image.pullPolicy` | `Always` | The image pull policy. |
| `image.tag` | `""` | Overrides the image tag (defaults to chart `appVersion` if empty). |
| **Persistence** | | |
| `persistence.enabled` | `false` | If true, creates a PersistentVolumeClaim for `/updater/data`. |
| `persistence.storage_capacity` | `5Gi` | The requested storage capacity for the PVC. |
| `persistence.storageClassName` | `manual-hostpath` | The storage class for the PVC. |
| `persistence.accessModes` | `[ReadWriteOnce]` | The access modes for the PVC. |
| **Service** | | |
| `service.type` | `ClusterIP` | Kubernetes service type. |
| `service.port` | `80` | The service port. |
| **Ingress** | | |
| `ingress.enabled` | `false` | If true, creates an Ingress resource. |
| `ingress.hosts` | `[chart-example.local]` | A list of hostnames to use for the Ingress. |
| **Autoscaling** | | |
| `autoscaling.enabled` | `false` | If true, enables Horizontal Pod Autoscaler. |
| `autoscaling.minReplicas` | `1` | Minimum number of replicas for HPA. |
| `autoscaling.maxReplicas` | `100` | Maximum number of replicas for HPA. |
| **Security/Contexts** | | |
| `podSecurityContext` | `{}` | Security context applied to the pod. |
| `securityContext` | `{}` | Security context applied to the container. |

---

### Additional Configuration Notes

For **Public IP Providers** options, such as `config.PUBLICIP_HTTP_PROVIDERS`, `config.PUBLICIPV4_HTTP_PROVIDERS`, `config.PUBLICIPV6_HTTP_PROVIDERS`, and `config.PUBLICIP_DNS_PROVIDERS`, the default value of `all` uses the complete set of built-in providers. Refer to the application's documentation for a full list of available providers if you wish to restrict them.