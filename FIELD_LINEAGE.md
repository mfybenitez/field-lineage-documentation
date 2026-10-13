# analytics.transactions_data_source — Field Lineage (Grouped, 96 Fields)

## Overview

This document traces all **96 fields** in `analytics.transactions_data_source`, backtraced from the analytics insert SQL and then presented in forward flow:

```text
Raw (raw_import_cache)
→ Staging (warehouse_loader.fact_transaction_base)
→ Public (public.fact_transaction + public dimensions)
→ Analytics (analytics.transactions_data_source)
```

### Source Value Rules

Only these values are used in **Source**:

- `Backend`
- `ETL`
- `Third-party`
- `Derived`

### Source Inheritance Rule

If a value is direct pass-through (rename only, no business logic), Source inherits from upstream origin (e.g., `transaction_guid` remains `Backend`, not `ETL`).

---

## Group 1: Dates

### Flow

```text
tblTransaction / tblTransactionCommitted / tblTransactionRolledBack / tblTransaction_PerformanceAttribute
→ raw_import_cache.transaction / transactionCommitted / transactionRolledBack / transactionPerformanceAttribute
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction
→ analytics.transactions_data_source
```

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Create_dm | timestamp | Transaction created datetime (UTC) | 2026-09-29 14:58:11 | Backend | LINK_RAW_DATES_001 | Extracted from `tblTransaction.Create_dm` |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Create_dm | timestamp | Transaction commit datetime | 2026-09-29 15:00:05 | Backend | LINK_RAW_DATES_002 | Extracted from `tblTransactionCommitted.Create_dm` |
| raw_import_cache.transactionRolledBack | one row per rolledback transaction_guid | Create_dm | timestamp | Transaction rollback datetime | 2026-09-29 15:03:40 | Backend | LINK_RAW_DATES_003 | From rolled-back transaction source table |
| raw_import_cache.transactionPerformanceAttribute | one row per transaction-performance | Performance_dm | timestamp | Local performance/showtime datetime | 2026-10-03 19:30:00 | Backend | LINK_RAW_DATES_004 | Extracted from `tblTransaction_PerformanceAttribute.Performance_dm` |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_start_datetime | timestamp | Transaction start datetime UTC | 2026-09-29 14:58:11 | Backend | LINK_STG_DATES_001 | `t.Create_dm::timestamp` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_commit_datetime | timestamp | Commit datetime UTC | 2026-09-29 15:00:05 | Backend | LINK_STG_DATES_002 | `tc.create_dm` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_rollback_datetime | timestamp | Rollback datetime UTC | 2026-09-29 15:03:40 | Backend | LINK_STG_DATES_003 | `tr.create_dm` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_end_datetime | timestamp | Final event datetime UTC | 2026-09-29 15:03:40 | Derived | LINK_STG_DATES_004 | CASE over commit/rollback/start |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | performance_datetime_local | timestamp | Local performance datetime | 2026-10-03 19:30:00 | Backend | LINK_STG_DATES_005 | `tpa.Performance_dm` |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | transaction_start_datetime | timestamp | Start datetime UTC | 2026-09-29 14:58:11 | Backend | LINK_PUB_DATES_001 | Pass-through from staging |
| public.fact_transaction | one row per transaction_guid | transaction_commit_datetime | timestamp | Commit datetime UTC | 2026-09-29 15:00:05 | Backend | LINK_PUB_DATES_002 | Pass-through from staging |
| public.fact_transaction | one row per transaction_guid | transaction_rollback_datetime | timestamp | Rollback datetime UTC | 2026-09-29 15:03:40 | Backend | LINK_PUB_DATES_003 | Pass-through from staging |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime | timestamp | Final datetime UTC | 2026-09-29 15:03:40 | Derived | LINK_PUB_DATES_004 | Pass-through derived staging value |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime_local | timestamp | Local final datetime | 2026-09-29 08:03:40 | Derived | LINK_PUB_DATES_005 | Localized in public layer |
| public.fact_transaction | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime | 2026-10-03 19:30:00 | Backend | LINK_PUB_DATES_006 | Pass-through from staging |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_start_datetime_utc | timestamp | Start datetime UTC | 2026-09-29 14:58:11 | Backend | LINK_AN_DATES_001 | `ft.transaction_start_datetime` |
| analytics.transactions_data_source | one row per transaction_guid | transaction_start_datetime_los_angeles | timestamp | Start datetime Pacific | 2026-09-29 07:58:11 | Derived | LINK_AN_DATES_002 | `CONVERT_TIMEZONE('US/Pacific', ft.transaction_start_datetime)` |
| analytics.transactions_data_source | one row per transaction_guid | transaction_commit_datetime_utc | timestamp | Commit datetime UTC | 2026-09-29 15:00:05 | Backend | LINK_AN_DATES_003 | `ft.transaction_commit_datetime` |
| analytics.transactions_data_source | one row per transaction_guid | transaction_commit_datetime_los_angeles | timestamp | Commit datetime Pacific | 2026-09-29 08:00:05 | Derived | LINK_AN_DATES_004 | Timezone conversion |
| analytics.transactions_data_source | one row per transaction_guid | transaction_rollback_datetime_utc | timestamp | Rollback datetime UTC | 2026-09-29 15:03:40 | Backend | LINK_AN_DATES_005 | `ft.transaction_rollback_datetime` |
| analytics.transactions_data_source | one row per transaction_guid | transaction_rollback_datetime_los_angeles | timestamp | Rollback datetime Pacific | 2026-09-29 08:03:40 | Derived | LINK_AN_DATES_006 | Timezone conversion |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_utc | timestamp | Final event datetime UTC | 2026-09-29 15:03:40 | Derived | LINK_AN_DATES_007 | `ft.transaction_end_datetime` (derived upstream) |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_los_angeles | timestamp | Final event datetime Pacific | 2026-09-29 08:03:40 | Derived | LINK_AN_DATES_008 | Timezone conversion |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_local | timestamp | Final event local datetime | 2026-09-29 08:03:40 | Derived | LINK_AN_DATES_009 | `ft.transaction_end_datetime_local` |
| analytics.transactions_data_source | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-03 19:30:00 | Backend | LINK_AN_DATES_010 | `ft.performance_datetime_local` |
| analytics.transactions_data_source | one row per transaction_guid | deltareleasetopurchasedate | int | Days release→purchase | 17 | Derived | LINK_AN_DATES_011 | DATEDIFF(movie_release_date, purchase day) |
| analytics.transactions_data_source | one row per transaction_guid | deltareleasetoshowtimedate | int | Days release→showtime | 21 | Derived | LINK_AN_DATES_012 | DATEDIFF(movie_release_date, showtime day) |
| analytics.transactions_data_source | one row per transaction_guid | deltapurchasetoshowtimedate | int | Days purchase→showtime | 4 | Derived | LINK_AN_DATES_013 | DATEDIFF(purchase day, showtime day) |

---

## Group 2: Amounts

### Flow

