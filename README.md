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

This repository provides a complete sandbox environment for integrating the **[onix-adapter](https://github.com/Beckn-One/beckn-onix)** (also known as Beckn-ONIX) with **BAP (Application Platform)** and **BPP (Provider Platform)** applications for EV Charging networks.

### What is Onix-Adapter?

The **onix-adapter** is a production-ready, plugin-based middleware adapter for the Beckn Protocol. It acts as a protocol adapter between Application Platforms (BAPs) and Provider Platforms (BPPs), ensuring secure, validated, and compliant message exchange across various commerce networks.

### Key Concepts

- **BAP (Application Platform)**: Buyer-side applications that help users search for and purchase products/services (e.g., consumer apps, aggregators)
- **BPP (Provider Platform)**: Seller-side platforms that provide products/services (e.g., merchant platforms, service providers)
- **Onix-Adapter**: Middleware that handles protocol compliance, message signing, validation, and routing between BAPs and BPPs
- **CDS (Catalog Discovery Service)**: Aggregates discover requests from BAPs and broadcasts to registered BPPs
- **Registry**: Maintains a registry of all network participants (BAPs, BPPs, CDS)

---

## Features

### Complete Sandbox Environment

- **Full Testing Environment**: Pre-configured complete sandbox with ONIX adapters, mock services, and infrastructure
- **Standalone Adapters**: Deploy only ONIX adapters for integration with your own services
- **Mock Services**: Simulated BAP, BPP, CDS, and Registry services for testing

### Architecture

- **REST API Communication**: Asynchronous HTTP/REST messaging for real-time interactions (available in `sandbox/api` and `onix-adaptor/api`)
- **Kafka Event Streaming**: High-throughput, distributed event processing (available in `sandbox/kafka` and `onix-adaptor/kafka`)
- **Redis Caching**: Performance optimization and async state management
- **Docker Compose**: Easy local development and testing setup

### Enterprise-Ready

- **Ed25519 Digital Signatures**: Cryptographically secure message signing and validation
- **JSON Schema Validation**: Ensures protocol compliance using schemas from `schemas/` directory
- **Configurable Routing**: YAML-based routing rules
- **Structured Logging**: JSON-formatted logs with transaction tracking

### Production Features

- **Health Checks**: Liveness and readiness probes for all services
- **Environment-Specific Configs**: Separate configurations for different services
- **API Collections**: Postman collections and Swagger specifications for all APIs

---

## Quick Start

### Prerequisites

- Docker Engine 20.10 or higher
- Docker Compose 2.0 or higher

### Recommended Starting Point

#### Option 1: Complete Sandbox Environment (Recommended for Testing)

Choose the stack that matches the transport layer you want to exercise:

- **Kafka-first flow (full mock ecosystem)** – lives in `sandbox/kafka`
- **HTTP-only mock services** – lives in `sandbox/api`

**Kafka Sandbox**

```bash
cd sandbox/kafka
docker compose up -d          # start all services
docker compose ps             # verify status
docker compose logs -f        # tail logs
# ...
docker compose down           # stop everything
```

**HTTP Sandbox**

```bash
cd sandbox/api
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
```

Each environment mounts configs relative to its folder, so run the commands from within `sandbox/kafka` or `sandbox/api`.  
**Available Endpoints (Kafka stack defaults):**
- **ONIX BAP**: `http://localhost:8001/bap/{caller|receiver}/{action}`
- **ONIX BPP**: `http://localhost:8002/bpp/{caller|receiver}/{action}`
- **Mock Registry**: `http://localhost:3030`
- **Mock CDS**: `http://localhost:8082`
- **Mock BAP**: `http://localhost:9001`
- **Mock BPP**: `http://localhost:9002`

#### Option 2: Standalone ONIX Adapters

Need only the ONIX adapters? Use the compose files under `onix-adaptor/` directly.

**Kafka Plugins (connect to real Kafka clusters)**
```bash
cd onix-adaptor/kafka
docker compose -f docker-compose-onix-bap-kafka-plugin.yml up -d
docker compose -f docker-compose-onix-bpp-kafka-plugin.yml up -d
# ...
docker compose -f docker-compose-onix-bap-kafka-plugin.yml down
docker compose -f docker-compose-onix-bpp-kafka-plugin.yml down
```

**HTTP/API Plugins (REST ↔︎ REST)**
```bash
cd onix-adaptor/api
docker compose -f docker-compose-onix-bap-plugin.yml up -d
docker compose -f docker-compose-onix-bpp-plugin.yml up -d
# ...
docker compose -f docker-compose-onix-bap-plugin.yml down
docker compose -f docker-compose-onix-bpp-plugin.yml down
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



## Documentation

### Integration Guides

- **[HTTP/REST Sandbox Guide](./sandbox/api/README.md)**: Complete Docker sandbox with HTTP/REST communication (ONIX adapters, mock services, Redis)
- **[Kafka Sandbox Guide](./sandbox/kafka/README.md)**: Complete Docker sandbox with Kafka event streaming (ONIX adapters, mock services, Redis, Kafka)
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
- **UBC EV Sandbox Repo**: [bhim/ubc-ev-sandbox (Discussions & updates)](https://github.com/bhim/ubc-ev-sandbox)

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
