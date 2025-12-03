# EV Charging Sandbox - Kafka Event Streaming

This directory contains a unified Docker Compose configuration that sets up a complete EV Charging sandbox environment with **Kafka event streaming**: ONIX adapters (BAP and BPP), mock services (CDS, Registry, BAP, BPP), Kafka broker, and supporting infrastructure.

## Architecture Overview

This setup creates a fully functional sandbox environment for testing and developing EV Charging network integrations using the ONIX protocol with **Apache Kafka** for high-throughput event streaming. The architecture includes:

- **ONIX Adapters**: Protocol adapters for BAP (Buyer App Provider) and BPP (Buyer Platform Provider) with Kafka integration
- **Apache Kafka (KRaft)**: Event streaming platform for distributed message processing
- **Mock Services**: Simulated services for testing without real implementations
- **Supporting Services**: Redis for caching and state management

## Services

### Core Services

1. **kafka** (Ports: 9092, 9093)
   - Apache Kafka broker in KRaft mode (no ZooKeeper dependency)
   - Event streaming platform for high-throughput messaging
   - Auto-creates topics as needed

2. **redis-bap** (Port: 6379)
   - Redis cache for the BAP adapter
   - Used for storing transaction state, caching registry lookups, and session management

3. **redis-bpp** (Port: 6380)
   - Redis cache for the BPP adapter
   - Used for storing transaction state, caching registry lookups, and session management

4. **onix-bap-plugin-kafka** (Port: 8001)
   - ONIX protocol adapter for BAP (Buyer App Provider) with Kafka integration
   - Handles protocol compliance, signing, validation, and routing for BAP transactions
   - **bapTxnCaller**: Kafka consumer that consumes requests from BAP Backend and routes them to BPP via HTTP
   - **bapTxnReceiver**: HTTP handler that receives callbacks from BPP and publishes them to BAP Backend via Kafka
   - **HTTP Endpoint**: `/bap/receiver/` - Receives callbacks from BPP adapter

5. **onix-bpp-plugin-kafka** (Port: 8002)
   - ONIX protocol adapter for BPP (Buyer Platform Provider) with Kafka integration
   - Handles protocol compliance, signing, validation, and routing for BPP transactions
   - **bppTxnCaller**: Kafka consumer that consumes callbacks from BPP Backend and routes them to BAP/CDS via HTTP
   - **bppTxnReceiver**: HTTP handler that receives requests from BAP adapter and publishes them to BPP Backend via Kafka
   - **HTTP Endpoint**: `/bpp/receiver/` - Receives requests from BAP adapter

### Mock Services

6. **mock-registry** (Port: 3030)
   - Mock implementation of the network registry service
   - Maintains a registry of all BAPs, BPPs, and CDS services on the network
   - Provides subscriber lookup and key management functionality

7. **mock-cds** (Port: 8082)
   - Mock Catalog Discovery Service (CDS)
   - Aggregates discover requests from BAPs and broadcasts to registered BPPs
   - Collects and aggregates responses from multiple BPPs
   - Handles signature verification and signing

8. **mock-bap-kafka** (Port: 9001)
   - Mock BAP backend service with Kafka integration
   - Simulates a Buyer App Provider application
   - Consumes callbacks from Kafka topics (`bap.on_*`)
   - Publishes requests to Kafka topics (`bap.*`)

9. **mock-bpp-kafka** (Port: 9002)
   - Mock BPP backend service with Kafka integration
   - Simulates a Buyer Platform Provider application
   - Consumes requests from Kafka topics (`bpp.*`)
   - Publishes callbacks to Kafka topics (`bpp.on_*`)

## Configuration Files

Each service has its own configuration file with a service name prefix. This section explains what each config file is used for:

### 1. ONIX BAP Configuration (`onix-adaptor/kafka/config/onix-bap/`)

**Purpose**: Complete adapter configuration for the ONIX BAP plugin with Kafka integration.

**Usage**: The actual configuration files are mounted from `../../onix-adaptor/kafka/config/onix-bap/`. This directory contains:

- **`adapter.yaml`**: Main adapter configuration file
- **`bapTxnCaller-routing.yaml`**: Routing rules for outgoing requests from BAP application (consumed from Kafka)
- **`bapTxnReciever-routing.yaml`**: Routing rules for incoming callbacks to BAP (published to Kafka)

