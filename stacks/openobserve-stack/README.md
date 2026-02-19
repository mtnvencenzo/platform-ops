# OpenObserve Stack w/OpenTelemetry Collector

This stack provides a Docker Compose setup for OpenObserve and the OpenTelemetry Collector. It enables you to collect, process, and forward traces, metrics, and logs from your applications to OpenObserve for observability and analysis.

## 📁 Contents

- **docker-compose.yml**: Orchestrates OpenObserve and the OpenTelemetry Collector.
- **otel-collector-config.yml**: Configuration for the OpenTelemetry Collector, specifying receivers, processors, and exporters (to OpenObserve).
- **README.md**: This documentation file.

## ⚙️ Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/) installed on your machine.

## 🚀 Setup & Usage

> This setup is intended for local development and testing. For production, review and adjust security and resource settings.

### 1. Start the OpenObserve stack (Docker Compose):

```bash
docker compose -f docker-compose.yml up -d

# Or if the containers have already been created
docker compose -f docker-compose.yml start
```

This will start OpenObserve and the OpenTelemetry Collector using the provided configuration, grouping the containers under the project name `openobserve-stack`.

To bring the stack down:

```bash
docker compose -f docker-compose.yml down -v
```

To force a rebuild and deploy of an individual container:

```bash
docker compose up -d --force-recreate --no-deps --build <service_name>
```

### 2. Deploy to Kubernetes with Argo CD (GitOps)

If you use Kubernetes and Argo CD, deploy the OpenObserve stack using the provided Argo CD Application manifest:

1. Ensure Argo CD is installed in your cluster and you have access to the `argocd` namespace.
2. Apply the Argo CD Application manifest:
	 ```bash
	 kubectl apply -f argocd/openobserve-stack-app.yaml
	 ```
	 This will create an Argo CD Application that manages the deployment of OpenObserve. Argo CD will create the required namespace and keep your deployment in sync with the manifests in this repository.

3. Monitor the deployment in the Argo CD UI or with:
	 ```bash
	 kubectl -n argocd get applications
	 ```

#### Notes

- The Kubernetes manifests are in `k8s/` and managed by Kustomize.
- The Argo CD Application manifest is in `argocd/openobserve-stack-app.yaml`.

### 3. Accessing OpenObserve

- **Web UI**: http://localhost:5080
- Default credentials (see `docker-compose.yml`):
	- Email: admin@example.com
	- Password: ComplexPassword#123

> Adjust ports and credentials as needed for your environment.

### 4. Send Telemetry Data

- Point your application(s) to the OpenTelemetry Collector endpoint (as defined in `otel-collector-config.yml`):
	- http://localhost:4317 (outside the compose network)
	- http://otel-collector:4317 (inside the compose network)
- The collector will receive, process, and forward telemetry data to OpenObserve.

**Monitor in OpenObserve:**
- Log in to the OpenObserve UI to view traces, metrics, and logs.

## 🛠️ Customization

- Modify `otel-collector-config.yml` to add or remove receivers, processors, or exporters as needed.
- Refer to the [OpenTelemetry Collector documentation](https://opentelemetry.io/docs/collector/configuration/) and [OpenObserve documentation](https://openobserve.ai/docs/) for advanced configuration.

## 🐞 Troubleshooting

- Check container logs for errors:
	```bash
	docker compose -f docker-compose.yml logs
	```
- Ensure network connectivity between the collector and OpenObserve.
- Validate your credentials and endpoint URLs in the configuration file.

---

For more information, see the official documentation for [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/) and [OpenObserve](https://openobserve.ai/docs/).

---
