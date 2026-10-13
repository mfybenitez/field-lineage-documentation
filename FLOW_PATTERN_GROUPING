# analytics.transactions_data_source — Flow-Pattern Grouped Lineage

## Purpose

This document groups fields by **technical lineage pattern** (not business theme), so every field in each group follows the same flow behavior.

Primary analytics SQL source:
- `fandango/jaws/main/src/main/resources/queries/populateTableQueries/analyticsTransactionsDataSource.sql`

Staging backtrace source:
- `fandango/jaws/main/src/main/resources/queries/populateTableQueries/factTransactionBase.sql`

Raw extraction references (provided):
- `warehouse_raw_import_cache_transaction.sql`
- `warehouse_raw_import_cache_transactionCommitted.sql`
- `warehouse_raw_import_cache_transactionItem.sql`
- `warehouse_raw_import_cache_transactionPerformanceAttribute.sql`
- `warehouse_raw_import_cache_trackingDeviceStatus.sql` (not used by current analytics query fields shown)

---

## Group A — Direct Pass-Through from `public.fact_transaction`

### Pattern Definition

```text
raw_import_cache -> warehouse_loader.fact_transaction_base -> public.fact_transaction -> analytics.transactions_data_source
```

Analytics expression shape:
```sql
SELECT public.fact_transaction.<column> [AS <analytics_column>]
```
(no business transformation in analytics layer; rename-only allowed)

### Fields

| Analytics Column | Public Source Column | Source | Notes |
|---|---|---|---|
| transaction_guid | transaction_guid | Backend | direct pass-through |
| is_successful | is_successful | Derived | derived upstream in `factTransactionBase.sql`, pass-through in analytics |
| transaction_start_datetime_utc | transaction_start_datetime | Backend | alias only (`... AS transaction_start_datetime_utc`) |
| transaction_commit_datetime_utc | transaction_commit_datetime | Backend | alias only |
| transaction_rollback_datetime_utc | transaction_rollback_datetime | Backend | alias only |
| transaction_end_datetime_utc | transaction_end_datetime | Derived | derived upstream CASE logic in staging |
| transaction_end_datetime_local | transaction_end_datetime_local | Derived | pass-through |
| point_of_sale_confirmation_code | point_of_sale_confirmation_code | Backend | pass-through |
| is_reserved_seating | is_reserved_seating | Derived | upstream CASE in staging |
| reserved_seats | reserved_seats | Backend | pass-through |
| loyalty_code | loyalty_code | Backend | pass-through |
| performance_datetime_local | performance_datetime_local | Backend | pass-through |
| payment_types | payment_types | Backend | pass-through |
| processor_order_ids | processor_order_ids | Third-party | pass-through |
| item_qty | item_count | Derived | rename-only in analytics (`item_count AS item_qty`) |
| gross_transaction_amount | gross_transaction_amount | Backend | pass-through |
| sales_fee_amount | sales_fee_amount | Backend | pass-through |
| waived_sales_fee_ticket_count | waived_sales_fee_ticket_count | Derived | pass-through |
| waived_fanclub_fee_ticket_count | waived_fanclub_fee_ticket_count | Derived | pass-through |
| returned_amount | returned_amount | Derived | pass-through + see Group E |
| returned_fee_amount | returned_fee_amount | Derived | pass-through + see Group E |
| ticket_sales_amount | ticket_sales_amount | Backend | pass-through |
| waived_sales_fee_amount | waived_sales_fee_amount | Backend | pass-through |
| waived_fanclub_fee_amount | waived_fanclub_fee_amount | Backend | pass-through |
| surcharge_fee_amount | surcharge_fee_amount | Backend | pass-through |
| sales_fee_return_amount | sales_fee_return_amount | Derived | pass-through |
| surcharge_fee_return_amount | surcharge_fee_return_amount | Derived | pass-through |
| reservation_fee_return_amount | reservation_fee_return_amount | Derived | pass-through |
| tax_return_amount | tax_return_amount | Derived | pass-through |
| card_fee_return_amount | card_fee_return_amount | Derived | pass-through |
| sales_fee_amount_for_returned_purchase | sales_fee_amount_for_returned_purchase | Derived | pass-through |
| POS_reference_cd | POS_reference_cd | Backend | pass-through |
| taxable_sales_fee_amount | taxable_sales_fee_amount | Derived | pass-through |
| is_test | is_test | Derived | upstream multi-condition CASE in staging |
| deviceType | deviceType | Backend | pass-through |
| is_fanclub | is_fanclub | Derived | upstream CASE in staging |
| charity_amount | charity_amount | Backend | pass-through |
| true_gross_transaction_amount | true_gross_transaction_amount | Derived | pass-through |
| returned_charity_amount | returned_charity_amount | Derived | pass-through + see Group E |
| charity_return_amount | charity_return_amount | Derived | pass-through |
| charity_name | charity_name | Backend | pass-through |
| partner_defined_data_value | partner_defined_data_value | Backend | pass-through |
| is_buynow | is_buynow | Derived | upstream CASE in staging |
| network_token_usage | network_token_usage | Derived | upstream CASE in staging |
| is_plf | is_plf | Derived | upstream amenity rollup logic |
| is_imax | is_imax | Derived | upstream amenity rollup logic |
| is_3d | is_3d | Derived | upstream amenity rollup logic |
| is_higher_fee_amenity | is_higher_fee_amenity | Derived | upstream amenity rollup logic |

