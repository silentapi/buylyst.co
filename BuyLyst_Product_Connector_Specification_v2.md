# BuyLyst.co — Product Catalog & SKU Connector System Specification

## Version 2.0  
## Document Type: Product Catalog, Sync, Connector & SKU Resolution Specification  
## Purpose: Define the architecture, workflows, operational requirements, and connector abstraction model for BuyLyst.co catalog synchronization and inventory SKU resolution systems.

---

# 1. Executive Summary

BuyLyst.co requires a scalable and extensible catalog synchronization system capable of:

- ingesting large trading card catalogs,
- syncing market pricing data,
- powering public and operational product search,
- supporting buylist workflows,
- resolving inventory export SKUs,
- enabling inventory exports,
- supporting future connector providers,
- and maintaining a fully local searchable catalog.

The platform must not hard-code any specific third-party provider directly into application workflows.

Instead, BuyLyst.co must treat:

- product catalog synchronization,
- pricing synchronization,
- and SKU resolution

as configurable connector-based systems.

For the initial implementation:

| Connector Type | Initial Supported Connector |
|---|---|
| Catalog & pricing sync | TCGCSV |
| SKU resolution | TCGPlayer Product Details API |

However, the architecture must support future connector expansion without requiring rewrites of:

- customer workflows,
- quick buy systems,
- reporting systems,
- inventory systems,
- or export systems.

---

# 2. Core Goals

The connector system must:

- maintain a complete local searchable catalog;
- support extremely fast user-facing search;
- avoid unnecessary third-party API calls;
- support scheduled and manual synchronization;
- support partial category/group/product synchronization;
- support inventory export workflows;
- preserve synchronization history and metadata;
- support future connector providers;
- separate product synchronization from SKU resolution;
- support auditing and diagnostics;
- support operational recovery and troubleshooting.

---

# 3. Connector Architecture

---

# 3.1 Connector Categories

BuyLyst.co separates external integrations into two independent connector systems.

---

## Catalog Sync Connectors

Responsible for:

- categories/games,
- groups/sets,
- products,
- market pricing,
- product metadata,
- source URLs,
- source images,
- synchronization timestamps,
- source update timestamps.

---

## SKU Resolution Connectors

Responsible for:

- export SKU resolution,
- condition-specific SKU mapping,
- variant/language matching,
- inventory export compatibility,
- unresolved SKU reporting.

---

# 3.2 Connector Independence

The system must not assume:

- the same provider powers both systems;
- all providers use identical identifiers;
- all providers use identical variant naming;
- all providers use identical category structures.

The inventory export system should not know how TCGCSV or TCGPlayer work internally.

Instead:

```text
Export System
    -> SKU Resolution Connector
        -> Normalized SKU Result
```

And:

```text
Catalog Sync System
    -> Catalog Sync Connector
        -> Normalized Local Catalog Data
```

---

# 3.3 Initial Supported Connectors

Initial supported connectors:

```json
{
  "catalogSyncConnector": "tcgcsv",
  "skuResolutionConnector": "tcgplayer_product_details"
}
```

Only these connectors should appear in admin settings initially.

The abstraction layer must still exist even though only one connector per category is initially supported.

---

# 3.4 Future Connector Expansion

Future connector examples may include:

- direct TCGPlayer APIs,
- distributor feeds,
- Shopify catalog imports,
- POS inventory feeds,
- custom CSV imports,
- proprietary store inventory systems,
- marketplace APIs.

The architecture must support adding future connectors without rewriting core operational systems.

---

# 4. Admin Connector Configuration

Connector configuration must exist in admin settings from day one.

Connector selection is global.

The initial implementation does not support selecting different connectors per game/category.

Example:

```json
{
  "catalogSyncConnector": "tcgcsv",
  "skuResolutionConnector": "tcgplayer_product_details"
}
```

Connector-specific settings should be namespaced.

Example:

```json
{
  "connectors": {
    "tcgcsv": {
      "baseUrl": "https://tcgcsv.com/tcgplayer",
      "lastUpdatedUrl": "https://tcgcsv.com/tcgplayer/last-updated.txt",
      "userAgent": "BuyLystCatalogSync/1.0",
      "requestDelayMs": 100
    },
    "tcgplayer_product_details": {
      "urlTemplate": "https://mp-search-api.tcgplayer.com/v2/product/{productId}/details"
    }
  }
}
```

---

# 5. Local Catalog Architecture

The local catalog must be fully stored inside BuyLyst.co.

User-facing workflows must not depend on live third-party catalog requests.

The local catalog powers:

- public buylist search,
- admin buylist management,
- quick buy workflows,
- intake workflows,
- reporting,
- analytics,
- inventory reconciliation,
- inventory exports.

---

# 5.1 Catalog Hierarchy

Catalog data is stored as separate concepts.

```text
Catalog
    -> Groups
        -> Product Variants
```

---

## Catalogs

Represent games/product families.

Examples:

- Magic: The Gathering
- Yu-Gi-Oh!
- Pokémon
- One Piece
- Lorcana

---

## Groups

Represent sets/expansions.

Examples:

- Origins
- Silver Tempest
- Tactical Masters

Groups reference catalogs.

---

## Product Variants

Represent sellable product variants.

Examples:

- Normal
- Foil
- Reverse Holofoil
- Extended Art

Products reference groups and catalogs.

---

# 6. Product Variant Identity Design

BuyLyst.co product variants must not rely on a single concatenated internal ID string as the authoritative identity.

Instead, product variants are identified using indexed fields.

---

# 6.1 Product Variant Identity

Recommended identity fields:

```json
{
  "catalogConnector": "tcgcsv",
  "sourceProductId": 652898,
  "sourceVariantName": "Foil"
}
```

Recommended unique index:

```text
catalogConnector + sourceProductId + normalizedSourceVariantName
```

This is the authoritative product variant identity.

---

# 6.2 Optional Convenience Key

The system may optionally generate a convenience key.

Example:

```text
tcgcsv:652898:foil
```

However:

- this key is NOT the authoritative identity;
- the indexed columns are.

---

# 6.3 Why This Design Was Chosen

This design provides:

- future multi-connector flexibility,
- easier migrations,
- cleaner indexing,
- safer normalization,
- easier reporting,
- better connector abstraction,
- reduced parsing complexity,
- safer variant reconciliation.

---

# 7. TCGCSV Catalog Connector

---

# 7.1 Purpose

TCGCSV is the initial catalog synchronization connector.

TCGCSV provides:

- categories,
- groups,
- products,
- pricing data.

TCGCSV is used to build BuyLyst.co’s local searchable catalog.

Official documentation:

```text
https://tcgcsv.com/docs
```

---

# 7.2 TCGCSV Operational Rules

Important operational behavior:

- TCGCSV updates approximately once daily.
- The system should check `last-updated.txt`.
- Excessive polling should be avoided.
- Requests should use a custom User-Agent.
- Requests should occur from backend services only.
- Sync jobs should support throttling/delay behavior.

Recommended User-Agent:

```text
BuyLystCatalogSync/1.0
```

---

# 7.3 TCGCSV Data Hierarchy

TCGCSV follows:

```text
Category -> Group -> Product -> Price
```

---

# 7.4 Categories

Categories represent games/product families.

Example:

```json
{
  "categoryId": 3,
  "name": "Pokemon"
}
```

Recommended local collection:

```text
catalogs
```

---

# 7.5 Groups

Groups represent sets/expansions.

Example:

```json
{
  "groupId": 3170,
  "name": "Silver Tempest",
  "categoryId": 3
}
```

Recommended local collection:

```text
groups
```

---

# 7.6 Products

Products represent product records independent of pricing variants.

Products contain:

- product names,
- images,
- source URLs,
- rarity,
- numbers,
- extended metadata,
- source timestamps.

Recommended local collection:

```text
products
```

---

# 7.7 Price Variants

Price records represent purchasable variants.

Examples:

- Normal
- Holofoil
- Reverse Holofoil
- Foil

Price records must be joined to products using:

```text
sourceProductId
```

Each joined variant becomes a local product variant record.

---

# 8. Local Product Variant Shape

Recommended normalized product variant shape:

```json
{
  "catalogConnector": "tcgcsv",

  "sourceProductId": 652898,
  "sourceVariantName": "Foil",

  "catalogId": 89,
  "groupId": 24344,

  "name": "Thousand-Tailed Watcher",
  "cleanName": "Thousand-Tailed Watcher",

  "rarity": "Rare",
  "number": "116/298",

  "marketPrice": 24.54,
  "lowPrice": 23,
  "midPrice": 29.505,
  "directLowPrice": 24.99,

  "imageUrl": "SOURCE_URL",
  "sourceUrl": "SOURCE_URL",

  "extendedData": {},

  "productModifiedOn": "ISO_TIMESTAMP",
  "lastSyncedAt": "ISO_TIMESTAMP"
}
```

---

# 9. Product Images

For the initial implementation:

- product images should use source URLs;
- images should not be cached locally initially.

The architecture should allow optional future image caching/CDN support later.

---

# 10. Synchronization System

Synchronization is an operational admin-managed system.

It is not simply a hidden cron job.

---

# 10.1 Admin Sync Management

Admins/managers must be able to:

- trigger syncs manually;
- schedule automated syncs;
- sync all catalogs;
- sync one catalog;
- sync one group;
- sync one product;
- view sync history;
- view sync summaries;
- view metadata failures;
- view sync diagnostics;
- retry failed syncs;
- cancel queued/running syncs where possible.

---

# 10.2 Scheduled Syncs

The system must support scheduled sync jobs.

Examples:

- nightly full sync,
- category-specific sync,
- release-day sync schedules.

Scheduling configuration should exist in admin settings.

---

# 10.3 Manual Syncs

Manual syncs must support:

- full sync;
- catalog sync;
- group sync;
- product sync.

Manual syncs must:

- be audit logged;
- expose progress/state;
- preserve summaries/errors.

---

# 10.4 Sync Job Tracking

Recommended sync job states:

| Status | Meaning |
|---|---|
| queued | Waiting to run |
| running | Currently syncing |
| completed | Finished successfully |
| partial | Completed with recoverable errors |
| failed | Sync failed |
| cancelled | Sync manually cancelled |

---

# 10.5 Sync Metadata

Sync jobs should preserve:

- start time,
- completion time,
- connector used,
- records processed,
- records updated,
- records skipped,
- metadata failures,
- pricing failures,
- parsing failures,
- upstream request failures.

---

# 11. SKU Resolution System

---

# 11.1 Purpose

SKU resolution exists to support inventory export workflows.

SKUs are not required for:

- public search,
- customer carts,
- intake workflows,
- quick buy workflows,
- normal buylist configuration.

---

# 11.2 Initial SKU Connector

Initial connector:

```text
TCGPlayer Product Details API
```

Example endpoint:

```text
https://mp-search-api.tcgplayer.com/v2/product/{productId}/details
```

---

# 11.3 SKU Lookup Timing

SKU resolution should happen:

- during inventory export;
- during manual admin/manager pre-resolution;
- when manually refreshing SKU mappings.

The system should not fetch SKUs during normal catalog synchronization.

---

# 11.4 SKU Mapping Inputs

SKU matching may require:

- source product ID,
- condition,
- variant,
- language.

---

# 11.5 Supported Conditions

Canonical conditions:

```text
Near Mint
Lightly Played
Moderately Played
Heavily Played
Damaged
```

---

# 11.6 Existing SKU Mappings

SKU mappings should generally be trusted once resolved.

The system should:

- use existing mappings if available;
- only fetch missing mappings;
- avoid unnecessary repeated lookups.

SKUs rarely change.

---

# 11.7 Manual SKU Resolution

Managers/admins must be able to:

- manually trigger SKU resolution;
- manually enter SKU mappings;
- override unresolved mappings;
- save verified mappings.

