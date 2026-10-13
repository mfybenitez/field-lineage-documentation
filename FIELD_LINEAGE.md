# analytics.transactions_data_source Lineage (96 Fields)

## Overview

This is a concise, grouped lineage document for `analytics.transactions_data_source` (96 columns), organized by **shared data flows** instead of field-by-field repetition.

Standard layer definitions used throughout:

- **Raw** = `raw_import_cache`
- **Staging** = `warehouse_loader`
- **Public** = `public`
- **Analytics** = `analytics.transactions_data_source`

High-level pipeline:

```text
MSSQL / source systems
→ raw_import_cache
→ warehouse_loader
→ public
→ analytics.transactions_data_source
```

## Controlled Source Values

Use only these values in **Source**:
- `Backend`
- `ETL`
- `Third-party`
- `Derived`

## Table Columns (all sections)

Every table below uses:

1. Redshift Table  
2. Table Grain  
3. Column Name  
4. Data Type  
5. Description  
6. Example Value  
7. Source  
8. GitHub Link  
9. Logic Explanation

> **Note:** GitHub links are placeholders for now, per your request.

---

## Flow Group 1: Payment + Transaction Lifecycle Fields

### Flow

```text
MSSQL.transaction + transactionCommitted + transactionRolledBack + payment events
→ raw_import_cache.transaction / transactionCommitted / transactionRolledBack / payment*
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction (+ return adjustments)
→ analytics.transactions_data_source
```

### Logic Summary

- Core transaction identity + timestamps are assembled from source lifecycle tables.
- `transaction_end_datetime` is selected with CASE logic from start/commit/rollback signals.
- Financial measures are loaded in public and passed/adjusted in analytics.

### Raw Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Transaction_GUID | varchar | Source transaction GUID | ABC123 | Backend | LINK_RAW_PAY_001 | Primary transaction identifier |
| raw_import_cache.transaction | one row per transaction_guid | transaction_id | bigint | Source transaction ID | 354 | Backend | LINK_RAW_PAY_002 | Numeric transaction identifier |
| raw_import_cache.transaction | one row per transaction_guid | Create_dm | timestamp | Source start timestamp (UTC) | 2026-09-29 14:58:11 | Backend | LINK_RAW_PAY_003 | Transaction creation timestamp |
| raw_import_cache.transactionCommitted | one row per committed transaction | create_dm | timestamp | Commit timestamp (UTC) | 2026-09-29 15:00:05 | Backend | LINK_RAW_PAY_004 | Present when transaction commits |
| raw_import_cache.transactionRolledBack | one row per rolled back transaction | create_dm | timestamp | Rollback timestamp (UTC) | 2026-09-29 15:03:40 | Backend | LINK_RAW_PAY_005 | Present when transaction reverses |

### Staging Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_guid | varchar | Conformed transaction GUID | ABC123 | ETL | LINK_STG_PAY_001 | Pass-through from raw |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_id | bigint | Conformed transaction ID | 354 | ETL | LINK_STG_PAY_002 | Pass-through from raw |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_start_datetime | timestamp | Start timestamp (UTC) | 2026-09-29 14:58:11 | ETL | LINK_STG_PAY_003 | From raw transaction.Create_dm |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_commit_datetime | timestamp | Commit timestamp (UTC) | 2026-09-29 15:00:05 | ETL | LINK_STG_PAY_004 | From transactionCommitted.create_dm |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_rollback_datetime | timestamp | Rollback timestamp (UTC) | 2026-09-29 15:03:40 | ETL | LINK_STG_PAY_005 | From transactionRolledBack.create_dm |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_end_datetime | timestamp | Final transaction event (UTC) | 2026-09-29 15:03:40 | Derived | LINK_STG_PAY_006 | CASE latest(start/commit/rollback) |

### Public Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | transaction_guid | varchar | Transaction GUID | ABC123 | ETL | LINK_PUB_PAY_001 | Pass-through |
| public.fact_transaction | one row per transaction_guid | transaction_id | bigint | Transaction ID | 354 | ETL | LINK_PUB_PAY_002 | Pass-through |
| public.fact_transaction | one row per transaction_guid | transaction_start_datetime | timestamp | Start timestamp (UTC) | 2026-09-29 14:58:11 | ETL | LINK_PUB_PAY_003 | Pass-through |
| public.fact_transaction | one row per transaction_guid | transaction_commit_datetime | timestamp | Commit timestamp (UTC) | 2026-09-29 15:00:05 | ETL | LINK_PUB_PAY_004 | Pass-through |
| public.fact_transaction | one row per transaction_guid | transaction_rollback_datetime | timestamp | Rollback timestamp (UTC) | 2026-09-29 15:03:40 | ETL | LINK_PUB_PAY_005 | Pass-through |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime | timestamp | Final event timestamp (UTC) | 2026-09-29 15:03:40 | Derived | LINK_PUB_PAY_006 | Pass-through of CASE result |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime_local | timestamp | Local final event timestamp | 2026-09-29 08:03:40 | Derived | LINK_PUB_PAY_007 | Localized timestamp |
| public.fact_transaction | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime | 2026-10-03 19:30:00 | Backend | LINK_PUB_PAY_008 | Source performance local time |
| public.fact_transaction | one row per transaction_guid | gross_transaction_amount | numeric(19,4) | Gross transaction amount | 42.50 | ETL | LINK_PUB_PAY_009 | Base financial metric |
| public.fact_transaction | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount | Purchase: 0.00 / Refund: -42.50 | Derived | LINK_PUB_PAY_010 | Return adjustments |
| public.fact_transaction | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | Purchase: 0.00 / Refund: -5.00 | Derived | LINK_PUB_PAY_011 | Fee return adjustments |