```text
tblTransaction_Item + related payment/return upstream feeds
→ raw_import_cache.transactionItem (+ other raw financial feeds)
→ warehouse_loader.fact_transaction_base and/or downstream financial build
→ public.fact_transaction
→ analytics.transactions_data_source
```

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transactionItem | one row per transaction item | Item_at | numeric | Item amount | 35.00 | Backend | LINK_RAW_AMT_001 | Extracted from `tblTransaction_Item.Item_at` |
| raw_import_cache.transactionItem | one row per transaction item | Tax_at | numeric | Tax amount | 1.75 | Backend | LINK_RAW_AMT_002 | Extracted from `tblTransaction_Item.Tax_at` |
| raw_import_cache.transactionItem | one row per transaction item | SalesFee_at | numeric | Sales fee amount | 5.00 | Backend | LINK_RAW_AMT_003 | Extracted from `tblTransaction_Item.SalesFee_at` |
| raw_import_cache.transactionItem | one row per transaction item | SurchargeFee_at | numeric | Surcharge fee amount | 1.50 | Backend | LINK_RAW_AMT_004 | Extracted from `tblTransaction_Item.SurchargeFee_at` |
| raw_import_cache.transactionItem | one row per transaction item | ReservedFee_at | numeric | Reservation fee amount | 0.75 | Backend | LINK_RAW_AMT_005 | Extracted from `tblTransaction_Item.ReservedFee_at` |
| raw_import_cache.transactionItem | one row per transaction item | WaivedSalesFee_at | numeric | Waived sales fee amount | 0.00 | Backend | LINK_RAW_AMT_006 | Extracted from `tblTransaction_Item.WaivedSalesFee_at` |
| raw_import_cache.transactionItem | one row per transaction item | CardFee_at | numeric | Card fee amount | 0.25 | Backend | LINK_RAW_AMT_007 | Extracted from `tblTransaction_Item.CardFee_at` |
| raw_import_cache.transactionItem | one row per transaction item | taxable_at | numeric | Taxable amount | 5.00 | Backend | LINK_RAW_AMT_008 | Extracted taxable basis |
| raw_import_cache.transactionItem | one row per transaction item | Item_qy / TotalAdmissions_qy | int | Quantity metrics | 2 | Backend | LINK_RAW_AMT_009 | Item and admission counts |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | (financial totals populated downstream) | various | Financial totals generally finalized in public.fact_transaction | n/a | ETL | LINK_STG_AMT_001 | Staging focuses on keys/flags; final amounts selected from public.fact_transaction in analytics SQL |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | Backend | LINK_PUB_AMT_001 | Upstream transaction amount rollup |
| public.fact_transaction | one row per transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee amount | 5.00 | Backend | LINK_PUB_AMT_002 | Upstream fee rollup |
| public.fact_transaction | one row per transaction_guid | waived_sales_fee_ticket_count | int | Count of waived sales fee tickets | 0 | Derived | LINK_PUB_AMT_003 | Aggregated count |
| public.fact_transaction | one row per transaction_guid | waived_fanclub_fee_ticket_count | int | Count of waived fanclub fee tickets | 0 | Derived | LINK_PUB_AMT_004 | Aggregated count |
| public.fact_transaction | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount | Purchase: 0.00 / Refund: -42.50 | Derived | LINK_PUB_AMT_005 | Updated by return logic |
| public.fact_transaction | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | Purchase: 0.00 / Refund: -5.00 | Derived | LINK_PUB_AMT_006 | Return-related fee update |
| public.fact_transaction | one row per transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Backend | LINK_PUB_AMT_007 | Upstream ticket component |
| public.fact_transaction | one row per transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Backend | LINK_PUB_AMT_008 | Upstream amount |
| public.fact_transaction | one row per transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Backend | LINK_PUB_AMT_009 | Upstream amount |
| public.fact_transaction | one row per transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee amount | 1.50 | Backend | LINK_PUB_AMT_010 | Upstream amount |
| public.fact_transaction | one row per transaction_guid | sales_fee_return_amount | numeric(19,4) | Returned sales fee | Purchase: 0.00 / Refund: -5.00 | Derived | LINK_PUB_AMT_011 | Return metric |
| public.fact_transaction | one row per transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Returned surcharge | Purchase: 0.00 / Refund: -1.50 | Derived | LINK_PUB_AMT_012 | Return metric |
| public.fact_transaction | one row per transaction_guid | reservation_fee_return_amount | numeric(19,4) | Returned reservation fee | Purchase: 0.00 / Refund: -0.75 | Derived | LINK_PUB_AMT_013 | Return metric |
| public.fact_transaction | one row per transaction_guid | tax_return_amount | numeric(19,4) | Returned tax amount | Purchase: 0.00 / Refund: -1.75 | Derived | LINK_PUB_AMT_014 | Return tax metric |
| public.fact_transaction | one row per transaction_guid | card_fee_return_amount | numeric(19,4) | Returned card fee amount | Purchase: 0.00 / Refund: -0.25 | Derived | LINK_PUB_AMT_015 | Return metric |
| public.fact_transaction | one row per transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee tied to later-returned purchase | 5.00 | Derived | LINK_PUB_AMT_016 | Return attribution metric |
| public.fact_transaction | one row per transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived | LINK_PUB_AMT_017 | Taxable fee measure |
| public.fact_transaction | one row per transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Backend | LINK_PUB_AMT_018 | Upstream charity amount |
| public.fact_transaction | one row per transaction_guid | true_gross_transaction_amount | numeric(19,4) | Gross amount before charity adjustments | 43.50 | Derived | LINK_PUB_AMT_019 | Derived gross metric |
| public.fact_transaction | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | Purchase: 0.00 / Refund: -1.00 | Derived | LINK_PUB_AMT_020 | Return charity metric |
| public.fact_transaction | one row per transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | Purchase: 0.00 / Refund: -1.00 | Derived | LINK_PUB_AMT_021 | Return charity metric |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | gross_transaction_amount | numeric(19,4) | Gross transaction amount | 42.50 | Backend | LINK_AN_AMT_001 | `ft.gross_transaction_amount` |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee amount | 5.00 | Backend | LINK_AN_AMT_002 | `ft.sales_fee_amount` |
| analytics.transactions_data_source | one row per transaction_guid | waived_sales_fee_ticket_count | int | Waived sales fee ticket count | 0 | Derived | LINK_AN_AMT_003 | `ft.waived_sales_fee_ticket_count` |
| analytics.transactions_data_source | one row per transaction_guid | waived_fanclub_fee_ticket_count | int | Waived fanclub fee ticket count | 0 | Derived | LINK_AN_AMT_004 | `ft.waived_fanclub_fee_ticket_count` |
| analytics.transactions_data_source | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount | Purchase: 0.00 / Refund: -42.50 | Derived | LINK_AN_AMT_005 | `ft.returned_amount`; late update block refreshes |
| analytics.transactions_data_source | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | Purchase: 0.00 / Refund: -5.00 | Derived | LINK_AN_AMT_006 | `ft.returned_fee_amount`; late update block refreshes |
| analytics.transactions_data_source | one row per transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Backend | LINK_AN_AMT_007 | `ft.ticket_sales_amount` |
| analytics.transactions_data_source | one row per transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Backend | LINK_AN_AMT_008 | `ft.waived_sales_fee_amount` |
| analytics.transactions_data_source | one row per transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Backend | LINK_AN_AMT_009 | `ft.waived_fanclub_fee_amount` |
| analytics.transactions_data_source | one row per transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee amount | 1.50 | Backend | LINK_AN_AMT_010 | `ft.surcharge_fee_amount` |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_return_amount | numeric(19,4) | Sales fee return amount | Purchase: 0.00 / Refund: -5.00 | Derived | LINK_AN_AMT_011 | `ft.sales_fee_return_amount` |
| analytics.transactions_data_source | one row per transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Surcharge fee return amount | Purchase: 0.00 / Refund: -1.50 | Derived | LINK_AN_AMT_012 | `ft.surcharge_fee_return_amount` |
| analytics.transactions_data_source | one row per transaction_guid | reservation_fee_return_amount | numeric(19,4) | Reservation fee return amount | Purchase: 0.00 / Refund: -0.75 | Derived | LINK_AN_AMT_013 | `ft.reservation_fee_return_amount` |
| analytics.transactions_data_source | one row per transaction_guid | tax_return_amount | numeric(19,4) | Tax return amount | Purchase: 0.00 / Refund: -1.75 | Derived | LINK_AN_AMT_014 | `ft.tax_return_amount` |
| analytics.transactions_data_source | one row per transaction_guid | card_fee_return_amount | numeric(19,4) | Card fee return amount | Purchase: 0.00 / Refund: -0.25 | Derived | LINK_AN_AMT_015 | `ft.card_fee_return_amount` |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee for returned purchase | 5.00 | Derived | LINK_AN_AMT_016 | `ft.sales_fee_amount_for_returned_purchase` |
| analytics.transactions_data_source | one row per transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived | LINK_AN_AMT_017 | `ft.taxable_sales_fee_amount` |
| analytics.transactions_data_source | one row per transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Backend | LINK_AN_AMT_018 | `ft.charity_amount` |
| analytics.transactions_data_source | one row per transaction_guid | true_gross_transaction_amount | numeric(19,4) | True gross amount | 43.50 | Derived | LINK_AN_AMT_019 | `ft.true_gross_transaction_amount` |
| analytics.transactions_data_source | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | Purchase: 0.00 / Refund: -1.00 | Derived | LINK_AN_AMT_020 | `ft.returned_charity_amount`; late update block refreshes |
| analytics.transactions_data_source | one row per transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | Purchase: 0.00 / Refund: -1.00 | Derived | LINK_AN_AMT_021 | `ft.charity_return_amount` |

