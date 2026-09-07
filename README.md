# package-master-data-flow-config-accelerator

**Version:** 1.0.1
**Platform:** Fuuz ≥ 2023.11.2
**Spec Version:** 2.0.0

---

## Overview

The Master Data Flow Configuration package provides the orchestration layer for scheduling and managing bulk master data imports from external ERP and datasource systems into Fuuz. It introduces a `MasterDataFlowConfiguration` model — a registry of all active import flows — and wraps it with a scheduler, dispatcher flow, and management screens. Any import flow that should run on a recurring schedule (e.g., syncing products, workcenters, customers from Plex, NetSuite, or another ERP) is registered here and managed centrally.

This is a low-level infrastructure package, typically installed once per environment and consumed by connector-specific packages like `package-plex-odbc-master-config-accelerator` or `package-mes-netsuite-accelerator`.

---

## Package Contents

```
master-data-flow-config/
├── manifest.json
├── definition.json
├── package-data.json
├── data/                        2 seed data files
├── dataFlows/                   4 flows
├── dataModels/                  2 models
├── savedTransforms/             2 reusable transforms
└── screens/                     6 screens
```

---

## Data Models

### MasterDataFlowConfiguration

The core scheduling registry. Each record represents one import flow to be run on a schedule.

| Field | Type | Description |
|-------|------|-------------|
| `active` | Boolean! | Whether this import is enabled; inactive records are skipped by the scheduler |
| `label` | String | Auto-generated display label set by update trigger as `{Connector Name} - {Connection Name}` |
| `connectionId` | String | The Fuuz `Connection` record ID to use for this import |
| `topic` | String | Pub/sub topic string formatted as `{SystemName}.{Source}.import` — used for subscriber-based dispatch |
| `dataFlowId` | String | The target integration flow to execute for this import |
| `schedule` | JSON | Cron or interval schedule definition |
| `lastRunAt` | DateTime | Timestamp of the last successful execution |
| `lastRunStatus` | String | Status code from the last run (success, error, etc.) |

The update trigger automatically resolves and caches the connector name and connection name into `label` whenever `connectionId` changes. `dataChangeCapture` is enabled with 120-day retention (exposed) for audit streaming.

### Supporting Configuration Schema Model

A companion model storing JSON schema definitions for import payload validation and connector-specific configuration metadata.

---

## Data Flows (4)

| Flow | Type | Description |
|------|------|-------------|
| Master Data Import Scheduler | System | Periodic trigger; queries all active `MasterDataFlowConfiguration` records and dispatches each to its configured import flow |
| Master Data Import Dispatcher | System | Receives a single configuration record and executes the target `dataFlowId` with the configured connection context |
| Master Data Configuration Sync | Integration | Routes inbound import requests by pub/sub `topic` pattern to the correct handler |
| Master Data Status Update | System | Updates `lastRunAt` and `lastRunStatus` on the configuration record after each import run completes |

---

## Screens (6)

- **Configuration List** — Table view of all import configurations with active status and last-run timestamps
- **Configuration Form** — Create/edit form for individual records; includes connection picker, flow selector, and schedule builder
- **Import Run History** — Timeline view of recent execution logs per configuration
- **Manual Trigger** — On-demand execution screen to run a specific import outside the schedule
- **Bulk Enable/Disable** — Administrative screen to toggle active status on multiple configurations at once
- **Import Status Dashboard** — Summary view of all imports with health indicators (last run status, run age, error counts)

---

## Installation

1. Import via Fuuz Package Manager
2. Ensure at least one `Connection` record exists for the target ERP/datasource
3. Install connector-specific packages (e.g., `package-plex-odbc-master-config-accelerator`) — their flows will self-register into `MasterDataFlowConfiguration` records or can be added manually
4. Configure schedules per import via the Configuration Form screen
5. Enable desired imports by setting `active = true`

---

## Dependencies

- **Fuuz Platform** ≥ 2023.11.2
- **Scheduler module** — required for the periodic import dispatcher
- **Pub/Sub module** — required for topic-based dispatch routing
- One or more ERP connector packages (e.g., `package-plex-odbc-master-config-accelerator`)

---

*Built on the [Fuuz Industrial Operations Platform](https://fuuz.com)*

## Service levels

No service level agreement applies to anything published here. It becomes a supported
deliverable only once it has been implemented by a Fuuz services professional or an
approved Fuuz partner.