### Analytics Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_guid | varchar | Unique transaction GUID | ABC123 | ETL | LINK_AN_PAY_001 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | transaction_id | bigint | Transaction ID | 354 | ETL | LINK_AN_PAY_002 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | transaction_start_datetime_utc | timestamp | Start time UTC | 2026-09-29 14:58:11 | ETL | LINK_AN_PAY_003 | From public.fact_transaction.transaction_start_datetime |
| analytics.transactions_data_source | one row per transaction_guid | transaction_start_datetime_los_angeles | timestamp | Start time Pacific | 2026-09-29 07:58:11 | Derived | LINK_AN_PAY_004 | CONVERT_TIMEZONE('US/Pacific', start_utc) |
| analytics.transactions_data_source | one row per transaction_guid | transaction_commit_datetime_utc | timestamp | Commit time UTC | 2026-09-29 15:00:05 | ETL | LINK_AN_PAY_005 | From public.fact_transaction.transaction_commit_datetime |
| analytics.transactions_data_source | one row per transaction_guid | transaction_commit_datetime_los_angeles | timestamp | Commit time Pacific | 2026-09-29 08:00:05 | Derived | LINK_AN_PAY_006 | CONVERT_TIMEZONE('US/Pacific', commit_utc) |
| analytics.transactions_data_source | one row per transaction_guid | transaction_rollback_datetime_utc | timestamp | Rollback time UTC | 2026-09-29 15:03:40 | ETL | LINK_AN_PAY_007 | From public.fact_transaction.transaction_rollback_datetime |
| analytics.transactions_data_source | one row per transaction_guid | transaction_rollback_datetime_los_angeles | timestamp | Rollback time Pacific | 2026-09-29 08:03:40 | Derived | LINK_AN_PAY_008 | CONVERT_TIMEZONE('US/Pacific', rollback_utc) |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_utc | timestamp | Final event UTC | 2026-09-29 15:03:40 | Derived | LINK_AN_PAY_009 | Final lifecycle timestamp |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_los_angeles | timestamp | Final event Pacific | 2026-09-29 08:03:40 | Derived | LINK_AN_PAY_010 | CONVERT_TIMEZONE('US/Pacific', end_utc) |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_local | timestamp | Local end timestamp | 2026-09-29 08:03:40 | Derived | LINK_AN_PAY_011 | From public local projection |
| analytics.transactions_data_source | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime | 2026-10-03 19:30:00 | Backend | LINK_AN_PAY_012 | Pass-through from public.fact_transaction |

---

## Flow Group 2: Amount Fields (Fees, Taxes, Returns, Charity)

### Flow

```text
MSSQL payment/order financials
→ raw_import_cache payment tables
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction + return enrichments
→ analytics.transactions_data_source
```

### Logic Summary

- Amount fields primarily come from `public.fact_transaction`.
- Return fields are populated via return workflows and update logic.
- Refund-sensitive fields show purchase/refund examples where behavior differs.

### Raw Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.payment* | payment event grain | amount components (placeholder) | numeric | Source amount components for fees/tax/tickets | 35.00 | Backend | LINK_RAW_AMT_001 | Source amounts used for ETL aggregates |

### Staging Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | ETL | LINK_STG_AMT_001 | Aggregated from source components |