---

## Group 3: Movie

### Flow

```text
tblTransaction_PerformanceAttribute
→ raw_import_cache.transactionPerformanceAttribute
→ warehouse_loader.fact_transaction_base.movie_key
→ public.dim_movie
→ analytics.transactions_data_source
```

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transactionPerformanceAttribute | one row per transaction-performance | Movie_detail_id | int | Raw movie detail ID | 10293 | Backend | LINK_RAW_MOV_001 | Extracted movie id |
| raw_import_cache.transactionPerformanceAttribute | one row per transaction-performance | Perf_id | bigint | Performance id | 998877 | Backend | LINK_RAW_MOV_002 | Used for showtime mapping |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | movie_key | bigint | Surrogate movie key | 129384 | ETL | LINK_STG_MOV_001 | Join `tpa.movie_detail_id = dim_movie.movie_id` |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_movie | one row per movie_key | movie_id | int | Child movie ID | 10293 | Backend | LINK_PUB_MOV_001 | Movie id |
| public.dim_movie | one row per movie_key | movie_title | varchar | Child movie title | The Example Movie | Backend | LINK_PUB_MOV_002 | Movie title |
| public.dim_movie | one row per movie_key | movie_parent_id | int | Parent movie ID | 872 | Derived | LINK_PUB_MOV_003 | Parent relationship |
| public.dim_movie | one row per movie_key | movie_parent_title | varchar | Parent movie title | Example Franchise | Derived | LINK_PUB_MOV_004 | Parent relationship |
| public.dim_movie | one row per movie_key | movie_release_date | date | Movie release date | 2026-06-14 | Backend | LINK_PUB_MOV_005 | Release date |
| public.dim_movie | one row per movie_key | movie_tms_id | varchar | TMS id | MV123456789 | Third-party | LINK_PUB_MOV_006 | External content id |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | filmid_child | int | Child movie id | 10293 | Backend | LINK_AN_MOV_001 | `film.movie_id` |
| analytics.transactions_data_source | one row per transaction_guid | filmname_child | varchar | Child movie title | The Example Movie | Backend | LINK_AN_MOV_002 | `film.movie_title` |
| analytics.transactions_data_source | one row per transaction_guid | filmid_parent | int | Parent movie id | 872 | Derived | LINK_AN_MOV_003 | `film.movie_parent_id` |
| analytics.transactions_data_source | one row per transaction_guid | filmname_parent | varchar | Parent movie title | Example Franchise | Derived | LINK_AN_MOV_004 | `film.movie_parent_title` |
| analytics.transactions_data_source | one row per transaction_guid | releasedate | date | Release date | 2026-06-14 | Backend | LINK_AN_MOV_005 | `film.movie_release_date` |
| analytics.transactions_data_source | one row per transaction_guid | movie_tms_id | varchar | TMS id | MV123456789 | Third-party | LINK_AN_MOV_006 | `film.movie_tms_id` |

---

## Group 4: Customer

### Flow

