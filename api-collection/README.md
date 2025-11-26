# API Collection Directory

This directory contains the artifacts you need to inspect the EV charging Postman suites and their documentation.

## Contents

- `postman-collection/`: Postman collections organized by protocol and action:
  - `bap/`: Buyer App Protocol (BAP) collections organized by action (cancel, confirm, discover, init, rating, select, support, track, update) plus an aggregated "all-api" bundle.
  - `bpp/`: Buyer Platform Protocol (BPP) collections organized by callback (on_cancel, on_confirm, on_discover, on_init, on_rating, on_select, on_status, on_support, on_track, on_update) plus an aggregated "all-apis" bundle.
- `swagger/`: OpenAPI specifications (`bap.yaml`, `bpp.yaml`) that describe caller and callback endpoints for the EV charging domain.
- `field-description/`: Supporting documentation for JSON schemas:
  - `Field_Documentation.csv`: CSV table with field descriptions and metadata.
  - `FIELD_REFERENCE.md`: Reference documentation for field definitions and usage.