### Public Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee amount | 5.00 | ETL | LINK_PUB_AMT_001 | Fee component |
| public.fact_transaction | one row per transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee | 0.00 | Derived | LINK_PUB_AMT_002 | Waiver logic |
| public.fact_transaction | one row per transaction_guid | waived_sales_fee_ticket_count | int | Ticket count with waived fees | 0 | Derived | LINK_PUB_AMT_003 | Waived-fee ticket count |
| public.fact_transaction | one row per transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | ETL | LINK_PUB_AMT_004 | Ticket component |
| public.fact_transaction | one row per transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge amount | 1.50 | ETL | LINK_PUB_AMT_005 | Surcharge component |
| public.fact_transaction | one row per transaction_guid | sales_fee_return_amount | numeric(19,4) | Returned sales fee | Purchase: 0.00 / Refund: -5.00 | Derived | LINK_PUB_AMT_006 | Return adjustment |
| public.fact_transaction | one row per transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Returned surcharge | Purchase: 0.00 / Refund: -1.50 | Derived | LINK_PUB_AMT_007 | Return adjustment |
| public.fact_transaction | one row per transaction_guid | reservation_fee_return_amount | numeric(19,4) | Returned reservation fee | Purchase: 0.00 / Refund: -0.75 | Derived | LINK_PUB_AMT_008 | Return adjustment |
| public.fact_transaction | one row per transaction_guid | tax_return_amount | numeric(19,4) | Returned tax | Purchase: 0.00 / Refund: -1.75 | Derived | LINK_PUB_AMT_009 | Return tax reversal |
| public.fact_transaction | one row per transaction_guid | card_fee_return_amount | numeric(19,4) | Returned card fee | Purchase: 0.00 / Refund: -0.25 | Derived | LINK_PUB_AMT_010 | Return card-fee reversal |
| public.fact_transaction | one row per transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee for later-returned purchase | Purchase: 5.00 / Refund: 0.00 | Derived | LINK_PUB_AMT_011 | Tracks prior purchase fee context |
| public.fact_transaction | one row per transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable fee amount | 5.00 | Derived | LINK_PUB_AMT_012 | Taxable fee basis |
| public.fact_transaction | one row per transaction_guid | charity_amount | numeric(19,4) | Charity amount on transaction | 1.00 | Third-party | LINK_PUB_AMT_013 | Donation amount |
| public.fact_transaction | one row per transaction_guid | true_gross_transaction_amount | numeric(19,4) | Gross before charity adjustments | 43.50 | Derived | LINK_PUB_AMT_014 | Adjusted gross |
| public.fact_transaction | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | Purchase: 0.00 / Refund: -1.00 | Derived | LINK_PUB_AMT_015 | Donation return component |
| public.fact_transaction | one row per transaction_guid | charity_return_amount | numeric(19,4) | Charity return value | Purchase: 0.00 / Refund: -1.00 | Derived | LINK_PUB_AMT_016 | Charity return metric |

### Analytics Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | ETL | LINK_AN_AMT_001 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee | 5.00 | ETL | LINK_AN_AMT_002 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived fee amount | 0.00 | Derived | LINK_AN_AMT_003 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | waived_sales_fee_ticket_count | int | Waived-fee ticket count | 0 | Derived | LINK_AN_AMT_004 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount | Purchase: 0.00 / Refund: -42.50 | Derived | LINK_AN_AMT_005 | From return update logic |
| analytics.transactions_data_source | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | Purchase: 0.00 / Refund: -5.00 | Derived | LINK_AN_AMT_006 | From return update logic |
| analytics.transactions_data_source | one row per transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | ETL | LINK_AN_AMT_007 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge amount | 1.50 | ETL | LINK_AN_AMT_008 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_return_amount | numeric(19,4) | Returned sales fee | Purchase: 0.00 / Refund: -5.00 | Derived | LINK_AN_AMT_009 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Returned surcharge | Purchase: 0.00 / Refund: -1.50 | Derived | LINK_AN_AMT_010 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | reservation_fee_return_amount | numeric(19,4) | Returned reservation fee | Purchase: 0.00 / Refund: -0.75 | Derived | LINK_AN_AMT_011 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | tax_return_amount | numeric(19,4) | Returned tax | Purchase: 0.00 / Refund: -1.75 | Derived | LINK_AN_AMT_012 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | card_fee_return_amount | numeric(19,4) | Returned card fee | Purchase: 0.00 / Refund: -0.25 | Derived | LINK_AN_AMT_013 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Fee for returned purchase | Purchase: 5.00 / Refund: 0.00 | Derived | LINK_AN_AMT_014 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived | LINK_AN_AMT_015 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Third-party | LINK_AN_AMT_016 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | true_gross_transaction_amount | numeric(19,4) | True gross amount | 43.50 | Derived | LINK_AN_AMT_017 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | Purchase: 0.00 / Refund: -1.00 | Derived | LINK_AN_AMT_018 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | Purchase: 0.00 / Refund: -1.00 | Derived | LINK_AN_AMT_019 | From public.fact_transaction |

---

## Flow Group 3: Movie Fields

### Flow

```text
MSSQL movie/performance attributes
→ raw_import_cache.transactionPerformanceAttribute
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction + public.dim_movie
→ analytics.transactions_data_source
```

### Logic Summary

- Movie keys are resolved in staging/public.
- Analytics denormalizes child/parent movie attributes and release-based date deltas.

### Raw Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transactionPerformanceAttribute | transaction-performance attribute grain | movie_detail_id (placeholder) | int | Raw movie detail identifier | 48192 | Backend | LINK_RAW_MOV_001 | Source movie/performance identity |

### Staging Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | movie_key | bigint | Surrogate movie key | 129384 | ETL | LINK_STG_MOV_001 | Resolved to conformed movie dimension |

### Public Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_movie | one row per movie_key | movie_id | int | Child movie ID | 10293 | ETL | LINK_PUB_MOV_001 | Conformed movie ID |
| public.dim_movie | one row per movie_key | movie_title | varchar | Child movie title | The Example Movie | ETL | LINK_PUB_MOV_002 | Reporting title |
| public.dim_movie | one row per movie_key | movie_parent_id | int | Parent movie ID | 872 | ETL | LINK_PUB_MOV_003 | Parent franchise grouping |
| public.dim_movie | one row per movie_key | movie_parent_title | varchar | Parent movie title | Example Franchise | ETL | LINK_PUB_MOV_004 | Parent reporting title |
| public.dim_movie | one row per movie_key | movie_release_date | date | Movie release date | 2026-06-14 | Backend | LINK_PUB_MOV_005 | Normalized release date |
| public.dim_movie | one row per movie_key | movie_tms_id | varchar | TMS identifier | MV123456789 | Third-party | LINK_PUB_MOV_006 | External metadata ID |