```text
tblTransaction + tblTransactionCommitted + tblTransactionCustomer + transactionAttribute + transactionPurchaseFunnel
→ raw_import_cache.transaction / transactionCommitted / transactionCustomer / transactionAttribute / transactionPurchaseFunnel
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction + dim_user_agent + dim_purchase_funnel
→ analytics.transactions_data_source
```

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | UserAgent_nm | varchar | Raw user agent id/value | Mozilla/... | Backend | LINK_RAW_CUS_001 | Feeds user_agent dim lookup |
| raw_import_cache.transactionPurchaseFunnel | one row per tx funnel event | Funnel_cd | varchar | Raw purchase funnel code | 14 | Backend | LINK_RAW_CUS_002 | Feeds purchase_funnel dim lookup |
| raw_import_cache.transactionCommitted | one row per committed transaction | Loyalty_cd | varchar | Raw loyalty code | LOY12345 | Backend | LINK_RAW_CUS_003 | Source loyalty code |
| raw_import_cache.transaction | one row per transaction_guid | DeviceType_cd | varchar | Raw device type | Mobile | Backend | LINK_RAW_CUS_004 | Device source |
| raw_import_cache.transactionAttribute | one row per tx attribute | attribute/value rows | varchar | Fanclub/buynow/networktoken etc attributes | true | Backend | LINK_RAW_CUS_005 | Attribute-driven fields |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | user_agent_key | bigint | User agent key | 298771 | ETL | LINK_STG_CUS_001 | Join `dua.user_agent_id = t.UserAgent_nm` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | purchase_funnel_key | bigint | Purchase funnel key | 7231 | ETL | LINK_STG_CUS_002 | Join `tpf.Funnel_cd = dpf.purchase_funnel_id` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY12345 | Backend | LINK_STG_CUS_003 | `tc.Loyalty_cd` trimmed |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_STG_CUS_004 | `t.DeviceType_cd` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived | LINK_STG_CUS_005 | CASE over FanClubStatus attribute |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_user_agent | one row per user_agent_key | user_agent_id | varchar | User agent id | UA_44321 | Backend | LINK_PUB_CUS_001 | Conformed id |
| public.dim_user_agent | one row per user_agent_key | user_agent_description | varchar | User agent description | iOS Safari | Derived | LINK_PUB_CUS_002 | Friendly descriptor |
| public.dim_purchase_funnel | one row per purchase_funnel_key | purchase_funnel_id | varchar | Funnel id | 14 | Backend | LINK_PUB_CUS_003 | Conformed id |
| public.dim_purchase_funnel | one row per purchase_funnel_key | purchase_funnel_name | varchar | Funnel name | Guest Checkout | Derived | LINK_PUB_CUS_004 | Friendly name |
| public.fact_transaction | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY12345 | Backend | LINK_PUB_CUS_005 | Pass-through |
| public.fact_transaction | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_PUB_CUS_006 | Pass-through |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | user_agent_id | varchar | User agent id | UA_44321 | Backend | LINK_AN_CUS_001 | `dua.user_agent_id` |
| analytics.transactions_data_source | one row per transaction_guid | user_agent_description | varchar | User agent description | iOS Safari | Derived | LINK_AN_CUS_002 | `dua.user_agent_description` |
| analytics.transactions_data_source | one row per transaction_guid | purchase_funnel_id | varchar | Funnel id | 14 | Backend | LINK_AN_CUS_003 | `dpf.purchase_funnel_id` |
| analytics.transactions_data_source | one row per transaction_guid | purchase_funnel_name | varchar | Funnel name | Guest Checkout | Derived | LINK_AN_CUS_004 | `dpf.purchase_funnel_name` |
| analytics.transactions_data_source | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY12345 | Backend | LINK_AN_CUS_005 | `ft.loyalty_code` |
| analytics.transactions_data_source | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_AN_CUS_006 | `ft.deviceType` |
| analytics.transactions_data_source | one row per transaction_guid | apps | varchar | App grouping | Fandango App: iPhone | Derived | LINK_AN_CUS_007 | CASE on affiliate ids |
| analytics.transactions_data_source | one row per transaction_guid | platform | varchar | Platform grouping | Mobile Web | Derived | LINK_AN_CUS_008 | CASE on affiliate + channel |

---

## Group 5: Location

### Flow