Manual overrides must be audit logged.

---

# 12. Custom Items

Quick Buy supports custom/manual items not present in the catalog.

Examples:

- bulk lots,
- accessories,
- miscellaneous inventory,
- damaged lots,
- uncatalogued products.

Custom items may not have valid SKUs.

---

# 12.1 Custom Item Export Rules

Custom items require inventory users to:

- manually provide a SKU;
- or explicitly mark the item as skipped.

The system must not silently export unresolved custom items.

---

# 13. Inventory Export Workflow

Inventory exports use:

- final inventory reconciliation data;
- final inventory conditions;
- final inventory quantities;
- resolved SKU mappings.

Exports must not use:

- original intake conditions;
- original intake quantities;
- unresolved SKU guesses.

---

# 13.1 Unresolved SKU Handling

If no valid SKU exists:

- the export must flag the item;
- the item must require manual review;
- the system must not guess.

Managers/inventory users must be able to:

- retry lookup;
- manually enter SKU;
- skip export item.

---

# 14. Reporting & Diagnostics

The connector system must support operational diagnostics.

Admins/managers should be able to view:

- sync durations,
- sync failures,
- unresolved products,
- unresolved SKUs,
- duplicate variants,
- parsing failures,
- stale products,
- stale mappings,
- connector health,
- import statistics.

---

# 15. Audit Logging Requirements

The following actions must be audit logged:

- sync started;
- sync completed;
- sync failed;
- manual sync requested;
- sync cancelled;
- SKU lookup performed;
- SKU lookup failed;
- manual SKU mapping created;
- manual SKU mapping edited;
- manual SKU mapping deleted;
- export blocked;
- export completed with skipped items;
- unresolved SKU approvals.

---

# 16. Recommended Configuration Structure

Example configuration:

```json
{
  "catalogSyncConnector": "tcgcsv",
  "skuResolutionConnector": "tcgplayer_product_details",

  "catalogSyncEnabled": true,

  "connectors": {
    "tcgcsv": {
      "baseUrl": "https://tcgcsv.com/tcgplayer",
      "lastUpdatedUrl": "https://tcgcsv.com/tcgplayer/last-updated.txt",
      "userAgent": "BuyLystCatalogSync/1.0",
      "requestDelayMs": 100
    },

    "tcgplayer_product_details": {
      "urlTemplate": "https://mp-search-api.tcgplayer.com/v2/product/{productId}/details"
    }
  }
}
```

---

# 17. Agent Implementation Guidance

Agents working on this system should follow these principles:

1. Do not hard-code connector-specific logic throughout the application.
2. Normalize connector output into local catalog structures.
3. Treat catalog sync and SKU resolution as separate systems.
4. Use indexed identity fields instead of concatenated IDs.
5. Do not make user-facing workflows depend on live catalog APIs.
6. Do not fetch SKUs during customer workflows.
7. Resolve SKUs during export or explicit admin action.
8. Preserve synchronization metadata and history.
9. Preserve operational diagnostics and error visibility.
10. Prefer safe failure over incorrect exports.
11. Preserve auditability for all connector operations.
12. Add tests for:
   - parsing,
   - synchronization,
   - joins,
   - SKU matching,
   - unresolved SKU handling,
   - connector selection behavior.

---

# 18. Summary

BuyLyst.co should use a connector-based architecture for:

- catalog synchronization,
- pricing synchronization,
- and SKU resolution.

The platform should maintain a fully local searchable catalog while using external providers only for synchronization and export resolution workflows.

For the initial implementation:

- TCGCSV is the initial catalog connector;
- TCGPlayer Product Details API is the initial SKU connector.

The architecture must support future connector expansion without requiring rewrites of operational systems.

The synchronization system must function as a first-class admin operational tool with:

- scheduling,
- manual execution,
- diagnostics,
- summaries,
- auditing,
- and error visibility.

This design provides BuyLyst.co with a scalable and extensible foundation suitable for real-world trading card retail operations.