---

## Group B — Derived in Analytics from `public.fact_transaction` (Date Math / Timezone)

### Pattern Definition

```text
public.fact_transaction -> analytics expression (CONVERT_TIMEZONE / DATEDIFF / DATE_TRUNC)
```

Analytics expression shape:
```sql
CONVERT_TIMEZONE(...)
DATEDIFF(...)
DATE_TRUNC(...)
```

### Fields

| Analytics Column | Derivation Inputs | Source | Notes |
|---|---|---|---|
| transaction_start_datetime_los_angeles | `public.fact_transaction.transaction_start_datetime` | Derived | timezone conversion in analytics |
| transaction_commit_datetime_los_angeles | `public.fact_transaction.transaction_commit_datetime` | Derived | timezone conversion |
| transaction_rollback_datetime_los_angeles | `public.fact_transaction.transaction_rollback_datetime` | Derived | timezone conversion |
| transaction_end_datetime_los_angeles | `public.fact_transaction.transaction_end_datetime` | Derived | timezone conversion |
| deltareleasetopurchasedate | `public.dim_movie.movie_release_date`, converted purchase day from `public.fact_transaction.transaction_end_datetime` | Derived | DATEDIFF in analytics |
| deltareleasetoshowtimedate | `public.dim_movie.movie_release_date`, `public.fact_transaction.performance_datetime_local` | Derived | DATEDIFF in analytics |
| deltapurchasetoshowtimedate | converted purchase day from `public.fact_transaction.transaction_end_datetime`, `public.fact_transaction.performance_datetime_local` | Derived | DATEDIFF in analytics |
| apps | affiliate-based CASE logic | Derived | CASE in analytics query |
| platform | affiliate/channel CASE logic | Derived | CASE in analytics query |
| Affiliate Grouping | affiliate-based CASE logic | Derived | large CASE in analytics |
| Affiliate Grouping - HIGH LEVEL BINS | affiliate-based CASE logic | Derived | CASE in analytics |

---

## Group C — Dimension-Sourced via Keys in `public.fact_transaction`

### Pattern Definition

```text
raw source ids -> staging keys in warehouse_loader.fact_transaction_base
-> public.fact_transaction.<dim_key>
-> LEFT JOIN public.dim_*
-> analytics.transactions_data_source
```

### Fields

| Analytics Column | Joined Dimension Column | Source | Notes |
|---|---|---|---|
| affiliate_name | `public.dim_affiliate.affiliate_name` | Derived | descriptive label |
| affiliate_group_name | `public.dim_affiliate.affiliate_group_name` | Derived | descriptive label |
| affiliate_category_name | `public.dim_affiliate.affiliate_category_name` | Derived | descriptive label |
| affiliate_id | `public.dim_affiliate.affiliate_id` | Backend | source affiliate id conformed in dim |
| channel | `public.dim_channel.channel_group` | Derived | grouped channel |
| channel_ungrouped | `public.dim_channel.channel_name` | Backend | source channel-conformed |
| filmid_child | `public.dim_movie.movie_id` | Backend | source movie id |
| filmname_child | `public.dim_movie.movie_title` | Derived | display title |
| filmid_parent | `public.dim_movie.movie_parent_id` | Derived | hierarchy |
| filmname_parent | `public.dim_movie.movie_parent_title` | Derived | hierarchy title |
| releasedate | `public.dim_movie.movie_release_date` | Backend | source release date |
| movie_tms_id | `public.dim_movie.movie_tms_id` | Third-party | external id |
| chainname | `public.dim_chain.chain_name` | Derived | display label |
| chainid | `public.dim_chain.chain_id` | Backend | source chain id |
| pos_vendor | `public.dim_chain.point_of_sale_vendor` | Third-party | vendor classification |
| awards_program_name | `public.dim_chain.awards_program_name` | Derived | descriptive label |
| theater_id | `public.dim_theater.theater_id` | Backend | source theater id |
| theater_code | `public.dim_theater.theater_code` | Derived | display code |
| theater_name | `public.dim_theater.theater_name` | Derived | display name |
| pdi_id | `public.dim_theater.pdi_id` | Backend | source identifier |
| pos_theater_id | `public.dim_theater.point_of_sale_theater_id` | Backend | POS identifier |
| latitude | `public.dim_theater.latitude` | Backend | location coordinate |
| longitude | `public.dim_theater.longitude` | Backend | location coordinate |
| point_of_sale_id | `public.dim_theater.point_of_sale_id` | Backend | POS id |
| address_1 | `public.dim_theater.address_1` | Backend | address |
| address_2 | `public.dim_theater.address_2` | Backend | address |
| city_name | `public.dim_theater.city_name` | Backend | address |
| state_code | `public.dim_theater.state_code` | Backend | address |
| zip_code | `public.dim_theater.zip_code` | Backend | address |
| country_code | `public.dim_theater.country_code` | Backend | address |
| tht_alternate_theater_phone_name | `public.dim_theater.alternate_theater_phone_name` | Derived | theater metadata |
| chain_specific_site_id | `public.dim_theater.chain_specific_site_id` | Backend | theater metadata |
| paymentech_terminal_id | `public.dim_theater.paymentech_terminal_id` | Third-party | payment terminal metadata |
| cybersource_terminal_id | `public.dim_theater.cybersource_terminal_id` | Third-party | payment terminal metadata |
| screen_count | `public.dim_theater.screen_count` | Backend | theater metadata |
| seat_count | `public.dim_theater.seat_count` | Backend | theater metadata |
| theater_time_zone_name | `public.dim_theater.theater_time_zone_name` | Backend | timezone metadata |
| transaction_type_id | `public.dim_transaction_type.transaction_type_id` | Backend | type id |
| transaction_type_name | `public.dim_transaction_type.transaction_type_name` | Derived | display name |
| user_agent_id | `public.dim_user_agent.user_agent_id` | Backend | source UA id conformed |
| user_agent_description | `public.dim_user_agent.user_agent_description` | Derived | descriptive label |
| purchase_funnel_id | `public.dim_purchase_funnel.purchase_funnel_id` | Backend | funnel id |
| purchase_funnel_name | `public.dim_purchase_funnel.purchase_funnel_name` | Derived | funnel label |
| return_method | `public.dim_return_type.return_method` | Derived | descriptive return method |
| return_type_code | `public.dim_return_type.return_type_code` | Backend | code |
| return_type_description | `public.dim_return_type.return_type_description` | Derived | description |
| property_id | `public.dim_property.property_id` | Backend | property id |
| property_group | `public.dim_property.property_group` | Derived | property group label |
| partner_channel | `public.dim_partner_channel.partner_channel_id` | Third-party | partner channel |
| partner_display_name | `public.dim_partner_channel.partner_display_name` | Third-party | partner display |
| partner_platform | `public.dim_partner_platform.partner_platform_id` | Third-party | partner platform |
| braintree_merchant_id | `public.dim_theater.braintree_merchant_id` | Third-party | merchant mapping |