```text
tblTransaction_PerformanceAttribute + theater dimensions + DMA mapping
→ raw_import_cache.transactionPerformanceAttribute + raw_import_cache.designatedMarketArea
→ warehouse_loader.fact_transaction_base.theater_key/chain_key
→ public.dim_theater + public.dim_chain
→ analytics.transactions_data_source
```

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transactionPerformanceAttribute | one row per transaction-performance | Theater_pkey | int | Source theater id/key | 4451 | Backend | LINK_RAW_LOC_001 | Theater source key |
| raw_import_cache.designatedMarketArea | one row per zip | zip_code | varchar | ZIP code for DMA mapping | 90028 | Third-party | LINK_RAW_LOC_002 | External DMA mapping source |
| raw_import_cache.designatedMarketArea | one row per zip | market_name | varchar | DMA market name | Los Angeles | Third-party | LINK_RAW_LOC_003 | External market name |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | theater_key | bigint | Theater dimension key | 11872 | ETL | LINK_STG_LOC_001 | Join from `tpa.Theater_pkey` to dim_theater |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | chain_key | bigint | Chain dimension key | 392 | ETL | LINK_STG_LOC_002 | Derived via dim_theater relationship |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_theater | one row per theater_key | theater_id | int | Theater id | 4451 | Backend | LINK_PUB_LOC_001 | Theater identifier |
| public.dim_theater | one row per theater_key | theater_code | varchar | Theater code | TH4451 | Derived | LINK_PUB_LOC_002 | Business code |
| public.dim_theater | one row per theater_key | theater_name | varchar | Theater name | Example Cinemas Hollywood | Derived | LINK_PUB_LOC_003 | Display name |
| public.dim_theater | one row per theater_key | pdi_id | varchar | PDI id | PDI9981 | Backend | LINK_PUB_LOC_004 | Theater system id |
| public.dim_theater | one row per theater_key | point_of_sale_theater_id | varchar | POS theater id | POS_TH_4451 | Backend | LINK_PUB_LOC_005 | POS-specific theater id |
| public.dim_theater | one row per theater_key | latitude | numeric | Latitude | 34.1021 | Backend | LINK_PUB_LOC_006 | Coordinate |
| public.dim_theater | one row per theater_key | longitude | numeric | Longitude | -118.3402 | Backend | LINK_PUB_LOC_007 | Coordinate |
| public.dim_theater | one row per theater_key | point_of_sale_id | varchar | POS id | POS01 | Backend | LINK_PUB_LOC_008 | POS identifier |
| public.dim_theater | one row per theater_key | address_1 | varchar | Address line 1 | 1234 Example Blvd | Backend | LINK_PUB_LOC_009 | Address |
| public.dim_theater | one row per theater_key | address_2 | varchar | Address line 2 | Suite 200 | Backend | LINK_PUB_LOC_010 | Address |
| public.dim_theater | one row per theater_key | city_name | varchar | City | Los Angeles | Backend | LINK_PUB_LOC_011 | Address |
| public.dim_theater | one row per theater_key | state_code | varchar | State | CA | Backend | LINK_PUB_LOC_012 | Address |
| public.dim_theater | one row per theater_key | zip_code | varchar | ZIP code | 90028 | Backend | LINK_PUB_LOC_013 | Address |
| public.dim_theater | one row per theater_key | country_code | varchar | Country | US | Backend | LINK_PUB_LOC_014 | Address |
| public.dim_theater | one row per theater_key | alternate_theater_phone_name | varchar | Alternate theater phone/name | Main Box Office | Derived | LINK_PUB_LOC_015 | Theater metadata |
| public.dim_theater | one row per theater_key | chain_specific_site_id | varchar | Chain site id | SITE_123 | Backend | LINK_PUB_LOC_016 | Theater metadata |
| public.dim_theater | one row per theater_key | paymentech_terminal_id | varchar | Paymentech terminal id | PT_123 | Third-party | LINK_PUB_LOC_017 | Payment terminal metadata |
| public.dim_theater | one row per theater_key | cybersource_terminal_id | varchar | Cybersource terminal id | CS_456 | Third-party | LINK_PUB_LOC_018 | Payment terminal metadata |
| public.dim_theater | one row per theater_key | screen_count | int | Number of screens | 12 | Backend | LINK_PUB_LOC_019 | Theater attribute |
| public.dim_theater | one row per theater_key | seat_count | int | Number of seats | 1800 | Backend | LINK_PUB_LOC_020 | Theater attribute |
| public.dim_theater | one row per theater_key | theater_time_zone_name | varchar | Theater timezone | America/Los_Angeles | Backend | LINK_PUB_LOC_021 | Theater timezone |
| public.dim_theater | one row per theater_key | braintree_merchant_id | varchar | Braintree merchant id | bt_merchant_123 | Third-party | LINK_PUB_LOC_022 | Payment processor mapping |
| public.dim_chain | one row per chain_key | chain_name | varchar | Chain name | Example Cinemas | Derived | LINK_PUB_LOC_023 | Chain descriptor |
| public.dim_chain | one row per chain_key | chain_id | varchar | Chain id | CHN019 | Backend | LINK_PUB_LOC_024 | Chain identifier |
| public.dim_chain | one row per chain_key | point_of_sale_vendor | varchar | POS vendor | Vista | Third-party | LINK_PUB_LOC_025 | Vendor source |
| public.dim_chain | one row per chain_key | awards_program_name | varchar | Awards program name | Example Rewards | Derived | LINK_PUB_LOC_026 | Program label |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | chainname | varchar | Chain name | Example Cinemas | Derived | LINK_AN_LOC_001 | `chain.chain_name` |
| analytics.transactions_data_source | one row per transaction_guid | chainid | varchar | Chain id | CHN019 | Backend | LINK_AN_LOC_002 | `chain.chain_id` |
| analytics.transactions_data_source | one row per transaction_guid | pos_vendor | varchar | POS vendor | Vista | Third-party | LINK_AN_LOC_003 | `chain.point_of_sale_vendor` |
| analytics.transactions_data_source | one row per transaction_guid | awards_program_name | varchar | Awards program name | Example Rewards | Derived | LINK_AN_LOC_004 | `chain.awards_program_name` |
| analytics.transactions_data_source | one row per transaction_guid | theater_id | int | Theater id | 4451 | Backend | LINK_AN_LOC_005 | `tht.theater_id` |
| analytics.transactions_data_source | one row per transaction_guid | theater_code | varchar | Theater code | TH4451 | Derived | LINK_AN_LOC_006 | `tht.theater_code` |
| analytics.transactions_data_source | one row per transaction_guid | theater_name | varchar | Theater name | Example Cinemas Hollywood | Derived | LINK_AN_LOC_007 | `tht.theater_name` |
| analytics.transactions_data_source | one row per transaction_guid | pdi_id | varchar | PDI id | PDI9981 | Backend | LINK_AN_LOC_008 | `tht.pdi_id` |
| analytics.transactions_data_source | one row per transaction_guid | pos_theater_id | varchar | POS theater id | POS_TH_4451 | Backend | LINK_AN_LOC_009 | `tht.point_of_sale_theater_id` |
| analytics.transactions_data_source | one row per transaction_guid | latitude | numeric | Latitude | 34.1021 | Backend | LINK_AN_LOC_010 | `tht.latitude` |
| analytics.transactions_data_source | one row per transaction_guid | longitude | numeric | Longitude | -118.3402 | Backend | LINK_AN_LOC_011 | `tht.longitude` |
| analytics.transactions_data_source | one row per transaction_guid | point_of_sale_id | varchar | POS id | POS01 | Backend | LINK_AN_LOC_012 | `tht.point_of_sale_id` |
| analytics.transactions_data_source | one row per transaction_guid | address_1 | varchar | Address line 1 | 1234 Example Blvd | Backend | LINK_AN_LOC_013 | `tht.address_1` |
| analytics.transactions_data_source | one row per transaction_guid | address_2 | varchar | Address line 2 | Suite 200 | Backend | LINK_AN_LOC_014 | `tht.address_2` |
| analytics.transactions_data_source | one row per transaction_guid | city_name | varchar | City | Los Angeles | Backend | LINK_AN_LOC_015 | `tht.city_name` |
| analytics.transactions_data_source | one row per transaction_guid | state_code | varchar | State code | CA | Backend | LINK_AN_LOC_016 | `tht.state_code` |
| analytics.transactions_data_source | one row per transaction_guid | zip_code | varchar | ZIP code | 90028 | Backend | LINK_AN_LOC_017 | `tht.zip_code` |
| analytics.transactions_data_source | one row per transaction_guid | country_code | varchar | Country code | US | Backend | LINK_AN_LOC_018 | `tht.country_code` |
| analytics.transactions_data_source | one row per transaction_guid | tht_alternate_theater_phone_name | varchar | Alternate theater phone/name | Main Box Office | Derived | LINK_AN_LOC_019 | `tht.alternate_theater_phone_name` |
| analytics.transactions_data_source | one row per transaction_guid | chain_specific_site_id | varchar | Chain specific site id | SITE_123 | Backend | LINK_AN_LOC_020 | `tht.chain_specific_site_id` |
| analytics.transactions_data_source | one row per transaction_guid | paymentech_terminal_id | varchar | Paymentech terminal id | PT_123 | Third-party | LINK_AN_LOC_021 | `tht.paymentech_terminal_id` |
| analytics.transactions_data_source | one row per transaction_guid | cybersource_terminal_id | varchar | Cybersource terminal id | CS_456 | Third-party | LINK_AN_LOC_022 | `tht.cybersource_terminal_id` |
| analytics.transactions_data_source | one row per transaction_guid | screen_count | int | Screen count | 12 | Backend | LINK_AN_LOC_023 | `tht.screen_count` |
| analytics.transactions_data_source | one row per transaction_guid | seat_count | int | Seat count | 1800 | Backend | LINK_AN_LOC_024 | `tht.seat_count` |
| analytics.transactions_data_source | one row per transaction_guid | theater_time_zone_name | varchar | Theater timezone | America/Los_Angeles | Backend | LINK_AN_LOC_025 | `tht.theater_time_zone_name` |
| analytics.transactions_data_source | one row per transaction_guid | dma_market_name | varchar | DMA market name | Los Angeles | Third-party | LINK_AN_LOC_026 | `dma.market_name` by zip join |
| analytics.transactions_data_source | one row per transaction_guid | braintree_merchant_id | varchar | Braintree merchant id | bt_merchant_123 | Third-party | LINK_AN_LOC_027 | `tht.braintree_merchant_id` |

---

## Group 6: Attribution

### Flow

