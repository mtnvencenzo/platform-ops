# Platform Ops

A collection of Docker Compose stacks for local development environments. Each stack provides containerized infrastructure services commonly needed for application development and testing.

## Stacks

| Stack | Description |
|-------|-------------|
| [ai-stack](./stacks/ai-stack/README.md) | Local AI engineering environment with Ollama, Open WebUI, Qdrant vector database, Hugging Face Text Embeddings, and Langfuse for LLM observability |
| [azure-stack](./stacks/azure-stack/README.md) | Azure service emulators including Azurite (Blob/Queue/Table), CosmosDB, Event Hubs, Service Bus, SQL Server, and App Configuration |
| [dapr-stack](./stacks/dapr-stack/README.md) | Dapr self-hosted runtime with placement and scheduler services for distributed application development |
| [elastic-stack](./stacks/elastic-stack/README.md) | Elastic Stack with Elasticsearch, Elastic APM, Kibana, and OpenTelemetry Collector for observability |
| [kafka-stack](./stacks/kafka-stack/README.md) | Apache Kafka environment supporting KRaft and Zookeeper modes with Schema Registry and Kafka UI |
| [openobserve-stack](./stacks/openobserve-stack/README.md) | Openobserve with otel collectory for observability |
| [postgres-stack](./stacks/postgres-stack/README.md) | PostgreSQL 16 database with pgAdmin web UI for database management |
| [rabbitmq-stack](./stacks/rabbitmq-stack/README.md) | RabbitMQ message broker with management UI and SSL/TLS support |
| [redis-stack](./stacks/redis-stack/README.md) | Redis server with AOF persistence and RedisInsight web UI |
| [dev-certs](./dev-certs/README.md) | Development SSL certificates shared across docker compose stacks |

## Getting Started

Setup your host machine with docker and a K3d cluster
**Kubernetes (k3d)**  
Follow the [Kubernetes Installation Guide](./INSTALL.md) which includes Docker installation along with k3d, Portainer (or Rancher), and ArgoCD setup.

> If you just want to run the stacks using Docker Compose, follow the [Docker Setup Guide](./docker-setup/README.md) to install and configure Docker.

## Prerequisites

- Docker 24+ and Docker Compose v2

## Usage

Each stack is self-contained with its own `docker-compose.yml` and documentation. Navigate to the stack directory and follow the README instructions to start the services.

## Community & Support

- [Contributing Guide](.github/CONTRIBUTING.md)
- [Code of Conduct](.github/CODE_OF_CONDUCT.md)
- [Support Guide](.github/SUPPORT.md)
- [Security Policy](.github/SECURITY.md)

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file.