### Analytics Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | filmid_child | int | Child movie ID | 10293 | ETL | LINK_AN_MOV_001 | From dim_movie.movie_id |
| analytics.transactions_data_source | one row per transaction_guid | filmname_child | varchar | Child movie title | The Example Movie | ETL | LINK_AN_MOV_002 | From dim_movie.movie_title |
| analytics.transactions_data_source | one row per transaction_guid | filmid_parent | int | Parent movie ID | 872 | ETL | LINK_AN_MOV_003 | From dim_movie.movie_parent_id |
| analytics.transactions_data_source | one row per transaction_guid | filmname_parent | varchar | Parent movie title | Example Franchise | ETL | LINK_AN_MOV_004 | From dim_movie.movie_parent_title |
| analytics.transactions_data_source | one row per transaction_guid | releasedate | date | Movie release date | 2026-06-14 | Backend | LINK_AN_MOV_005 | From dim_movie.movie_release_date |
| analytics.transactions_data_source | one row per transaction_guid | movie_tms_id | varchar | TMS ID | MV123456789 | Third-party | LINK_AN_MOV_006 | From dim_movie.movie_tms_id |
| analytics.transactions_data_source | one row per transaction_guid | deltareleasetopurchasedate | int | Days release→purchase | 17 | Derived | LINK_AN_MOV_007 | DATEDIFF(release, purchase_date) |
| analytics.transactions_data_source | one row per transaction_guid | deltareleasetoshowtimedate | int | Days release→showtime | 21 | Derived | LINK_AN_MOV_008 | DATEDIFF(release, showtime_date) |
| analytics.transactions_data_source | one row per transaction_guid | deltapurchasetoshowtimedate | int | Days purchase→showtime | 4 | Derived | LINK_AN_MOV_009 | DATEDIFF(purchase_date, showtime_date) |

---

## Flow Group 4: Customer / Device / Funnel Fields

### Flow

```text
MSSQL customer/session/device/funnel context
→ raw_import_cache.transaction + related attributes
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction + related dimensions
→ analytics.transactions_data_source
```

### Logic Summary

- Device and funnel identifiers are conformed through dimensions.
- Analytics exposes customer behavior context fields and business classifications.

### Raw Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | user_agent_id (placeholder) | int | Raw user-agent ID | 44321 | Backend | LINK_RAW_CUS_001 | Browser/device identity |
| raw_import_cache.transaction | one row per transaction_guid | purchase_funnel_id (placeholder) | int | Raw purchase funnel ID | 14 | Backend | LINK_RAW_CUS_002 | Funnel attribution from source |

### Staging Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | user_agent_key | bigint | User-agent surrogate key | 298771 | ETL | LINK_STG_CUS_001 | Map raw user_agent to conformed dimension |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | purchase_funnel_key | bigint | Funnel surrogate key | 7231 | ETL | LINK_STG_CUS_002 | Map raw funnel to conformed dimension |

### Public Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_user_agent | one row per user_agent_key | user_agent_id | varchar | User-agent ID | UA_44321 | ETL | LINK_PUB_CUS_001 | Conformed ID |
| public.dim_user_agent | one row per user_agent_key | user_agent_description | varchar | User-agent descriptor | iOS Safari | ETL | LINK_PUB_CUS_002 | Friendly browser/device string |
| public.dim_purchase_funnel | one row per purchase_funnel_key | purchase_funnel_id | varchar | Funnel ID | 14 | ETL | LINK_PUB_CUS_003 | Conformed funnel ID |
| public.dim_purchase_funnel | one row per purchase_funnel_key | purchase_funnel_name | varchar | Funnel name | Guest Checkout | ETL | LINK_PUB_CUS_004 | Reporting funnel name |
| public.fact_transaction | one row per transaction_guid | deviceType | varchar | Device class | Mobile | Backend | LINK_PUB_CUS_005 | Source-derived device class |
| public.fact_transaction | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY12345 | Backend | LINK_PUB_CUS_006 | Loyalty/member identifier |

### Analytics Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | user_agent_id | varchar | User-agent ID | UA_44321 | ETL | LINK_AN_CUS_001 | From dim_user_agent |
| analytics.transactions_data_source | one row per transaction_guid | user_agent_description | varchar | User-agent descriptor | iOS Safari | ETL | LINK_AN_CUS_002 | From dim_user_agent |
| analytics.transactions_data_source | one row per transaction_guid | purchase_funnel_id | varchar | Funnel ID | 14 | ETL | LINK_AN_CUS_003 | From dim_purchase_funnel |
| analytics.transactions_data_source | one row per transaction_guid | purchase_funnel_name | varchar | Funnel name | Guest Checkout | ETL | LINK_AN_CUS_004 | From dim_purchase_funnel |
| analytics.transactions_data_source | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Derived | LINK_AN_CUS_005 | Device classification for reporting |
| analytics.transactions_data_source | one row per transaction_guid | platform | varchar | Platform | Mobile Web | Derived | LINK_AN_CUS_006 | CASE logic from device + UA |
| analytics.transactions_data_source | one row per transaction_guid | apps | varchar | App category | Fandango App: iPhone | Derived | LINK_AN_CUS_007 | Business grouping by app/channel/device |
| analytics.transactions_data_source | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY12345 | Backend | LINK_AN_CUS_008 | Pass-through from public.fact_transaction |