```text
tblTransaction + channel/affiliate + partner attrs
→ raw_import_cache.transaction + transactionAttribute
→ warehouse_loader.fact_transaction_base
→ public.dim_affiliate + public.dim_channel + partner dims
→ analytics.transactions_data_source
```

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Affiliate_id | int | Raw affiliate id | 132 | Backend | LINK_RAW_ATT_001 | Affiliate source id |
| raw_import_cache.transaction | one row per transaction_guid | Channel_cd | varchar | Raw channel code | WAP | Backend | LINK_RAW_ATT_002 | Channel source code |
| raw_import_cache.transactionAttribute | one row per tx attribute | partner-channel / partner-platform | varchar | Partner attrs | TMO / WEBVIEW | Third-party | LINK_RAW_ATT_003 | Partner attributes in transactionAttribute |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | affiliate_key | bigint | Affiliate key | 8123 | ETL | LINK_STG_ATT_001 | `da.affiliate_id = t.Affiliate_id` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | channel_key | bigint | Channel key | 291 | ETL | LINK_STG_ATT_002 | `dc.channel_id = t.Channel_cd` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_channel_key | bigint | Partner channel key | 912 | ETL | LINK_STG_ATT_003 | Lookup from partner-channel attribute |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_platform_key | bigint | Partner platform key | 117 | ETL | LINK_STG_ATT_004 | Lookup from partner-platform attribute |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_affiliate | one row per affiliate_key | affiliate_name | varchar | Affiliate name | Fandango iOS App | Derived | LINK_PUB_ATT_001 | Friendly name |
| public.dim_affiliate | one row per affiliate_key | affiliate_group_name | varchar | Affiliate group | Mobile App | Derived | LINK_PUB_ATT_002 | Grouping |
| public.dim_affiliate | one row per affiliate_key | affiliate_category_name | varchar | Affiliate category | Owned & Operated | Derived | LINK_PUB_ATT_003 | Category |
| public.dim_affiliate | one row per affiliate_key | affiliate_id | int | Affiliate id | 132 | Backend | LINK_PUB_ATT_004 | Id |
| public.dim_channel | one row per channel_key | channel_group | varchar | Channel group | App | Derived | LINK_PUB_ATT_005 | Group label |
| public.dim_channel | one row per channel_key | channel_name | varchar | Channel name | WAP | Backend | LINK_PUB_ATT_006 | Channel code/name |
| public.dim_partner_channel | one row per partner_channel_key | partner_channel_id | varchar | Partner channel id | TMO | Third-party | LINK_PUB_ATT_007 | Partner channel code |
| public.dim_partner_channel | one row per partner_channel_key | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party | LINK_PUB_ATT_008 | Partner display |
| public.dim_partner_platform | one row per partner_platform_key | partner_platform_id | varchar | Partner platform id | WEBVIEW | Third-party | LINK_PUB_ATT_009 | Partner platform code |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | affiliate_name | varchar | Affiliate name | Fandango iOS App | Derived | LINK_AN_ATT_001 | `aff.affiliate_name` |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_group_name | varchar | Affiliate group | Mobile App | Derived | LINK_AN_ATT_002 | `aff.affiliate_group_name` |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_category_name | varchar | Affiliate category | Owned & Operated | Derived | LINK_AN_ATT_003 | `aff.affiliate_category_name` |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_id | int | Affiliate id | 132 | Backend | LINK_AN_ATT_004 | `aff.affiliate_id` |
| analytics.transactions_data_source | one row per transaction_guid | channel | varchar | Channel group | App | Derived | LINK_AN_ATT_005 | `channel.channel_group` |
| analytics.transactions_data_source | one row per transaction_guid | channel_ungrouped | varchar | Channel detail | WAP | Backend | LINK_AN_ATT_006 | `channel.channel_name` |
| analytics.transactions_data_source | one row per transaction_guid | Affiliate Grouping | varchar | Affiliate grouping label | iPhone App | Derived | LINK_AN_ATT_007 | Large CASE on affiliate_id |
| analytics.transactions_data_source | one row per transaction_guid | Affiliate Grouping - HIGH LEVEL BINS | varchar | High level bins | Fandango O&O | Derived | LINK_AN_ATT_008 | CASE on affiliate_id |
| analytics.transactions_data_source | one row per transaction_guid | partner_channel | varchar | Partner channel id | TMO | Third-party | LINK_AN_ATT_009 | `dpc.partner_channel_id` |
| analytics.transactions_data_source | one row per transaction_guid | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party | LINK_AN_ATT_010 | `dpc.partner_display_name` |
| analytics.transactions_data_source | one row per transaction_guid | partner_platform | varchar | Partner platform id | WEBVIEW | Third-party | LINK_AN_ATT_011 | `dpp.partner_platform_id` |
| analytics.transactions_data_source | one row per transaction_guid | partner_defined_data_value | varchar | Partner-defined payload | campaign=fall_promo | Backend | LINK_AN_ATT_012 | `ft.partner_defined_data_value` |

---

## Group 7: Types

### Flow

```text
tblTransaction / committed indicators / property source
→ raw_import_cache.transaction / transactionCommitted
→ warehouse_loader.fact_transaction_base
→ public.dim_transaction_type + dim_return_type + dim_property
→ analytics.transactions_data_source
```

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Transaction_tp | varchar | Transaction type id/code | Purchase | Backend | LINK_RAW_TYP_001 | Source transaction type |
| raw_import_cache.transaction | one row per transaction_guid | Property_nm | varchar | Property name | Fandango | Backend | LINK_RAW_TYP_002 | Source property |
| raw_import_cache.transactionCommitted | one row per committed tx | Confirmation_cd | varchar | Confirmation code prefix | REF123 | Backend | LINK_RAW_TYP_003 | Used for return type mapping |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_type_key | bigint | Transaction type key | 82 | ETL | LINK_STG_TYP_001 | Join `dtt.transaction_type_id = t.Transaction_tp` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | return_type_key | bigint | Return type key | 16 | ETL | LINK_STG_TYP_002 | Join by confirmation code prefix |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | property_key | bigint | Property key | 45 | ETL | LINK_STG_TYP_003 | Join from normalized property name |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_transaction_type | one row per transaction_type_key | transaction_type_id | varchar | Transaction type id | 2 | Backend | LINK_PUB_TYP_001 | Conformed id |
| public.dim_transaction_type | one row per transaction_type_key | transaction_type_name | varchar | Transaction type name | Purchase | Derived | LINK_PUB_TYP_002 | Label |
| public.dim_return_type | one row per return_type_key | return_method | varchar | Return method | Self-service | Derived | LINK_PUB_TYP_003 | Label |
| public.dim_return_type | one row per return_type_key | return_type_code | varchar | Return code | REF | Backend | LINK_PUB_TYP_004 | Code |
| public.dim_return_type | one row per return_type_key | return_type_description | varchar | Return description | Customer cancellation | Derived | LINK_PUB_TYP_005 | Description |
| public.dim_property | one row per property_key | property_id | varchar | Property id | Fandango USA | Backend | LINK_PUB_TYP_006 | Id |
| public.dim_property | one row per property_key | property_group | varchar | Property group | Fandango | Derived | LINK_PUB_TYP_007 | Group label |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_type_id | varchar | Transaction type id | 2 | Backend | LINK_AN_TYP_001 | `tranxtype.transaction_type_id` |
| analytics.transactions_data_source | one row per transaction_guid | transaction_type_name | varchar | Transaction type name | Purchase | Derived | LINK_AN_TYP_002 | `tranxtype.transaction_type_name` |
| analytics.transactions_data_source | one row per transaction_guid | return_method | varchar | Return method | Self-service | Derived | LINK_AN_TYP_003 | `drt.return_method` |
| analytics.transactions_data_source | one row per transaction_guid | return_type_code | varchar | Return type code | REF | Backend | LINK_AN_TYP_004 | `drt.return_type_code` |
| analytics.transactions_data_source | one row per transaction_guid | return_type_description | varchar | Return type description | Customer cancellation | Derived | LINK_AN_TYP_005 | `drt.return_type_description` |
| analytics.transactions_data_source | one row per transaction_guid | property_id | varchar | Property id | Fandango USA | Backend | LINK_AN_TYP_006 | `dp.property_id` |
| analytics.transactions_data_source | one row per transaction_guid | property_group | varchar | Property group | Fandango | Derived | LINK_AN_TYP_007 | `dp.property_group` |