**Key Sections in `adapter.yaml`**:
- **`appName`**: Application identifier ("onix-ev-charging")
- **`log`**: Logging configuration (level, destinations, context keys)
- **`http`**: HTTP server settings (port: 8001, timeouts) - Required for bapTxnReceiver
- **`modules`**: Two main modules:
  - **`bapTxnReceiver`**: Handles incoming callbacks from CDS/BPPs via HTTP
    - Validates signatures
    - Publishes to Kafka topics (`bap.on_*`)
    - Validates schemas
  - **`bapTxnCaller`**: Consumes requests from Kafka topics (`bap.*`)
    - Routes requests using `bapTxnCaller-routing.yaml`
    - Signs requests
    - Routes to BPP/CDS via HTTP
- **`plugins`**: Plugin configuration including:
  - Registry connection to mock-registry
  - Key manager for signing/encryption
  - Redis cache configuration
  - Schema validator
  - Router for request routing
  - Kafka consumer and producer configurations

**Kafka Topics**:
- **Consumer Topics** (bapTxnCaller): `bap.discover`, `bap.select`, `bap.init`, `bap.confirm`, etc.
- **Producer Topics** (bapTxnReceiver): `bap.on_discover`, `bap.on_select`, `bap.on_init`, `bap.on_confirm`, etc.

**Note**: The actual configs are in `onix-adaptor/kafka/config/onix-bap/`.

### 2. ONIX BPP Configuration (`onix-adaptor/kafka/config/onix-bpp/`)

**Purpose**: Complete adapter configuration for the ONIX BPP plugin with Kafka integration.

**Usage**: The actual configuration files are mounted from `../../onix-adaptor/kafka/config/onix-bpp/`. This directory contains:

- **`adapter.yaml`**: Main adapter configuration file
- **`bppTxnCaller-routing.yaml`**: Routing rules for outgoing responses from BPP (consumed from Kafka)
- **`bppTxnReciever-routing.yaml`**: Routing rules for incoming requests to BPP (published to Kafka)

**Key Sections in `adapter.yaml`**:
- **`appName`**: Application identifier ("bpp-ev-charging")
- **`log`**: Logging configuration
- **`http`**: HTTP server settings (port: 8002, timeouts) - Required for bppTxnReceiver
- **`modules`**: Two main modules:
  - **`bppTxnReceiver`**: Handles incoming requests from CDS/BAPs via HTTP
    - Validates signatures
    - Publishes to Kafka topics (`bpp.*`)
    - Validates schemas
  - **`bppTxnCaller`**: Consumes callbacks from Kafka topics (`bpp.on_*`)
    - Routes responses using `bppTxnCaller-routing.yaml`
    - Signs responses
    - Routes to BAP/CDS via HTTP
- **`plugins`**: Similar plugin configuration as BAP, but for BPP role

**Kafka Topics**:
- **Consumer Topics** (bppTxnCaller): `bpp.on_discover`, `bpp.on_select`, `bpp.on_init`, `bpp.on_confirm`, etc.
- **Producer Topics** (bppTxnReceiver): `bpp.discover`, `bpp.select`, `bpp.init`, `bpp.confirm`, etc.

**Note**: The actual configs are in `onix-adaptor/kafka/config/onix-bpp/`.

### 3. `mock-registry_config.yml`

**Purpose**: Configuration for the mock registry service that maintains the network participant registry.

**Usage**: Mounted into the mock-registry container at `/app/config/config.yaml`.

**Key Sections**:
- **`server.port`**: Server port (3030)
- **`subscribers`**: List of all network participants (same as HTTP sandbox)
- **`defaults`**: Validity period for registry entries

### 4. `mock-cds_config.yml`

**Purpose**: Configuration for the mock Catalog Discovery Service that aggregates discover requests.

**Usage**: Mounted into the mock-cds container at `/app/config/config.yaml`.

**Key Sections**: Same as HTTP sandbox, but CDS communicates with BPPs via HTTP (not Kafka).

### 5. `mock-bap-kafka_config.yml`

**Purpose**: Configuration for the mock BAP backend service with Kafka integration.

**Usage**: Mounted into the mock-bap-kafka container at `/app/config/config.yaml`.

**Key Sections**:
- **`server.port`**: Server port (9001)
- **`kafka`**: Kafka broker configuration
- **`topics`**: Kafka topics to consume from (`bap.on_*`) and produce to (`bap.*`)

### 6. `mock-bpp-kafka_config.yml`

**Purpose**: Configuration for the mock BPP backend service with Kafka integration.

**Usage**: Mounted into the mock-bpp-kafka container at `/app/config/config.yaml`.

**Key Sections**:
- **`server.port`**: Server port (9002)
- **`kafka`**: Kafka broker configuration
- **`topics`**: Kafka topics to consume from (`bpp.*`) and produce to (`bpp.on_*`)

## Quick Start

### Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- Access to Docker images (pulled automatically)

### Starting All Services

```bash
# Navigate to this directory
cd sandbox/kafka

# Start all services
docker compose up -d

# View logs
docker compose logs -f

# Check service status
docker compose ps
```

