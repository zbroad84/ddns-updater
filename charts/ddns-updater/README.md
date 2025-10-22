# ddns-updater Helm Chart

A Helm chart for deploying the `qmcgaw/ddns-updater` application to a Kubernetes cluster. This chart supports two primary deployment modes: a **periodic CronJob** for simple updates or a **long-running Deployment** with a Web UI.

* **Chart Version:** `1.0.6`
* **App Version:** `2.9.0`
* **Type:** `application`

---

## Installation via Helm

1.  Add the Helm chart repo

    ```bash
    helm repo add ddns-updater [https://raw.githubusercontent.com/zbroad84/ddns-updater/gh-pages](https://raw.githubusercontent.com/zbroad84/ddns-updater/gh-pages)
    ```

2.  Inspect & modify the default values (optional)

    ```bash
    helm show values ddns-updater/ddns-updater > values.yaml
    ```

3.  Install the chart

    ```bash
    helm upgrade --install ddns-updater ddns-updater/ddns-updater --values values.yaml
    ```

---

## Configuration

### Manual Configuration via `config.json` ⚠️

The `ddns-updater` application is designed to be configured primarily via environment variables (supplied by the `ddns-updater-config` ConfigMap in this chart).

**However, if you experience issues with environment variables not correctly populating the application's settings (e.g., DNS records, provider credentials), the recommended workaround is to manually provide a complete `config.json` file on the host.**

1.  Create your `config.json` file containing all required settings.
2.  Ensure your `values.yaml` is configured to mount this file via `hostPath` persistence.

The chart uses the following values to manage this manual configuration:

| Parameter | Purpose |
| :--- | :--- |
| `persistence.enabled: true` | Must be enabled to mount data. |
| `persistence.hostPath` | Must point to the directory on the host containing your `config.json`. |
| `config.DATADIR` | Must match the internal mount path (`/updater/data` by default). |
| `config.CONFIG_FILEPATH` | Must match the internal path to the file (`/updater/data/config.json` by default). |

For Specifics the configuration and container usage, please see: https://github.com/zbroad84/ddns-updater/blob/master/README.md#container

---

## Deployment Modes and Conditional Logic (The `if` Statements) 🚦

This chart is designed to deploy either a long-running server or a periodic job. The deployed resources are **conditionally controlled** by the values below, which correspond directly to the `if` statements in the chart templates.

| Key | Default | Controls | Description |
| :--- | :--- | :--- | :--- |
| **`cronjob.enabled`** | `true` | **CronJob** | **Conditional Flag:** If `true`, the application is deployed as a **Kubernetes CronJob** (Update and Exit Mode). Must be `false` for Continuous Server Mode. |
| **`config.SERVER_ENABLED`** | `false` | **Deployment** | **Conditional Flag:** If `true`, the **Deployment** and built-in web server/UI are enabled. This is the primary switch for **Continuous Server Mode**. |
| **`persistence.enabled`** | `true` | **PVC/PV** | **Conditional Flag:** If `true`, a **PVC** and associated **PV** (if using `hostPath`) are created and mounted. |
| **`service.enable_app_service`** | `false` | **Service** | **Conditional Flag:** If `true`, deploys the main application **Service**. Requires `config.SERVER_ENABLED: true`. |
| **`ingress.enabled`** | `false` | **Ingress** | **Conditional Flag:** If `true`, an **Ingress** resource is created. Requires `service.enable_app_service: true`. |

### 1. Update and Exit (Default Mode) ⚙️

This mode deploys a **Kubernetes CronJob** (conditional on **`cronjob.enabled: true`**). The container starts, executes the update, and exits. It's ideal for scheduled, periodic updates.

* **Configuration:**
    ```yaml
    cronjob:
      enabled: true          # Enables the CronJob template
      schedule: "0 * * * *"  # Runs every hour
    config:
      SERVER_ENABLED: false  # Ensures the Deployment template is skipped
    ```
* **Note:** The CronJob uses a `restartPolicy: OnFailure` for the Pod template, which is standard for batch jobs.

### 2. Continuous Server Mode (Web UI) 🌐