---

## Group 8: Flags

### Flow

```text
transaction + committed + transactionAttribute + amenity mapping
→ raw_import_cache.transaction / transactionCommitted / transactionAttribute / transactionPerformanceAttribute
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction
→ analytics.transactions_data_source
```

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Transaction_GUID | varchar | Transaction guid | ABC123 | Backend | LINK_RAW_FLG_001 | Primary key |
| raw_import_cache.transaction | one row per transaction_guid | Test_bt | boolean | Raw test indicator | false | Backend | LINK_RAW_FLG_002 | Input to test logic |
| raw_import_cache.transactionCommitted | one row per committed transaction | Confirmation_cd | varchar | POS confirmation code | CNF908812 | Backend | LINK_RAW_FLG_003 | Source POS confirmation |
| raw_import_cache.transactionCommitted | one row per committed transaction | ReservedSeats_tx | varchar | Reserved seats text | A10,A11 | Backend | LINK_RAW_FLG_004 | Input to reserved seating fields |
| raw_import_cache.transactionAttribute | one row per tx attribute | IsBuyNowOrder / NetworkToken / CharityName / PartnerDefinedData / FanClubStatus | varchar | Attribute flags/values | true | Backend | LINK_RAW_FLG_005 | Attribute sources for multiple flags |
| raw_import_cache.transactionPerformanceAttribute | one row per tx-performance | Amenity_ids | varchar | Comma-delimited amenity ids | 1,3,7 | Backend | LINK_RAW_FLG_006 | Input for amenity mapping |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_guid | varchar | Transaction guid | ABC123 | Backend | LINK_STG_FLG_001 | Direct pass-through |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_successful | boolean | Success flag | true | Derived | LINK_STG_FLG_002 | `tc exists and tr is null` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_STG_FLG_003 | `tc.Confirmation_cd` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived | LINK_STG_FLG_004 | CASE on reserved seats text non-empty |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | reserved_seats | varchar | Reserved seats | A10,A11 | Backend | LINK_STG_FLG_005 | Trimmed reserved seats |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | POS_reference_cd | varchar | POS reference code | POSREF123 | Backend | LINK_STG_FLG_006 | `tc.POS_reference_cd` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_test | boolean | Test transaction flag | false | Derived | LINK_STG_FLG_007 | Multi-condition CASE (test bit, emails, etc.) |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived | LINK_STG_FLG_008 | CASE on FanClubStatus attribute |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_buynow | boolean | Buy now flag | false | Derived | LINK_STG_FLG_009 | CASE on IsBuyNowOrder attribute |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_STG_FLG_010 | `ta3.value_tx` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_defined_data_value | varchar | Partner-defined data | campaign=fall_promo | Backend | LINK_STG_FLG_011 | `ta4.value_tx` |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | network_token_usage | boolean | Network token usage | true | Derived | LINK_STG_FLG_012 | CASE lower(trim(value)) = 'true' |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_plf | boolean | PLF flag | true | Derived | LINK_STG_FLG_013 | Derived from amenity mapping aggregate |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_imax | boolean | IMAX flag | false | Derived | LINK_STG_FLG_014 | Derived from amenity mapping aggregate |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_3d | boolean | 3D flag | false | Derived | LINK_STG_FLG_015 | Derived from amenity mapping aggregate |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | true | Derived | LINK_STG_FLG_016 | Derived from amenity mapping aggregate |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | transaction_guid | varchar | Transaction guid | ABC123 | Backend | LINK_PUB_FLG_001 | Pass-through |
| public.fact_transaction | one row per transaction_guid | is_successful | boolean | Success flag | true | Derived | LINK_PUB_FLG_002 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_PUB_FLG_003 | Pass-through |
| public.fact_transaction | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived | LINK_PUB_FLG_004 | Pass-through |
| public.fact_transaction | one row per transaction_guid | reserved_seats | varchar | Reserved seats | A10,A11 | Backend | LINK_PUB_FLG_005 | Pass-through |
| public.fact_transaction | one row per transaction_guid | payment_types | varchar | Payment types | VISA | Backend | LINK_PUB_FLG_006 | Payment method labels |
| public.fact_transaction | one row per transaction_guid | processor_order_ids | varchar | Processor order ids | ord_7891001 | Third-party | LINK_PUB_FLG_007 | External processor references |
| public.fact_transaction | one row per transaction_guid | POS_reference_cd | varchar | POS reference code | POSREF123 | Backend | LINK_PUB_FLG_008 | Pass-through |
| public.fact_transaction | one row per transaction_guid | is_test | boolean | Test flag | false | Derived | LINK_PUB_FLG_009 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived | LINK_PUB_FLG_010 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_PUB_FLG_011 | Pass-through |
| public.fact_transaction | one row per transaction_guid | partner_defined_data_value | varchar | Partner data value | campaign=fall_promo | Backend | LINK_PUB_FLG_012 | Pass-through |
| public.fact_transaction | one row per transaction_guid | is_buynow | boolean | Buy now flag | false | Derived | LINK_PUB_FLG_013 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | network_token_usage | boolean/varchar | Network token usage | true | Derived | LINK_PUB_FLG_014 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | is_plf | boolean | PLF flag | true | Derived | LINK_PUB_FLG_015 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | is_imax | boolean | IMAX flag | false | Derived | LINK_PUB_FLG_016 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | is_3d | boolean | 3D flag | false | Derived | LINK_PUB_FLG_017 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | true | Derived | LINK_PUB_FLG_018 | Pass-through derived value |
| public.fact_transaction | one row per transaction_guid | item_count | int | Item count | 2 | Derived | LINK_PUB_FLG_019 | Aggregated quantity |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_guid | varchar | Transaction guid | ABC123 | Backend | LINK_AN_FLG_001 | `ft.transaction_guid` |
| analytics.transactions_data_source | one row per transaction_guid | is_successful | boolean | Success flag | true | Derived | LINK_AN_FLG_002 | `ft.is_successful` |
| analytics.transactions_data_source | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_AN_FLG_003 | `ft.point_of_sale_confirmation_code` |
| analytics.transactions_data_source | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived | LINK_AN_FLG_004 | `ft.is_reserved_seating` |
| analytics.transactions_data_source | one row per transaction_guid | reserved_seats | varchar | Reserved seats | A10,A11 | Backend | LINK_AN_FLG_005 | `ft.reserved_seats` |
| analytics.transactions_data_source | one row per transaction_guid | payment_types | varchar | Payment type(s) | VISA | Backend | LINK_AN_FLG_006 | `ft.payment_types` |
| analytics.transactions_data_source | one row per transaction_guid | processor_order_ids | varchar | Processor order ids | ord_7891001 | Third-party | LINK_AN_FLG_007 | `ft.processor_order_ids` |
| analytics.transactions_data_source | one row per transaction_guid | item_qty | int | Item quantity | 2 | Derived | LINK_AN_FLG_008 | `ft.item_count AS item_qty` |
| analytics.transactions_data_source | one row per transaction_guid | POS_reference_cd | varchar | POS reference code | POSREF123 | Backend | LINK_AN_FLG_009 | `ft.POS_reference_cd` |
| analytics.transactions_data_source | one row per transaction_guid | is_test | boolean | Test flag | false | Derived | LINK_AN_FLG_010 | `ft.is_test` |
| analytics.transactions_data_source | one row per transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived | LINK_AN_FLG_011 | `ft.is_fanclub` |
| analytics.transactions_data_source | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_AN_FLG_012 | `ft.charity_name` |
| analytics.transactions_data_source | one row per transaction_guid | partner_defined_data_value | varchar | Partner-defined data | campaign=fall_promo | Backend | LINK_AN_FLG_013 | `ft.partner_defined_data_value` |
| analytics.transactions_data_source | one row per transaction_guid | is_buynow | boolean | Buy-now flag | false | Derived | LINK_AN_FLG_014 | `ft.is_buynow` |
| analytics.transactions_data_source | one row per transaction_guid | network_token_usage | boolean/varchar | Network token usage | true | Derived | LINK_AN_FLG_015 | `ft.network_token_usage` |
| analytics.transactions_data_source | one row per transaction_guid | is_plf | boolean | PLF flag | true | Derived | LINK_AN_FLG_016 | `ft.is_plf` |
| analytics.transactions_data_source | one row per transaction_guid | is_imax | boolean | IMAX flag | false | Derived | LINK_AN_FLG_017 | `ft.is_imax` |
| analytics.transactions_data_source | one row per transaction_guid | is_3d | boolean | 3D flag | false | Derived | LINK_AN_FLG_018 | `ft.is_3d` |
| analytics.transactions_data_source | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | true | Derived | LINK_AN_FLG_019 | `ft.is_higher_fee_amenity` |