---

## Flow Group 5: Theater / Location Fields

### Flow

```text
MSSQL theater/location metadata
→ raw_import_cache location + DMA source
→ warehouse_loader keys
→ public.dim_theater + public.dim_chain
→ analytics.transactions_data_source
```

### Logic Summary

- Theater and chain dimensions provide location + operator context.
- DMA market is joined by ZIP code.

### Raw Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.designatedMarketArea | one row per zip_code | zip_code | varchar | ZIP for DMA mapping | 90028 | Third-party | LINK_RAW_LOC_001 | ZIP used to map market |
| raw_import_cache.designatedMarketArea | one row per zip_code | dma_market_name | varchar | DMA market label | Los Angeles | Third-party | LINK_RAW_LOC_002 | Market name by ZIP |

### Staging Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | theater_key | bigint | Theater surrogate key | 11872 | ETL | LINK_STG_LOC_001 | Resolve to dim_theater |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | chain_key | bigint | Chain surrogate key | 392 | ETL | LINK_STG_LOC_002 | Resolve to dim_chain |

### Public Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_theater | one row per theater_key | theater_id | int | Theater ID | 4451 | ETL | LINK_PUB_LOC_001 | Conformed theater ID |
| public.dim_theater | one row per theater_key | theater_code | varchar | Theater code | TH4451 | ETL | LINK_PUB_LOC_002 | Business theater code |
| public.dim_theater | one row per theater_key | theater_name | varchar | Theater name | Example Cinemas Hollywood | ETL | LINK_PUB_LOC_003 | Reporting name |
| public.dim_theater | one row per theater_key | pdi_id | varchar | PDI ID | PDI9981 | ETL | LINK_PUB_LOC_004 | Cross-system location ID |
| public.dim_theater | one row per theater_key | point_of_sale_theater_id | varchar | POS theater ID | POS_TH_4451 | Backend | LINK_PUB_LOC_005 | External POS theater ID |
| public.dim_theater | one row per theater_key | latitude | numeric | Latitude | 34.1021 | Backend | LINK_PUB_LOC_006 | Coordinate |
| public.dim_theater | one row per theater_key | longitude | numeric | Longitude | -118.3402 | Backend | LINK_PUB_LOC_007 | Coordinate |
| public.dim_theater | one row per theater_key | point_of_sale_id | varchar | POS ID | POS01 | Backend | LINK_PUB_LOC_008 | POS system identifier |
| public.dim_theater | one row per theater_key | address_1 | varchar | Address line 1 | 1234 Example Blvd | Backend | LINK_PUB_LOC_009 | Street address |
| public.dim_theater | one row per theater_key | address_2 | varchar | Address line 2 | Suite 200 | Backend | LINK_PUB_LOC_010 | Secondary address |
| public.dim_theater | one row per theater_key | city_name | varchar | City | Los Angeles | Backend | LINK_PUB_LOC_011 | City |
| public.dim_theater | one row per theater_key | state_code | varchar | State | CA | Backend | LINK_PUB_LOC_012 | State code |
| public.dim_theater | one row per theater_key | zip_code | varchar | ZIP | 90028 | Backend | LINK_PUB_LOC_013 | ZIP |
| public.dim_theater | one row per theater_key | country_code | varchar | Country code | US | Backend | LINK_PUB_LOC_014 | Country |
| public.dim_theater | one row per theater_key | braintree_merchant_id | varchar | Braintree merchant ID | bt_merchant_123 | Third-party | LINK_PUB_LOC_015 | Payment merchant mapping |
| public.dim_chain | one row per chain_key | chain_name | varchar | Chain name | Example Cinemas | ETL | LINK_PUB_LOC_016 | Chain descriptor |
| public.dim_chain | one row per chain_key | chain_id | varchar | Chain ID | CHN019 | ETL | LINK_PUB_LOC_017 | Chain identifier |
| public.dim_chain | one row per chain_key | point_of_sale_vendor | varchar | POS vendor | Vista | Third-party | LINK_PUB_LOC_018 | POS vendor name |
| public.dim_chain | one row per chain_key | awards_program_name | varchar | Awards program | Example Rewards | Backend | LINK_PUB_LOC_019 | Program descriptor |

