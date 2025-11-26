# UBC EV Charging Sandbox

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-Supported-blue.svg)](https://www.docker.com/)

**A comprehensive sandbox environment for testing and developing EV Charging network integrations using the ONIX protocol adapter (Beckn-ONIX) with BAP and BPP applications**

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Quick Start](#quick-start)
- [Architecture](#architecture)
- [Repository Structure](#repository-structure)
- [Configuration](#configuration)
- [Usage Examples](#usage-examples)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [Support](#support)
- [License](#license)

---

## Overview

This repository provides a complete sandbox environment for integrating the **[onix-adapter](https://github.com/Beckn-One/beckn-onix)** (also known as Beckn-ONIX) with **BAP (Buyer App Provider)** and **BPP (Buyer Platform Provider)** applications for EV Charging networks.

### What is Onix-Adapter?

The **onix-adapter** is a production-ready, plugin-based middleware adapter for the Beckn Protocol. It acts as a protocol adapter between Beckn Application Platforms (BAPs) and Beckn Provider Platforms (BPPs), ensuring secure, validated, and compliant message exchange across various commerce networks.

### Key Concepts

- **BAP (Beckn Application Platform)**: Buyer-side applications that help users search for and purchase products/services (e.g., consumer apps, aggregators)
- **BPP (Beckn Provider Platform)**: Seller-side platforms that provide products/services (e.g., merchant platforms, service providers)
- **Onix-Adapter**: Middleware that handles protocol compliance, message signing, validation, and routing between BAPs and BPPs
- **CDS (Catalog Discovery Service)**: Aggregates discover requests from BAPs and broadcasts to registered BPPs
- **Registry**: Maintains a registry of all network participants (BAPs, BPPs, CDS)

---

## Features

### 🚀 Complete Sandbox Environment

- **Full Testing Environment**: Pre-configured complete sandbox with ONIX adapters, mock services, and infrastructure
- **Standalone Adapters**: Deploy only ONIX adapters for integration with your own services
- **Mock Services**: Simulated BAP, BPP, CDS, and Registry services for testing

### 🏗️ Architecture

- **REST API Communication**: Synchronous HTTP/REST communication for real-time interactions
- **Redis Caching**: Performance optimization and state management
- **Docker Compose**: Easy local development and testing setup

### 🔐 Enterprise-Ready

- **Ed25519 Digital Signatures**: Cryptographically secure message signing and validation
- **JSON Schema Validation**: Ensures protocol compliance using schemas from `schemas/` directory
- **Configurable Routing**: YAML-based routing rules
- **Structured Logging**: JSON-formatted logs with transaction tracking

### 📊 Production Features

- **Health Checks**: Liveness and readiness probes for all services
- **Environment-Specific Configs**: Separate configurations for different services
- **API Collections**: Postman collections and Swagger specifications for all APIs

---

## Quick Start

### Prerequisites

- Docker Engine 20.10 or higher
- Docker Compose 2.0 or higher
- Access to onix-adapter Docker images:
  - `manendrapalsingh/onix-bap-plugin:latest`
  - `manendrapalsingh/onix-bpp-plugin:latest`
  - `manendrapalsingh/mock-bap:latest`
  - `manendrapalsingh/mock-bpp:latest`
  - `manendrapalsingh/mock-cds:latest`
  - `manendrapalsingh/mock-registry:latest`

### Recommended Starting Point

#### Option 1: Complete Sandbox Environment (Recommended for Testing)

Start with the **Complete Sandbox** for a full testing environment with all services:

```bash
# Navigate to the sandbox directory
cd sandbox

# Start all services (ONIX adapters, mock services, Redis)
docker-compose up -d

# Verify services are running
docker-compose ps

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

For detailed instructions, see: **[Sandbox Guide](./sandbox/README.md)**

**Available Endpoints:**
- **ONIX BAP**: `http://localhost:8001/bap/caller/{action}` and `http://localhost:8001/bap/receiver/{action}`
- **ONIX BPP**: `http://localhost:8002/bpp/caller/{action}` and `http://localhost:8002/bpp/receiver/{action}`
- **Mock Registry**: `http://localhost:3030`
- **Mock CDS**: `http://localhost:8082`
- **Mock BAP**: `http://localhost:9001`
- **Mock BPP**: `http://localhost:9002`

#### Option 2: Standalone ONIX Adapters

For deploying only the ONIX adapters without mock services:

**BAP Adapter:**
```bash
# Navigate to the onix-adaptor directory
cd onix-adaptor

# Start BAP services
docker-compose -f docker-compose-onix-bap-plugin.yml up -d

# Verify services are running
docker-compose -f docker-compose-onix-bap-plugin.yml ps

# View logs
docker-compose -f docker-compose-onix-bap-plugin.yml logs -f onix-bap-plugin

# Stop services
docker-compose -f docker-compose-onix-bap-plugin.yml down
```

**BPP Adapter:**
```bash
# Navigate to the onix-adaptor directory
cd onix-adaptor

# Start BPP services
docker-compose -f docker-compose-onix-bpp-plugin.yml up -d

# Verify services are running
docker-compose -f docker-compose-onix-bpp-plugin.yml ps

# View logs
docker-compose -f docker-compose-onix-bpp-plugin.yml logs -f onix-bpp-plugin

# Stop services
docker-compose -f docker-compose-onix-bpp-plugin.yml down
```

For detailed instructions, see: **[ONIX Adapter Integration Guide](./onix-adaptor/README.md)**

---

## Architecture

### Integration Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    Integration Architecture                  │
└─────────────────────────────────────────────────────────────┘

Phase 1: Discovery (Aggregation via CDS)
┌────────┐         ┌──────────────┐         ┌─────────┐
│  BAP   │ ──────> │ ONIX BAP     │ ──────> │   CDS   │
│        │         │ Caller       │         │         │
└────────┘         └──────────────┘         └────┬────┘
                                                  │
                                    ┌─────────────┴─────────────┐
                                    │   Aggregates from BPPs    │
                                    └─────────────┬─────────────┘
                                                  │
┌────────┐         ┌──────────────┐         ┌────▼────┐
│  BAP   │ <────── │ ONIX BAP     │ <────── │   CDS   │
│        │         │ Receiver     │         │         │
└────────┘         └──────────────┘         └─────────┘

Phase 2+: Direct BPP Communication
┌────────┐         ┌──────────────┐         ┌──────────────┐         ┌────────┐
│  BAP   │ ──────> │ ONIX BAP     │ ──────> │ ONIX BPP     │ ──────> │  BPP   │
│        │         │ Caller       │         │ Receiver     │         │        │
└────────┘         └──────────────┘         └──────────────┘         └────┬───┘
                                                                            │
┌────────┐         ┌──────────────┐         ┌──────────────┐         ┌────▼───┐
│  BAP   │ <────── │ ONIX BAP     │ <────── │ ONIX BPP     │ <────── │  BPP   │
│        │         │ Receiver     │         │ Caller       │         │        │
└────────┘         └──────────────┘         └──────────────┘         └────────┘
```

### Core Components

1. **Transaction Modules**
   - `bapTxnReceiver`: Receives callback responses at BAP
   - `bapTxnCaller`: Sends requests from BAP to BPP/CDS
   - `bppTxnReceiver`: Receives requests at BPP
   - `bppTxnCaller`: Sends responses from BPP to BAP/CDS

2. **Processing Pipeline**
   - Signature validation (`validateSign`)
   - Routing determination (`addRoute`)
   - Schema validation (`validateSchema`)
   - Message signing (`sign`)

3. **Plugins**
   - Cache (Redis-based)
   - Router (YAML-based routing)
   - Signer/SignValidator (Ed25519)
   - SchemaValidator (JSON schema validation)
   - KeyManager (HashiCorp Vault or simple key management)
   - Registry (Subscriber lookup)

---

## Repository Structure

```
ubc-ev-sandbox/
├── api-collection/                   # API documentation and collections
│   ├── field-description/            # Field documentation
│   │   ├── Field_Documentation.csv
│   │   └── FIELD_REFERENCE.md
│   ├── postman-collection/           # Postman API collections
│   │   ├── bap/                      # BAP API collections by action
│   │   │   ├── all-api/
│   │   │   ├── cancel/
│   │   │   ├── confirm/
│   │   │   ├── discover/
│   │   │   ├── init/
│   │   │   ├── rating/
│   │   │   ├── select/
│   │   │   ├── support/
│   │   │   ├── track/
│   │   │   └── update/
│   │   └── bpp/                      # BPP API collections by callback
│   │       ├── all-apis/
│   │       ├── on_cancel/
│   │       ├── on_confirm/
│   │       ├── on_discover/
│   │       ├── on_init/
│   │       ├── on_rating/
│   │       ├── on_select/
│   │       ├── on_status/
│   │       ├── on_support/
│   │       ├── on_track/
│   │       └── on_update/
│   ├── swagger/                      # OpenAPI specifications
│   │   ├── bap.yaml
│   │   └── bpp.yaml
│   └── README.md
├── onix-adaptor/                     # Standalone ONIX adapter integration
│   ├── config/
│   │   ├── onix-bap/                 # BAP adapter configuration
│   │   │   ├── adapter.yaml
│   │   │   ├── bap_caller_routing.yaml
│   │   │   └── bap_receiver_routing.yaml
│   │   └── onix-bpp/                 # BPP adapter configuration
│   │       ├── adapter.yaml
│   │       ├── bpp_caller_routing.yaml
│   │       └── bpp_receiver_routing.yaml
│   ├── docker-compose-onix-bap-plugin.yml
│   ├── docker-compose-onix-bpp-plugin.yml
│   └── README.md
├── sandbox/                          # Complete sandbox environment
│   ├── docker-compose.yml            # Unified compose file for all services
│   ├── onix-bap_config.yml           # Reference config for ONIX BAP
│   ├── onix-bpp_config.yml           # Reference config for ONIX BPP
│   ├── mock-registry_config.yml      # Mock registry configuration
│   ├── mock-cds_config.yml           # Mock CDS configuration
│   ├── mock-bap_config.yml           # Mock BAP configuration
│   ├── mock-bpp_config.yml           # Mock BPP configuration
│   ├── bap-evn.json                  # BAP environment variables
│   ├── bpp-evn.json                  # BPP environment variables
│   └── README.md
├── schemas/                          # JSON schema files for validation
│   └── beckn.one_deg_ev-charging/
│       └── v2.0.0/
│           ├── all.json              # Combined schema
│           ├── discover.json
│           ├── select.json
│           ├── init.json
│           ├── confirm.json
│           ├── update.json
│           ├── cancel.json
│           ├── track.json
│           ├── support.json
│           ├── rating.json
│           ├── on_discover.json
│           ├── on_select.json
│           ├── on_init.json
│           ├── on_confirm.json
│           ├── on_update.json
│           ├── on_cancel.json
│           ├── on_track.json
│           ├── on_status.json
│           ├── on_support.json
│           └── on_rating.json
├── LICENSE
└── README.md                         # This file
```

---

## Configuration

### Configuration Files

Each integration method includes:

1. **Docker Compose Files**: Service definitions with networking and volumes
2. **Adapter Configuration** (`adapter.yaml`): Core adapter settings, modules, and plugins
3. **Routing Configuration**: YAML files defining routing rules for BAP and BPP
4. **Environment Variables**: Container environment configuration

### Key Configuration Areas

- **HTTP Settings**: Port, timeouts, and connection pooling
- **Plugin Configuration**: Cache, router, signer, validators
- **Module Definition**: Transaction receivers and callers
- **Routing Rules**: Phase 1 (CDS) and Phase 2+ (Direct BPP) routing
- **Registry Configuration**: Subscriber lookup and key management

### Configuration Workflow

1. **Service Discovery Flow**:
   - BAP sends discover request → ONIX BAP adapter
   - ONIX BAP routes to → Mock CDS
   - Mock CDS broadcasts to → All registered BPPs
   - BPPs respond → Mock CDS aggregates
   - Mock CDS sends aggregated response → ONIX BAP → Mock BAP

2. **Transaction Flow** (Phase 2+):
   - BAP sends select/init/confirm → ONIX BAP adapter
   - ONIX BAP routes directly to → ONIX BPP (bypasses CDS)
   - ONIX BPP forwards to → Mock BPP backend
   - Mock BPP responds → ONIX BPP
   - ONIX BPP routes callback → ONIX BAP → Mock BAP

---

## Usage Examples

### Complete Sandbox Environment

```bash
# Navigate to the sandbox directory
cd sandbox

# Start all services (ONIX adapters, mock services, Redis)
docker-compose up -d

# Check service status
docker-compose ps

# View logs for all services
docker-compose logs -f

# View logs for specific service
docker-compose logs -f onix-bap-plugin

# Stop all services
docker-compose down
```

### Standalone ONIX Adapters

#### BAP Integration

```bash
# Navigate to the onix-adaptor directory
cd onix-adaptor

# Start BAP services
docker-compose -f docker-compose-onix-bap-plugin.yml up -d

# Check service status
docker-compose -f docker-compose-onix-bap-plugin.yml ps

# View logs
docker-compose -f docker-compose-onix-bap-plugin.yml logs -f onix-bap-plugin

# Stop services
docker-compose -f docker-compose-onix-bap-plugin.yml down
```

**BAP Endpoints:**
- Caller: `http://localhost:8001/bap/caller/{action}`
- Receiver: `http://localhost:8001/bap/receiver/{action}`

#### BPP Integration

```bash
# Navigate to the onix-adaptor directory
cd onix-adaptor

# Start BPP services
docker-compose -f docker-compose-onix-bpp-plugin.yml up -d

# Check service status
docker-compose -f docker-compose-onix-bpp-plugin.yml ps

# View logs
docker-compose -f docker-compose-onix-bpp-plugin.yml logs -f onix-bpp-plugin

# Stop services
docker-compose -f docker-compose-onix-bpp-plugin.yml down
```

**BPP Endpoints:**
- Caller: `http://localhost:8002/bpp/caller/{action}`
- Receiver: `http://localhost:8002/bpp/receiver/{action}`

### Example API Request

This example works with both the **Complete Sandbox** and **Standalone ONIX Adapters** setups:

```bash
# Send a discover request from BAP
# This will be routed to CDS (in sandbox) or your configured CDS endpoint
curl -X POST http://localhost:8001/bap/caller/discover \
  -H "Content-Type: application/json" \
  -d '{
    "context": {
      "domain": "ev_charging_network",
      "version": "1.0.0",
      "action": "discover",
      "bap_id": "example-bap.com",
      "bap_uri": "http://mock-bap:9001",
      "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
      "message_id": "550e8400-e29b-41d4-a716-446655440001",
      "timestamp": "2023-06-15T09:30:00.000Z",
      "ttl": "PT30S"
    },
    "message": {
      "intent": {
        "fulfillment": {
          "start": {
            "location": {
              "gps": "12.9715987,77.5945627"
            }
          },
          "end": {
            "location": {
              "gps": "12.9715987,77.5945627"
            }
          }
        }
      }
    }
  }'
```

**Note**: 
- In the **Complete Sandbox** environment, `bap_uri` can reference `mock-bap:9001` for internal Docker network communication
- For **Standalone ONIX Adapters**, update `bap_uri` to point to your actual BAP backend service endpoint
- The request will be automatically routed to CDS (for discover) or BPP (for other actions) based on the routing configuration

---

## Documentation

### Integration Guides

- **[Sandbox Guide](./sandbox/README.md)**: Complete Docker sandbox with all services (ONIX adapters, mock services, Redis)
- **[ONIX Adapter Integration Guide](./onix-adaptor/README.md)**: Standalone Docker-based ONIX adapter deployment

### API Documentation

- **[API Collection Guide](./api-collection/README.md)**: Postman collections, Swagger specifications, and field documentation
- **[Schema Documentation](./schemas/README.md)**: JSON schema definitions for validation

### Related Documentation

- **[ONIX-Adapter Repository](https://github.com/Beckn-One/beckn-onix)**: Official onix-adapter source code and documentation
- **[ONIX Configuration Guide](https://github.com/Beckn-One/beckn-onix/blob/main/CONFIG.md)**: Detailed configuration parameters
- **[ONIX Setup Guide](https://github.com/Beckn-One/beckn-onix/blob/main/SETUP.md)**: Installation and setup instructions
- **[Beckn Protocol Specifications](https://github.com/beckn/protocol-specifications)**: Protocol documentation

---

## Contributing

We welcome contributions! When contributing examples or improvements:

1. **Follow the directory structure**: Maintain consistency with existing examples
2. **Include documentation**: Each integration method should have a comprehensive README
3. **Provide working examples**: Include Docker Compose files, configuration files, and usage examples
4. **Add troubleshooting guides**: Help users resolve common issues
5. **Test your changes**: Ensure all configurations work before submitting

### Contribution Guidelines

- Clear documentation with inline comments
- Working configuration files
- Environment variable examples
- Troubleshooting sections
- Consistent code formatting

---

## Support

For issues, questions, or contributions:

1. **Check Documentation**: Review the relevant integration guide and troubleshooting sections
2. **Review Examples**: Examine existing configuration files and examples
3. **Open an Issue**: Report bugs or request features via GitHub Issues
4. **Check ONIX Repository**: Refer to the [main onix-adapter repository](https://github.com/Beckn-One/beckn-onix) for core functionality

### Resources

- **ONIX Issues**: [Beckn-One/beckn-onix Issues](https://github.com/Beckn-One/beckn-onix/issues)
- **ONIX Discussions**: [GitHub Discussions](https://github.com/Beckn-One/beckn-onix/discussions)

---

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.

The onix-adapter itself is licensed under the MIT License. See the [onix-adapter LICENSE](https://github.com/Beckn-One/beckn-onix/blob/main/LICENSE) for more information.

---

## Acknowledgments

- **[Beckn Foundation](https://beckn.org)**: For the Beckn Protocol specifications
- **[Beckn-One](https://github.com/Beckn-One)**: For the onix-adapter project
- All contributors to the onix-adapter and this integration guide

---

Built with ❤️ for the open Value Network ecosystem