### Stopping Services

```bash
# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v
```

## Service Endpoints

Once all services are running, you can access:

| Service | Endpoint | Description |
|---------|----------|-------------|
| **ONIX BAP** | `http://localhost:8001` | BAP adapter HTTP endpoint |
| | `http://localhost:8001/bap/receiver/{action}` | Receive callbacks from BPP |
| **ONIX BPP** | `http://localhost:8002` | BPP adapter HTTP endpoint |
| | `http://localhost:8002/bpp/receiver/{action}` | Receive requests from BAP |
| **Kafka** | `localhost:9092` | Kafka broker (internal) |
| **Mock Registry** | `http://localhost:3030` | Registry service |
| | `http://localhost:3030/lookup` | Lookup subscribers |
| **Mock CDS** | `http://localhost:8082` | Catalog Discovery Service |
| | `http://localhost:8082/csd` | CDS aggregation endpoint |
| **Mock BAP** | `http://localhost:9001` | Mock BAP backend |
| **Mock BPP** | `http://localhost:9002` | Mock BPP backend |

## Message Flow

### Phase 1: Discovery Flow (Kafka + HTTP)

1. **BAP Backend** publishes `discover` request to Kafka topic `bap.discover`
2. **ONIX BAP plugin's bapTxnCaller** consumes from Kafka, processes, and routes to **Mock CDS** via HTTP
3. **Mock CDS** broadcasts to all registered BPPs via HTTP
4. **ONIX BPP plugin's bppTxnReceiver** receives HTTP request, validates, and publishes to Kafka topic `bpp.discover`
5. **Mock BPP Backend** consumes from `bpp.discover`, processes, and publishes response to `bpp.on_discover`
6. **ONIX BPP plugin's bppTxnCaller** consumes from `bpp.on_discover`, processes, and routes to **Mock CDS** via HTTP
7. **Mock CDS** aggregates responses and sends to **ONIX BAP plugin's bapTxnReceiver** via HTTP
8. **ONIX BAP plugin's bapTxnReceiver** receives HTTP callback, validates, and publishes to Kafka topic `bap.on_discover`
9. **BAP Backend** consumes aggregated response from `bap.on_discover`

### Phase 2+: Transaction Flow (Kafka + HTTP)

1. **BAP Backend** publishes transaction request (select, init, confirm, etc.) to Kafka topic `bap.*`
2. **ONIX BAP plugin's bapTxnCaller** consumes from Kafka, processes, and routes directly to **ONIX BPP** via HTTP (bypasses CDS)
3. **ONIX BPP plugin's bppTxnReceiver** receives HTTP request, validates, and publishes to Kafka topic `bpp.*`
4. **Mock BPP Backend** consumes from `bpp.*`, processes, and publishes response to `bpp.on_*`
5. **ONIX BPP plugin's bppTxnCaller** consumes from `bpp.on_*`, processes, and routes to **ONIX BAP** via HTTP
6. **ONIX BAP plugin's bapTxnReceiver** receives HTTP callback, validates, and publishes to Kafka topic `bap.on_*`
7. **BAP Backend** consumes response from `bap.on_*`

## Testing with Kafka Messages

This sandbox includes pre-formatted JSON messages and bash scripts for testing. See the message directories:

- **BAP Messages**: `message/bap/` - Contains example JSON files and publish scripts
  - See [BAP Message README](./message/bap/README.md) for details
- **BPP Messages**: `message/bpp/` - Contains example JSON files and publish scripts
  - See [BPP Message README](./message/bpp/README.md) for details

### Quick Test Example

```bash
# Publish a discover request from BAP Backend
cd sandbox/kafka/message/bap/test
./publish-discover-by-station.sh

# Check logs to see the flow
docker compose logs -f onix-bap-plugin-kafka onix-bpp-plugin-kafka
```

## Kafka Management

### Using Kafka CLI Tools

```bash
# List topics
docker exec kafka kafka-topics.sh --list --bootstrap-server localhost:9092

# Create a topic manually
docker exec kafka kafka-topics.sh --create --topic bap.discover --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

# Describe a topic
docker exec kafka kafka-topics.sh --describe --topic bap.discover --bootstrap-server localhost:9092

# Consume messages from a topic
docker exec kafka kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic bap.discover --from-beginning

# Produce messages to a topic
docker exec -it kafka kafka-console-producer.sh --bootstrap-server localhost:9092 --topic bap.discover
```

### Consumer Group Management

```bash
# List consumer groups
docker exec kafka kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# Describe consumer group
docker exec kafka kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group bap_caller_group
```

## Network Architecture