### Analytics Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | theater_id | int | Theater ID | 4451 | ETL | LINK_AN_LOC_001 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | theater_code | varchar | Theater code | TH4451 | ETL | LINK_AN_LOC_002 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | theater_name | varchar | Theater name | Example Cinemas Hollywood | ETL | LINK_AN_LOC_003 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | pdi_id | varchar | PDI ID | PDI9981 | ETL | LINK_AN_LOC_004 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | pos_theater_id | varchar | POS theater ID | POS_TH_4451 | Backend | LINK_AN_LOC_005 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | latitude | numeric | Latitude | 34.1021 | Backend | LINK_AN_LOC_006 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | longitude | numeric | Longitude | -118.3402 | Backend | LINK_AN_LOC_007 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | point_of_sale_id | varchar | POS ID | POS01 | Backend | LINK_AN_LOC_008 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | address_1 | varchar | Address line 1 | 1234 Example Blvd | Backend | LINK_AN_LOC_009 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | address_2 | varchar | Address line 2 | Suite 200 | Backend | LINK_AN_LOC_010 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | city_name | varchar | City | Los Angeles | Backend | LINK_AN_LOC_011 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | state_code | varchar | State | CA | Backend | LINK_AN_LOC_012 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | zip_code | varchar | ZIP | 90028 | Backend | LINK_AN_LOC_013 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | country_code | varchar | Country | US | Backend | LINK_AN_LOC_014 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | chainname | varchar | Chain name | Example Cinemas | ETL | LINK_AN_LOC_015 | From dim_chain |
| analytics.transactions_data_source | one row per transaction_guid | chainid | varchar | Chain ID | CHN019 | ETL | LINK_AN_LOC_016 | From dim_chain |
| analytics.transactions_data_source | one row per transaction_guid | pos_vendor | varchar | POS vendor | Vista | Third-party | LINK_AN_LOC_017 | From dim_chain |
| analytics.transactions_data_source | one row per transaction_guid | awards_program_name | varchar | Awards program | Example Rewards | Backend | LINK_AN_LOC_018 | From dim_chain |
| analytics.transactions_data_source | one row per transaction_guid | braintree_merchant_id | varchar | Merchant ID | bt_merchant_123 | Third-party | LINK_AN_LOC_019 | From dim_theater |
| analytics.transactions_data_source | one row per transaction_guid | dma_market_name | varchar | DMA market | Los Angeles | Third-party | LINK_AN_LOC_020 | ZIP-based DMA join |

---

## Flow Group 6: Affiliate / Channel / Partner Fields

### Flow

```text
MSSQL affiliate + channel + partner metadata
→ raw_import_cache.transaction affiliate markers
→ warehouse_loader keys
→ public affiliate/channel/partner dimensions
→ analytics.transactions_data_source
```

### Logic Summary

- Attribution IDs are conformed in staging/public.
- Analytics includes both detailed and grouped attribution fields.

### Raw Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Affiliate_id | int | Source affiliate ID | 132 | Backend | LINK_RAW_AFF_001 | Source attribution ID |

### Staging Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | affiliate_key | bigint | Affiliate surrogate key | 8123 | ETL | LINK_STG_AFF_001 | Resolve to dim_affiliate |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | channel_key | bigint | Channel surrogate key | 291 | ETL | LINK_STG_AFF_002 | Resolve to dim_channel |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_channel_key | bigint | Partner channel key | 912 | ETL | LINK_STG_AFF_003 | Resolve to partner channel dimension |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_platform_key | bigint | Partner platform key | 117 | ETL | LINK_STG_AFF_004 | Resolve to partner platform dimension |

### Public Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_affiliate | one row per affiliate_key | affiliate_id | int | Affiliate ID | 132 | ETL | LINK_PUB_AFF_001 | Conformed affiliate ID |
| public.dim_affiliate | one row per affiliate_key | affiliate_name | varchar | Affiliate name | Fandango iOS App | ETL | LINK_PUB_AFF_002 | Attribution name |
| public.dim_affiliate | one row per affiliate_key | affiliate_group_name | varchar | Affiliate group | Mobile App | Derived | LINK_PUB_AFF_003 | Business grouping |
| public.dim_affiliate | one row per affiliate_key | affiliate_category_name | varchar | Affiliate category | Owned & Operated | Derived | LINK_PUB_AFF_004 | Business category |
| public.dim_channel | one row per channel_key | channel_group | varchar | Channel group | App | Derived | LINK_PUB_AFF_005 | Grouped channel |
| public.dim_channel | one row per channel_key | channel_name | varchar | Channel name | iOS | Derived | LINK_PUB_AFF_006 | Detailed channel |
| public.dim_partner_channel | one row per partner_channel_key | partner_channel_id | varchar | Partner channel ID | TMO | Third-party | LINK_PUB_AFF_007 | Partner channel code |
| public.dim_partner_channel | one row per partner_channel_key | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party | LINK_PUB_AFF_008 | Friendly partner name |
| public.dim_partner_platform | one row per partner_platform_key | partner_platform_id | varchar | Partner platform ID | WEBVIEW | Third-party | LINK_PUB_AFF_009 | Partner platform code |