This mode deploys a long-lived **Kubernetes Deployment** (conditional on **`config.SERVER_ENABLED: true`**), enabling the Web UI and a continuous background update loop.

* **Configuration:**
    ```yaml
    cronjob:
      enabled: false       # Disables the CronJob template
    config:
      SERVER_ENABLED: true # Enables the Deployment template and web server
    service:
      enable_app_service: true # Expose the web UI
      port: 8000
    ```

---

## Configuration Values

| Key | Default | Description |
| :--- | :--- | :--- |
| **App Configuration** | | |
| `config.PERIOD` | `5m` | The default period for checking the public IP address (used in Deployment mode). |
| `config.SERVER_ENABLED` | `false` | **Conditional Flag:** Enables the Deployment and web server/UI. |
| `config.ROOT_URL` | `/` | URL path to append to all paths for the webUI (used in Ingress path). |
| `config.DATADIR` | `/updater/data` | Internal directory for reading/writing data. This is the PVC mount point. |
| **CronJob** | | |
| `cronjob.enabled` | `true` | **Conditional Flag:** If `true`, deploys the **CronJob** resource. |
| `cronjob.schedule` | `"0 * * * *"` | The cron schedule for the job. Only used when `cronjob.enabled: true`. |
| **Image** | | |
| `replicaCount` | `1` | The number of deployment replicas. Only used when `cronjob.enabled: false`. |
| `image.repository` | `qmcgaw/ddns-updater` | The Docker image repository. |
| `image.tag` | `""` | Overrides the image tag (defaults to chart `appVersion` if empty). |
| **ServiceAccount** | | |
| `serviceAccount.create` | `true` | **Conditional Flag:** If `true`, a **ServiceAccount** resource will be created. |
| `serviceAccount.automount` | `true` | If `true`, automatically mounts the ServiceAccount's API credentials. |
| **Persistence (PVC/PV)** | | |
| `persistence.enabled` | `true` | **Conditional Flag:** If `true`, a **PVC** and (if using `hostPath`) a **PV** will be created. |
| `persistence.storage_capacity` | `5Gi` | The requested storage capacity for the PVC/PV. |
| `persistence.storageClassName` | `manual-hostpath` | The storage class for the PVC/PV. |
| `persistence.hostPath` | `/updater/data` | **Required if using `hostPath` PV.** The path on the node where data will be stored. |
| `persistence.persistentVolumeReclaimPolicy` | `Retain` | The reclaim policy for the PV. |
| **Service** | | |
| `service.enable_app_service` | `false` | **Conditional Flag:** If `true`, deploys the main application **Service**. |
| `service.enable_hc_service` | `false` | **Conditional Flag:** If `true`, deploys a separate **Health Check Service**. |
| `service.type` | `ClusterIP` | Kubernetes service type for both services. |
| `service.port` | `8000` | The application service port. **Must match `config.LISTENING_ADDRESS` port.** |
| `service.health_check_port` | `9999` | The health check service port. |
| **Ingress** | | |
| `ingress.enabled` | `false` | **Conditional Flag:** If `true`, an **Ingress** resource is created. |
| `ingress.host` | `chart-example.local` | The hostname to use for the Ingress. |
| `ingress.className` | `""` | The Ingress class to use (e.g., `nginx`). |
| `ingress.annotations` | `{}` | Annotations to add to the Ingress (e.g., cert-manager setup). |

---

### Additional Configuration Notes

* **Probes and Resources:** `livenessProbe`, `readinessProbe`, and `resources` are defined in `values.yaml` but are **only applied** when running in **Continuous Server Mode** (`cronjob.enabled: false`). They are intentionally omitted from the CronJob template.
* **Public IP Providers:** For configuration options like `config.PUBLICIP_HTTP_PROVIDERS` and others, the default value of `all` uses the complete set of built-in providers. Refer to the application's documentation for a full list of available providers if you wish to restrict them.
* **Node Affinity:** `nodeSelector`, `tolerations`, and `affinity` are available for both Deployment and CronJob templates, allowing you to control where the pods are scheduled.
