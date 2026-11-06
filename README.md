# Platform Ops

A collection of Kubernetes-first infrastructure stacks for local development environments. Most stacks are deployed to Kubernetes with Argo CD and Kustomize; legacy Docker Compose deployments are available for some stacks.

> See below for installation instructions for a k3d cluster with Portainer (or Rancher) and Argo CD.

## Stacks

| Stack | Description |
|-------|-------------|
| [ai-stack](./stacks/ai-stack/README.md) | Local AI engineering environment with Ollama, Open WebUI, Qdrant vector database, and text embeddings; optional Langfuse Compose deployment is legacy |
| [azure-stack](./stacks/azure-stack/README.md) | Azure service emulators including Azurite (Blob/Queue/Table), CosmosDB, Event Hubs, Service Bus, SQL Server, and App Configuration |
| [dapr-stack](./stacks/dapr-stack/README.md) | Dapr self-hosted runtime with placement and scheduler services for distributed application development |
| [elastic-stack](./stacks/elastic-stack/README.md) | Elastic Stack with Elasticsearch, Elastic APM, Kibana, and OpenTelemetry Collector for observability |
| [kafka-stack](./stacks/kafka-stack/README.md) | Apache Kafka environment supporting KRaft and Zookeeper modes with Schema Registry and Kafka UI |
| [openobserve-stack](./stacks/openobserve-stack/README.md) | OpenObserve with OpenTelemetry Collector for observability |
| [postgres-stack](./stacks/postgres-stack/README.md) | PostgreSQL 16 database with pgAdmin web UI for database management |
| [rabbitmq-stack](./stacks/rabbitmq-stack/README.md) | RabbitMQ message broker with management UI and SSL/TLS support |
| [redis-stack](./stacks/redis-stack/README.md) | Redis server with AOF persistence and RedisInsight web UI |
| [dev-certs](./dev-certs/README.md) | Development SSL certificates shared across docker compose stacks |

## Getting Started

**Kubernetes (k3d)**  
Follow the [Kubernetes Installation Guide](./INSTALL.md) to set up Docker, k3d, Portainer (or Rancher), and Argo CD.

> To use the available legacy Docker Compose deployments, follow the [Docker Setup Guide](./docker-setup/README.md) to install and configure Docker. Check each stack's README for supported deployment options.

## Prerequisites

- Docker 24+ and Docker Compose v2

## Usage

Each stack has its own directory and README with deployment instructions. Follow the documented Kubernetes deployment for the primary setup; use Docker Compose only where a stack documents it as an available option.

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file.