---

## Group D — Raw Join in Analytics Query

### Pattern Definition

```text
analytics query directly joins raw_import_cache table
(no public dimension mediation for selected field)
```

### Fields

| Analytics Column | Raw Join Source | Source | Notes |
|---|---|---|---|
| dma_market_name | `raw_import_cache.designatedMarketArea.market_name` via `dma.zip_code = public.dim_theater.zip_code` | Third-party | raw join occurs directly in analytics SQL |

---

## Group E — Post-Insert Refresh Pattern (Return Sync)

### Pattern Definition

After INSERT, analytics SQL runs an UPDATE for return-sensitive metrics:

```sql
UPDATE analytics.transactions_data_source
SET returned_charity_amount = public.fact_transaction.returned_charity_amount,
    returned_fee_amount = public.fact_transaction.returned_fee_amount,
    returned_amount = public.fact_transaction.returned_amount
...
```

This catches newly observed returns where purchase may be older.

### Fields

| Analytics Column | Insert Source | Refresh Source | Source | Notes |
|---|---|---|---|---|
| returned_amount | `public.fact_transaction.returned_amount` | `public.fact_transaction.returned_amount` | Derived | explicitly refreshed post-insert |
| returned_fee_amount | `public.fact_transaction.returned_fee_amount` | `public.fact_transaction.returned_fee_amount` | Derived | explicitly refreshed post-insert |
| returned_charity_amount | `public.fact_transaction.returned_charity_amount` | `public.fact_transaction.returned_charity_amount` | Derived | explicitly refreshed post-insert |

---

## Coverage Notes

- The provided analytics SQL insert list in your paste includes **112 fields**, not 96.
- This flow-grouped document covers those listed fields by pattern.
- If you want a strict **96-field** version, provide the exact target schema snapshot (column list + date/commit), and we can lock coverage to that version.

---

## Quick Validation Rules

Use these checks when reviewing any field:

1. If analytics expression is exactly `public.fact_transaction.<x>` (or rename only) → **Group A**.
2. If analytics expression uses CASE / DATEDIFF / CONVERT_TIMEZONE → **Group B**.
3. If analytics expression selects from `public.dim_*` joined by `public.fact_transaction.*_key` → **Group C**.
4. If analytics expression selects from `raw_import_cache.*` directly in analytics query → **Group D**.
5. If field is updated in post-insert UPDATE block → also belongs to **Group E** overlay.

---

## Link Placeholders (to replace)

- `LINK_RAW_*` → files in `fandango/dataeng_datawarehouse/dags/ticketing/warehouse_ticketing_half_hourly/sql/extractTicketingTables/`
- `LINK_STG_*` → `fandango/jaws/.../factTransactionBase.sql`
- `LINK_AN_*` → `fandango/jaws/.../analyticsTransactionsDataSource.sql`
