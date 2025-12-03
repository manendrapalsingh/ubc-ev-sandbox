# ONIX Adapter Deployment Guide

Standalone ONIX adapter deployments now live under `onix-adaptor/`.  
Use this guide when you want to run the ONIX BAP/BPP plugins without the mock services that ship with the full sandbox (located in `sandbox/`).

## Directory Layout

```
onix-adaptor/
├── api/              # REST-only adapters (HTTP ↔︎ HTTP)
│   ├── config/
│   │   ├── onix-bap/     # BAP adapter configs & routing
│   │   │   ├── adapter.yaml
│   │   │   ├── bap_caller_routing.yaml
│   │   │   └── bap_receiver_routing.yaml
│   │   └── onix-bpp/     # BPP adapter configs & routing
│   │       ├── adapter.yaml
│   │       ├── bpp_caller_routing.yaml
│   │       └── bpp_receiver_routing.yaml
│   ├── docker-compose-onix-bap-plugin.yml
│   ├── docker-compose-onix-bpp-plugin.yml
│   └── README.md
└── kafka/            # Kafka-enabled adapters (Kafka ↔︎ HTTP)
    ├── config/
    │   ├── onix-bap/     # BAP adapter configs & routing
    │   │   ├── adapter.yaml
    │   │   ├── bapTxnCaller-routing.yaml
    │   │   └── bapTxnReciever-routing.yaml
    │   └── onix-bpp/     # BPP adapter configs & routing
    │       ├── adapter.yaml
    │       ├── bppTxnCaller-routing.yaml
    │       └── bppTxnReciever-routing.yaml
    ├── docker-compose-onix-bap-kafka-plugin.yml
    ├── docker-compose-onix-bpp-kafka-plugin.yml
    └── README.md
```

Each compose file expects configuration from the sibling `config/` tree (already wired via relative mounts), so the only prerequisite is running the commands from the appropriate subdirectory (`api/` or `kafka/`).

## Quick Start (Kafka Plugins)

Run these when integrating with the Kafka sandbox layout (`sandbox/kafka`) or connecting to your own Kafka cluster.

### BAP Plugin
```bash
cd onix-adaptor/kafka
docker compose -f docker-compose-onix-bap-kafka-plugin.yml up -d
```

### BPP Plugin
```bash
cd onix-adaptor/kafka
docker compose -f docker-compose-onix-bpp-kafka-plugin.yml up -d
```

Both stacks mount config and schema files from `onix-adaptor/kafka/config` and `../../schemas/` (via relative paths), matching the structure used by `sandbox/kafka/docker-compose.yml`.

For detailed Kafka integration documentation, see: **[Kafka Integration Guide](./kafka/README.md)**

## Quick Start (HTTP/API Plugins)

If you need only the REST adapters:

```bash
cd onix-adaptor/api
docker compose -f docker-compose-onix-bap-plugin.yml up -d
docker compose -f docker-compose-onix-bpp-plugin.yml up -d
```

The API compose files rely on `onix-adaptor/api/config/...` for adapter/routing config.

For detailed API integration documentation, see: **[API Integration Guide](./api/README.md)**

## Configuration Notes

- **API adapters**: Update `api/config/onix-bap` or `api/config/onix-bpp` YAML files to point at your BAP/BPP endpoints.
- **Kafka adapters**: Update `kafka/config/onix-bap` or `kafka/config/onix-bpp` YAML files to point at your BAP/BPP endpoints and Kafka brokers.
- Schemas resolve from `../../schemas` relative to each subdirectory; keep that path intact or adjust the compose volumes.
- When using the sandbox Kafka mocks, ensure the registry entries match your chosen `subscriber_id`/`subscriber_uri`.

## Documentation

For detailed integration guides:
- **[API Integration Guide](./api/README.md)**: REST/HTTP adapter setup and configuration
- **[Kafka Integration Guide](./kafka/README.md)**: Kafka event streaming adapter setup and configuration
- **[Upstream ONIX Documentation](https://github.com/Beckn-One/beckn-onix)**: Core onix-adapter repository and documentation