All services run on a shared Docker network (`onix-network`) allowing:
- Service-to-service communication using container names as hostnames
- Isolated networking from other Docker containers
- Easy service discovery without IP address management

## Customization

### Adding New BPP Endpoints

1. Edit `mock-cds_config.yml` to add BPP endpoints
2. Add the BPP to `mock-registry_config.yml`
3. Restart services: `docker compose restart mock-cds mock-registry`

### Modifying ONIX Adapter Routing

To change how requests/responses are routed through the ONIX adapters, edit the routing configuration files:

1. **For BAP routing changes**, edit `onix-adaptor/kafka/config/onix-bap/`:
   - `bapTxnCaller-routing.yaml` - Modify outgoing request routing
   - `bapTxnReciever-routing.yaml` - Modify incoming callback routing

2. **For BPP routing changes**, edit `onix-adaptor/kafka/config/onix-bpp/`:
   - `bppTxnCaller-routing.yaml` - Modify outgoing response routing
   - `bppTxnReciever-routing.yaml` - Modify incoming request routing

3. **Restart the adapter services**:
   ```bash
   docker compose restart onix-bap-plugin-kafka onix-bpp-plugin-kafka
   ```

## Troubleshooting

### Service Won't Start

1. **Check ports are available**:
   ```bash
   lsof -i :8001  # BAP
   lsof -i :8002  # BPP
   lsof -i :9092  # Kafka
   lsof -i :3030  # Registry
   ```

2. **Check container logs**:
   ```bash
   docker compose logs <service-name>
   ```

### Kafka Connection Issues

- Verify Kafka is running: `docker ps | grep kafka`
- Check Kafka logs: `docker logs kafka`
- Verify network connectivity: Ensure plugin containers can reach `kafka:9092`

### Messages Not Being Consumed

- Check consumer is running: `docker ps | grep onix-.*-plugin-kafka`
- Verify topics exist: `docker exec kafka kafka-topics.sh --list --bootstrap-server localhost:9092`
- Check adapter logs: `docker logs onix-bap-plugin-kafka` or `docker logs onix-bpp-plugin-kafka`
- Verify consumer group status: `docker exec kafka kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group <group-name>`

## File Structure

```
sandbox/kafka/
├── docker-compose.yml              # Unified compose file for all services
├── README.md                        # This file
├── mock-registry_config.yml         # Mock registry configuration
├── mock-cds_config.yml              # Mock CDS configuration
├── mock-bap-kafka_config.yml       # Mock BAP backend configuration
├── mock-bpp-kafka_config.yml       # Mock BPP backend configuration
└── message/                         # Test messages and scripts
    ├── bap/
    │   ├── example/                 # BAP request JSON files
    │   ├── test/                    # BAP publish scripts
    │   └── README.md
    └── bpp/
        ├── example/                 # BPP callback JSON files
        ├── test/                    # BPP publish scripts
        └── README.md

# Actual ONIX adapter configurations (mounted from parent directory)
onix-adaptor/kafka/config/
├── onix-bap/
│   ├── adapter.yaml                 # Main BAP adapter configuration
│   ├── bapTxnCaller-routing.yaml    # BAP caller routing rules
│   └── bapTxnReciever-routing.yaml  # BAP receiver routing rules
└── onix-bpp/
    ├── adapter.yaml                 # Main BPP adapter configuration
    ├── bppTxnCaller-routing.yaml    # BPP caller routing rules
    └── bppTxnReciever-routing.yaml  # BPP receiver routing rules
```

## Additional Resources

- [ONIX Protocol Documentation](https://github.com/beckn/onix)
- [BAP/BPP Specification](https://github.com/beckn/protocol-specifications)
- [Kafka Integration Guide](../../onix-adaptor/kafka/README.md) - Detailed ONIX adapter Kafka integration documentation
- [HTTP/REST Sandbox Guide](../api/README.md) - Alternative sandbox using HTTP/REST communication

## Notes

- The ONIX adapter configuration directory structure includes:
  - `adapter.yaml`: Main adapter configuration
  - `{bap|bpp}TxnCaller-routing.yaml`: Routing rules for outgoing requests/responses (consumed from Kafka)
  - `{bap|bpp}TxnReciever-routing.yaml`: Routing rules for incoming requests/responses (published to Kafka)
- Volume mounts:
  - Config directory: `/app/config/message-baised/kafka/onix-{bap|bpp}` (internal container path)
  - Schema directory: `/app/schemas` (read-only, from root `schemas/` directory)
- All mock services use simplified configurations suitable for testing.
- Production deployments should use proper key management and secure configurations.
- Network service discovery uses Docker container names, which must match the service names in configuration files.
- Kafka topics are auto-created when first message is published (if auto-creation is enabled).

