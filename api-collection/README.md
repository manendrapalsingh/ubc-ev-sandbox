# API Collection Directory

This directory contains the artifacts you need to inspect the EV charging Postman suites and their documentation.

## Contents

- `postman-collection/`: Postman collections organized by protocol and action:
  - `bap/`: Buyer App Protocol (BAP) collections organized by action (cancel, confirm, discover, init, rating, select, support, track, update) plus an aggregated "all-api" bundle.
  - `bpp/`: Buyer Platform Protocol (BPP) collections organized by callback (on_cancel, on_confirm, on_discover, on_init, on_rating, on_select, on_status, on_support, on_track, on_update, catalog_publish, on_catalog_publish) plus an aggregated "all-apis" bundle.
- `swagger/`: OpenAPI specifications (`bap.yaml`, `bpp.yaml`) that describe caller and callback endpoints for the EV charging domain.
  - **Note**: The Swagger files are aligned with the Postman collections. The `location` object has been removed from the context schema as it is not part of the standard Beckn protocol context.
- `field-description/`: Supporting documentation for JSON schemas:
  - `Field_Documentation.csv`: CSV table with field descriptions and metadata.
  - `FIELD_REFERENCE.md`: Reference documentation for field definitions and usage.

## Important Notes

### Context Schema
- The `location` object (country and city codes) has been **removed** from the context schema in both Swagger files and Postman collections.
- The context now includes: `version`, `action`, `domain`, `bap_id`, `bap_uri`, `bpp_id`, `bpp_uri`, `transaction_id`, `message_id`, `timestamp`, `ttl`, and `schema_context`.
- Location information for charging stations is available in the message body (e.g., `beckn:availableAt` in catalog items, `deliveryAttributes.location` in fulfillment).

### API Alignment
- All Swagger specifications are aligned with their corresponding Postman collections.
- BAP collection includes: discover, select, init, confirm, update, track, cancel, rating, support.
- BPP collection includes: on_discover, on_select, on_init, on_confirm, on_update, on_track, on_cancel, on_rating, on_support, on_status, catalog_publish, on_catalog_publish.