### Analytics Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | affiliate_id | int | Affiliate ID | 132 | ETL | LINK_AN_AFF_001 | From dim_affiliate |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_name | varchar | Affiliate name | Fandango iOS App | ETL | LINK_AN_AFF_002 | From dim_affiliate |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_group_name | varchar | Affiliate group | Mobile App | Derived | LINK_AN_AFF_003 | From dim_affiliate |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_category_name | varchar | Affiliate category | Owned & Operated | Derived | LINK_AN_AFF_004 | From dim_affiliate |
| analytics.transactions_data_source | one row per transaction_guid | channel | varchar | Channel group | App | Derived | LINK_AN_AFF_005 | From dim_channel.channel_group |
| analytics.transactions_data_source | one row per transaction_guid | channel_ungrouped | varchar | Channel name | iOS | Derived | LINK_AN_AFF_006 | From dim_channel.channel_name |
| analytics.transactions_data_source | one row per transaction_guid | partner_channel | varchar | Partner channel | TMO | Third-party | LINK_AN_AFF_007 | From dim_partner_channel |
| analytics.transactions_data_source | one row per transaction_guid | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party | LINK_AN_AFF_008 | From dim_partner_channel |
| analytics.transactions_data_source | one row per transaction_guid | partner_platform | varchar | Partner platform | WEBVIEW | Third-party | LINK_AN_AFF_009 | From dim_partner_platform |
| analytics.transactions_data_source | one row per transaction_guid | partner_defined_data_value | varchar | Partner payload | campaign=fall_promo | Third-party | LINK_AN_AFF_010 | Pass-through metadata |
| analytics.transactions_data_source | one row per transaction_guid | Affiliate Grouping | varchar | Detailed business grouping | iPhone App | Derived | LINK_AN_AFF_011 | CASE logic over affiliate/channel |
| analytics.transactions_data_source | one row per transaction_guid | Affiliate Grouping - HIGH LEVEL BINS | varchar | High-level grouping | Fandango O&O | Derived | LINK_AN_AFF_012 | Coarse CASE grouping |

---

## Flow Group 7: Transaction Type / Return Type / Property Fields

### Flow

```text
MSSQL transaction metadata
→ raw_import_cache type/property markers
→ warehouse_loader keys
→ public reference dimensions
→ analytics.transactions_data_source
```

### Logic Summary

- Reference dimensions provide transaction type, return type, and property classification labels.

### Raw Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | transaction_type_id (placeholder) | int | Source transaction type ID | 2 | Backend | LINK_RAW_REF_001 | Purchase/refund type marker |

### Staging Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_type_key | bigint | Transaction type key | 82 | ETL | LINK_STG_REF_001 | Resolve to dim_transaction_type |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | return_type_key | bigint | Return type key | 16 | ETL | LINK_STG_REF_002 | Resolve to dim_return_type |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | property_key | bigint | Property key | 45 | ETL | LINK_STG_REF_003 | Resolve to dim_property |

### Public Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_transaction_type | one row per transaction_type_key | transaction_type_id | varchar | Transaction type ID | 2 | ETL | LINK_PUB_REF_001 | Conformed type ID |
| public.dim_transaction_type | one row per transaction_type_key | transaction_type_name | varchar | Transaction type name | Purchase | Derived | LINK_PUB_REF_002 | Friendly type label |
| public.dim_return_type | one row per return_type_key | return_method | varchar | Return method | Self-service | Derived | LINK_PUB_REF_003 | Return channel |
| public.dim_return_type | one row per return_type_key | return_type_code | varchar | Return code | CXL | Backend | LINK_PUB_REF_004 | Return code label |
| public.dim_return_type | one row per return_type_key | return_type_description | varchar | Return description | Customer cancellation | Derived | LINK_PUB_REF_005 | Friendly return description |
| public.dim_property | one row per property_key | property_id | varchar | Property ID | FDG | Backend | LINK_PUB_REF_006 | Property identifier |
| public.dim_property | one row per property_key | property_group | varchar | Property group | Fandango | Derived | LINK_PUB_REF_007 | Property grouping |

### Analytics Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_type_id | varchar | Transaction type ID | 2 | ETL | LINK_AN_REF_001 | From dim_transaction_type |
| analytics.transactions_data_source | one row per transaction_guid | transaction_type_name | varchar | Transaction type name | Purchase | Derived | LINK_AN_REF_002 | From dim_transaction_type |
| analytics.transactions_data_source | one row per transaction_guid | return_method | varchar | Return method | Self-service | Derived | LINK_AN_REF_003 | From dim_return_type |
| analytics.transactions_data_source | one row per transaction_guid | return_type_code | varchar | Return type code | CXL | Backend | LINK_AN_REF_004 | From dim_return_type |
| analytics.transactions_data_source | one row per transaction_guid | return_type_description | varchar | Return description | Customer cancellation | Derived | LINK_AN_REF_005 | From dim_return_type |
| analytics.transactions_data_source | one row per transaction_guid | property_id | varchar | Property ID | FDG | Backend | LINK_AN_REF_006 | From dim_property |
| analytics.transactions_data_source | one row per transaction_guid | property_group | varchar | Property group | Fandango | Derived | LINK_AN_REF_007 | From dim_property |

---

## Flow Group 8: Flags + Operational Fields

### Flow

```text
MSSQL transaction/product attributes
→ raw_import_cache.transaction
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction
→ analytics.transactions_data_source
```

### Logic Summary

- Most flags are pass-through or light business derivations in public/analytics.
- Includes format flags, reserved seating, POS confirmation, and success indicator.

