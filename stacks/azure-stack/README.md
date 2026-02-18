# Azure Stack - Local Development Environment

This stack provides a Docker Compose setup for running Azure service emulators locally, giving you a complete Azure development environment without needing cloud resources. Perfect for local development, testing, and CI/CD pipelines.

![Azure Stack Architecture](./assets/azure-stack.drawio.svg)

## 📁 Contents

- **docker-compose.yml**: Docker Compose configuration for Azure service emulators
- **volumes/**: Persistent data directories for Azurite and CosmosDB
- **assets/**: Documentation and setup guides for individual services
- **README.md**: This documentation file

## ⚙️ Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/) installed on your machine

## 🏗️ Architecture

This setup provides the following Azure service emulators:

- **[Azurite](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite)** - Azure Storage emulator for Blob, Queue, and Table storage (ports 10000-10002)
- **[CosmosDB Emulator](https://learn.microsoft.com/en-us/azure/cosmos-db/emulator-linux)** - Azure Cosmos DB emulator for NoSQL database development (ports 8081, 1234)
- **[Event Hubs Emulator](https://learn.microsoft.com/en-us/azure/event-hubs/overview-emulator)** - Azure Event Hubs emulator for event streaming (ports 5672, 9092, 5300)
- **[Service Bus Emulator](https://github.com/Azure/azure-service-bus-emulator-installer)** - Azure Service Bus emulator for messaging (port 5671)
- **[SQL Server 2022](https://hub.docker.com/_/microsoft-mssql-server)** - Microsoft SQL Server database for Service Bus dependencies (port 1434)
- **[App Configuration Emulator](https://learn.microsoft.com/en-us/azure/azure-app-configuration/overview)** - Azure App Configuration emulator for centralized configuration (port 8483)

### Service Dependencies

The services have the following dependencies to ensure proper startup order:

- **Event Hubs** depends on **Azurite** - Event Hubs uses Azurite for metadata and blob storage ([Official Template](https://github.com/Azure/azure-event-hubs-emulator-installer/blob/main/Docker-Compose-Template/docker-compose-default.yml))
- **Service Bus** depends on **SQL Server** - Service Bus requires SQL Server for persistence and message storage ([Official Template](https://github.com/Azure/azure-service-bus-emulator-installer/blob/main/Docker-Compose-Template/docker-compose-default.yml))

### Data Persistence

Based on official Microsoft documentation:

- **✅ Azurite**: Persists data to `./volumes/azurite-data` 
- **✅ CosmosDB**: Persists data to `./volumes/cosmosdb-data`
- **✅ SQL Server**: Persists data to `./volumes/sqlserver-data`
- **✅ App Configuration**: Persists data protection keys to `./volumes/appconfig-data`
- **❌ Event Hubs**: No persistence - [data doesn't persist by design](https://learn.microsoft.com/en-us/azure/event-hubs/overview-emulator#known-limitations)
- **❌ Service Bus**: No file persistence - uses SQL Server for storage

> **Note**: SQL Server was included as a dependency for the Service Bus emulator, but it's a fully functioning SQL Server 2022 Express instance that can be used for any development needs.

All containers run within a dedicated `azure-network` bridge network for secure inter-service communication.

## 🚀 Setup & Usage

> This setup is designed for local development and testing. Do not use in production environments.

### 1. Start the Azure service emulators:

    ```bash
    docker compose -f docker-compose.yml up -d

    # Or if the containers have already been created
    docker compose -f docker-compose.yml start
    ```

### 2. Stop the services:
    ```bash
    docker compose -f docker-compose.yml down -v
    ```

### 3. Rebuild and restart a specific service:
    ```bash
    docker compose -f docker-compose.yml up -d --force-recreate --no-deps --build <service_name>
    ```

### 4. heck all services status:
    ```bash
    docker compose ps
    ```

## 🛠️ Customization

- Modify `docker-compose.yml` to adjust service configurations, ports, or resource limits as needed for your environment.
- Update volume mounts in `volumes/` directory for persistent data storage requirements.
- See individual service documentation in the `assets/` folder for detailed configuration options.

## 📊 Service Endpoints & Ports

### Azure Storage (Azurite)
- **Blob Storage**: `localhost:10000` (HTTP)
- **Queue Storage**: `localhost:10001` (HTTP)
- **Table Storage**: `localhost:10002` (HTTP)
- **Account Name**: `devstoreaccount1`
- **Account Key**: `Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==`

**Connection String:**
```
# Docker compose
DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;TableEndpoint=http://127.0.0.1:10002/devstoreaccount1;

# K8s Cluster
DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://azure-blob.127.0.0.1.sslip.io:8080/devstoreaccount1;QueueEndpoint=http://azure-queue.127.0.0.1.sslip.io:8080/devstoreaccount1;TableEndpoint=http://azure-table.127.0.0.1.sslip.io:8080/devstoreaccount1;
```

**Test Connection:**
```bash
# Test blob storage endpoint
curl http://localhost:10000/devstoreaccount1?comp=list

# Test all endpoints
curl http://localhost:10000  # Blob
curl http://localhost:10001  # Queue  
curl http://localhost:10002  # Table
```

> Note: When adding blob containers for pulic access `PublicAccessType` = `BlobContainer or Container` you might have manually use the storage explorer to set the public access type *(even when setting the access type in code when creating the container)* 

### Azure CosmosDB Emulator
- **CosmosDB Endpoint**: `localhost:8081` (HTTPS)
- **Data Explorer**: `localhost:1234` (HTTP)
- **Account Key**: `C2y6yDjf5R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

**Connection String:**
```
AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==;
```

**Test Connection:**
```bash
# Test CosmosDB certificate endpoint
curl -k https://localhost:8081/_explorer/emulator.pem

# Test Data Explorer (Web UI)
curl http://localhost:1234

# Test direct API endpoint
curl -k https://localhost:8081
```

### Azure Event Hubs Emulator
- **AMQP Endpoint**: `localhost:5672` (TCP)
- **Kafka Endpoint**: `localhost:9092` (TCP)
- **Health Endpoint**: `localhost:5300` (HTTP)
- **Protocols**: AMQP 1.0, Kafka
- **Default Event Hub**: Available after first connection

**Connection String:**
```
Endpoint=sb://localhost:5672/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=SAS_KEY_VALUE;UseDevelopmentEmulator=true;
```

**Test Connection:**
```bash
# Test health endpoint
curl http://localhost:5300/health

# Test AMQP endpoint connectivity
curl http://localhost:5672

# Check container logs for startup confirmation
docker compose logs eventhubs
```

### Azure Service Bus Emulator
- **AMQP Endpoint**: `localhost:5671` (TCP)
- **Protocol**: AMQP 1.0
- **Default Namespace**: Available after first connection

**Connection String:**
```
Endpoint=sb://localhost:5671/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=SAS_KEY_VALUE;UseDevelopmentEmulator=true;
```

**Note**: Service Bus depends on SQL Server being healthy, which can take a few minutes to initialize the first time it starts due to the upgrade process. If Service Bus doesn't start automatically, run: `docker compose up -d servicebus` after SQL Server is ready.


**Test Connection:**
```bash
# Test AMQP endpoint connectivity
curl http://localhost:5671

# Check if Service Bus is connected to SQL Server
docker compose logs servicebus | grep -i "successfully"

# Verify SQL Server dependency is healthy
docker compose ps sqlserver
```

### SQL Server 2022 Express
- **Database Endpoint**: `localhost:1434` (TCP)
- **Username**: `sa`
- **Password**: `StrongPassword123!`
- **Connection String**: `Server=localhost,1434;Database=master;User Id=sa;Password=StrongPassword123!;TrustServerCertificate=true;`
- **Purpose**: Required dependency for Service Bus emulator, but fully functional for development use
- **Note**: External port 1434 is used to avoid conflicts with existing SQL Server instances on port 1433

**Test Connection:**
```bash
# Test connection using docker exec
docker compose exec sqlserver /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P StrongPassword123! -C -Q 'SELECT 1'

# Check health status
docker compose ps sqlserver

# Test external connection (requires sqlcmd installed locally)
sqlcmd -S localhost,1434 -U sa -P StrongPassword123! -C -Q "SELECT @@VERSION"
```

### Azure App Configuration Emulator
- **HTTP Endpoint**: `localhost:8483` (HTTP)
- **Web UI**: `http://localhost:8483` (Management interface with anonymous authentication)
- **REST API**: `http://localhost:8483/kv` (Key-value operations)
- **Authentication**: Anonymous authentication enabled for local development
- **Features**: Complete App Configuration functionality including:
  - Configuration key-value storage and retrieval
  - Feature flag management
  - Web-based management interface with data persistence
  - Full REST API for client SDK integration
  - Anonymous authentication (no credentials required)

**Connection String:**
```
Endpoint=http://localhost:8483
```

**Test Connection:**
```bash
# Test Web UI (now accessible without authentication)
curl http://localhost:8483

# Test REST API endpoint (now returns data instead of 401)
curl -i http://localhost:8483/kv

# Create a test configuration
curl -X PUT "http://localhost:8483/kv/TestKey" \
  -H "Content-Type: application/json" \
  -d '{"value": "TestValue"}'

# Check container logs for startup confirmation
docker compose logs appconfig | grep "Now listening"
```

## 🐞 Troubleshooting

- Check container logs for errors:
  ```bash
  docker compose logs
  
  # Or for a specific service
  docker compose logs azurite
  docker compose logs cosmosdb
  docker compose logs eventhubs
  docker compose logs servicebus
  docker compose logs sqlserver
  docker compose logs appconfig
  ```
- Ensure no port conflicts with existing services on your machine.
- Validate that Docker has sufficient resources allocated for all containers.
- Check that volumes have proper permissions for data persistence.
- For CosmosDB certificate issues, see the [CosmosDB setup guide](./assets/readme-cosmos.md).
- For Azurite connection issues, see the [Azurite setup guide](./assets/readme-azurite.md).

## ⚙️ Configuration Insights

### Volume Mount Best Practices
Based on official Microsoft documentation and container analysis:

- **Event Hubs**: No data volumes needed - data doesn't persist by design ([limitation](https://learn.microsoft.com/en-us/azure/event-hubs/overview-emulator#known-limitations))
- **Service Bus**: No data volumes needed - uses SQL Server for all persistence  
- **App Configuration**: Mounts `/home/app/.aspnet` for data protection keys persistence
- **SQL Server**: External port `1434` to avoid conflicts with existing instances

### Startup Dependencies
- **Service Bus** waits up to 60 seconds for SQL Server health check (`SQL_WAIT_INTERVAL=60`)
- **Event Hubs** requires Azurite for metadata storage and blob operations
- **Health checks** ensure proper startup order but may require manual intervention for Service Bus if SQL Server takes longer than expected

### Port Configuration  
- All services use standard internal ports within Docker network
- External ports can be customized to avoid conflicts with existing services
- App Configuration provides complete functionality (Web UI + REST API) on single port 8483

## 📚 Additional Resources

### Official Microsoft Documentation
- **[Azurite](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite)** - Azure Storage emulator documentation
- **[CosmosDB Emulator](https://learn.microsoft.com/en-us/azure/cosmos-db/emulator-linux)** - Linux container emulator guide
- **[Event Hubs Emulator](https://learn.microsoft.com/en-us/azure/event-hubs/overview-emulator)** - Event Hubs emulator overview
- **[Event Hubs Testing Guide](https://learn.microsoft.com/en-us/azure/event-hubs/test-locally-with-event-hub-emulator)** - Local testing documentation
- **[Service Bus Emulator](https://github.com/Azure/azure-service-bus-emulator-installer)** - GitHub repository and setup
- **[App Configuration](https://learn.microsoft.com/en-us/azure/azure-app-configuration/overview)** - App Configuration service overview
- **[SQL Server 2022](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-overview)** - SQL Server on Linux

### Docker Templates & Configuration
- **[Event Hubs Docker Template](https://github.com/Azure/azure-event-hubs-emulator-installer/blob/main/Docker-Compose-Template/docker-compose-default.yml)** - Official Microsoft compose file
- **[Service Bus Docker Template](https://github.com/Azure/azure-service-bus-emulator-installer/blob/main/Docker-Compose-Template/docker-compose-default.yml)** - Official Microsoft compose file
- **[SQL Server Container](https://hub.docker.com/_/microsoft-mssql-server)** - Official SQL Server container

### Tools & Utilities  
- **[Azure Storage Explorer](https://azure.microsoft.com/en-us/products/storage/storage-explorer)** - GUI for Azure Storage (works with Azurite)
- **[Azure Data Studio](https://docs.microsoft.com/en-us/sql/azure-data-studio/)** - Cross-platform database tool for SQL Server

### Configuration References
- **[Event Hubs Config](https://github.com/Azure/azure-event-hubs-emulator-installer/blob/main/EventHub-Emulator/Config/Config.json)** - Default Event Hubs configuration
- **[Service Bus Config](https://github.com/Azure/azure-service-bus-emulator-installer/blob/main/ServiceBus-Emulator/Config/Config.json)** - Default Service Bus configuration

---
For more information about Azure service emulators, see the official [Azure documentation](https://docs.microsoft.com/azure/).