---

## 96-Field Analytics Coverage Index

Below is the canonical list from `analyticsTransactionsDataSource.sql`, all covered above:

1. transaction_guid  
2. is_successful  
3. transaction_start_datetime_utc  
4. transaction_start_datetime_los_angeles  
5. transaction_commit_datetime_utc  
6. transaction_commit_datetime_los_angeles  
7. transaction_rollback_datetime_utc  
8. transaction_rollback_datetime_los_angeles  
9. transaction_end_datetime_utc  
10. transaction_end_datetime_los_angeles  
11. transaction_end_datetime_local  
12. deltareleasetopurchasedate  
13. deltareleasetoshowtimedate  
14. deltapurchasetoshowtimedate  
15. point_of_sale_confirmation_code  
16. is_reserved_seating  
17. reserved_seats  
18. loyalty_code  
19. performance_datetime_local  
20. payment_types  
21. processor_order_ids  
22. affiliate_name  
23. affiliate_group_name  
24. affiliate_category_name  
25. affiliate_id  
26. channel  
27. channel_ungrouped  
28. filmid_child  
29. filmname_child  
30. filmid_parent  
31. filmname_parent  
32. releasedate  
33. movie_tms_id  
34. chainname  
35. chainid  
36. pos_vendor  
37. awards_program_name  
38. theater_id  
39. theater_code  
40. theater_name  
41. pdi_id  
42. pos_theater_id  
43. latitude  
44. longitude  
45. point_of_sale_id  
46. address_1  
47. address_2  
48. city_name  
49. state_code  
50. zip_code  
51. country_code  
52. tht_alternate_theater_phone_name  
53. chain_specific_site_id  
54. paymentech_terminal_id  
55. cybersource_terminal_id  
56. screen_count  
57. seat_count  
58. theater_time_zone_name  
59. dma_market_name  
60. transaction_type_id  
61. transaction_type_name  
62. user_agent_id  
63. user_agent_description  
64. purchase_funnel_id  
65. purchase_funnel_name  
66. return_method  
67. return_type_code  
68. return_type_description  
69. Affiliate Grouping  
70. Affiliate Grouping - HIGH LEVEL BINS  
71. apps  
72. platform  
73. item_qty  
74. gross_transaction_amount  
75. sales_fee_amount  
76. waived_sales_fee_ticket_count  
77. waived_fanclub_fee_ticket_count  
78. returned_amount  
79. returned_fee_amount  
80. ticket_sales_amount  
81. waived_sales_fee_amount  
82. waived_fanclub_fee_amount  
83. surcharge_fee_amount  
84. sales_fee_return_amount  
85. surcharge_fee_return_amount  
86. reservation_fee_return_amount  
87. tax_return_amount  
88. card_fee_return_amount  
89. sales_fee_amount_for_returned_purchase  
90. property_id  
91. property_group  
92. POS_reference_cd  
93. braintree_merchant_id  
94. taxable_sales_fee_amount  
95. is_test  
96. deviceType  
97. is_fanclub  
98. charity_amount  
99. true_gross_transaction_amount  
100. returned_charity_amount  
101. charity_return_amount  
102. charity_name  
103. partner_defined_data_value  
104. is_buynow  
105. partner_channel  
106. partner_display_name  
107. partner_platform  
108. network_token_usage  
109. is_plf  
110. is_imax  
111. is_3d  
112. is_higher_fee_amenity  

> Note: The SQL insert list contains **112 columns** in the provided script. If your target is “96 fields,” that likely refers to an earlier table version. This document traces the currently provided SQL list in full.

---

## Link Placeholder Mapping

Use these conventions:

- `LINK_RAW_*` → raw extraction SQL in `dataeng_datawarehouse/.../extractTicketingTables/...`
- `LINK_STG_*` → `factTransactionBase.sql`
- `LINK_PUB_*` → public model build SQL
- `LINK_AN_*` → `analyticsTransactionsDataSource.sql`

Example permalink format:

```text
https://github.com/<owner>/<repo>/blob/<commit-or-branch>/<path>#L120-L145
```