### Raw Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | is_test (placeholder) | bit/boolean | Test marker | 0 | Backend | LINK_RAW_FLG_001 | Source QA/test marker |

### Staging Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | item_count | int | Item count | 2 | ETL | LINK_STG_FLG_001 | Transaction item aggregation |

### Public Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | payment_types | varchar | Payment type(s) | VISA | Backend | LINK_PUB_FLG_001 | Normalized payment labels |
| public.fact_transaction | one row per transaction_guid | processor_order_ids | varchar | Processor order IDs | ord_7891001 | Third-party | LINK_PUB_FLG_002 | External processor references |
| public.fact_transaction | one row per transaction_guid | is_test | boolean | Is test transaction | false | Backend | LINK_PUB_FLG_003 | Source test flag |
| public.fact_transaction | one row per transaction_guid | is_fanclub | boolean | FanClub flag | true | Derived | LINK_PUB_FLG_004 | Business classification |
| public.fact_transaction | one row per transaction_guid | is_buynow | boolean | Buy now flag | false | Derived | LINK_PUB_FLG_005 | Flow classification |
| public.fact_transaction | one row per transaction_guid | network_token_usage | varchar/boolean | Network token usage | true | Third-party | LINK_PUB_FLG_006 | Tokenization indicator |
| public.fact_transaction | one row per transaction_guid | is_plf | boolean | PLF flag | true | Derived | LINK_PUB_FLG_007 | Format classification |
| public.fact_transaction | one row per transaction_guid | is_imax | boolean | IMAX flag | false | Derived | LINK_PUB_FLG_008 | Format classification |
| public.fact_transaction | one row per transaction_guid | is_3d | boolean | 3D flag | false | Derived | LINK_PUB_FLG_009 | Format classification |
| public.fact_transaction | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | true | Derived | LINK_PUB_FLG_010 | Premium amenity indicator |
| public.fact_transaction | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_PUB_FLG_011 | POS confirmation |
| public.fact_transaction | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Backend | LINK_PUB_FLG_012 | Seating mode indicator |
| public.fact_transaction | one row per transaction_guid | reserved_seats | varchar/int | Reserved seats | A10,A11 | Backend | LINK_PUB_FLG_013 | Seat details |
| public.fact_transaction | one row per transaction_guid | item_count | int | Item count | 2 | ETL | LINK_PUB_FLG_014 | From staging |
| public.fact_transaction | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Third-party | LINK_PUB_FLG_015 | Donation target |

### Analytics Fields

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | payment_types | varchar | Payment methods | VISA | Backend | LINK_AN_FLG_001 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | processor_order_ids | varchar | Processor order IDs | ord_7891001 | Third-party | LINK_AN_FLG_002 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_test | boolean | Test flag | false | Backend | LINK_AN_FLG_003 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_fanclub | boolean | FanClub flag | true | Derived | LINK_AN_FLG_004 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_buynow | boolean | Buy-now flag | false | Derived | LINK_AN_FLG_005 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | network_token_usage | varchar/boolean | Network token usage | true | Third-party | LINK_AN_FLG_006 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_plf | boolean | PLF flag | true | Derived | LINK_AN_FLG_007 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_imax | boolean | IMAX flag | false | Derived | LINK_AN_FLG_008 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_3d | boolean | 3D flag | false | Derived | LINK_AN_FLG_009 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity | true | Derived | LINK_AN_FLG_010 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_AN_FLG_011 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Backend | LINK_AN_FLG_012 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | reserved_seats | varchar/int | Reserved seat info | A10,A11 | Backend | LINK_AN_FLG_013 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | item_qty | int | Item quantity | 2 | ETL | LINK_AN_FLG_014 | Renamed from item_count |
| analytics.transactions_data_source | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Third-party | LINK_AN_FLG_015 | From public.fact_transaction |
| analytics.transactions_data_source | one row per transaction_guid | is_successful | boolean | Successful transaction flag | true | Derived | LINK_AN_FLG_016 | Commit present and no rollback rule |

---

## Field Coverage Checklist (96)

Use this checklist to confirm all analytics fields are represented in grouped tables:

- Identity keys ✅
- All datetime UTC/Pacific/local lifecycle fields ✅
- All amount/fee/tax/return/charity fields ✅
- Movie/dimension descriptors ✅
- Customer/device/funnel fields ✅
- Theater/location/chain fields ✅
- Affiliate/channel/partner fields ✅
- Flags + operational fields ✅

---

## Placeholder Link Convention

Replace placeholders by layer:

- `LINK_RAW_*` → raw SQL/DAG source for `raw_import_cache`
- `LINK_STG_*` → `warehouse_loader` transformation SQL
- `LINK_PUB_*` → `public` model SQL
- `LINK_AN_*` → analytics SQL (e.g., `analyticsTransactionsDataSource.sql`)

Recommended format:

```text
https://github.com/<owner>/<repo>/blob/main/path/to/file.sql#L120-L145
```

---

## Notes

- This grouped format intentionally removes repetitive per-field narrative.
- It preserves lineage clarity while being shorter and easier to maintain.
- Next iteration: replace placeholders with exact row-level permalinks.
