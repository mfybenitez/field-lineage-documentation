# analytics.transactions_data_source — Field Lineage (Grouped by Same Flow)

---

<details open>
<summary><strong>Group A — Direct Pass-Through</strong></summary>

### Flow Diagram (Direct field projection from `public.fact_transaction`)
```text
Raw (raw_import_cache.transaction, raw_import_cache.transactionCommitted)
-> Staging (warehouse_loader.fact_transaction_base)
-> Public (public.fact_transaction)
-> Analytics (analytics.transactions_data_source)
```

<details>
<summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Transaction_GUID | varchar | Transaction identifier | TXN-123 | Backend | LINK_RAW_A_001 | Base transaction key extracted from source transaction table. |
| raw_import_cache.transaction | one row per transaction_guid | Create_dm | timestamp | Transaction start datetime UTC | 2026-10-01 16:10:00 | Backend | LINK_RAW_A_002 | Raw start datetime used to populate start timestamp lineage. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Create_dm | timestamp | Commit datetime UTC | 2026-10-01 16:11:05 | Backend | LINK_RAW_A_003 | Raw commit event timestamp feeding commit lineage. |
| raw_import_cache.transactionRolledBack | one row per rolledback transaction_guid | Create_dm | timestamp | Rollback datetime UTC | NULL | Backend | LINK_RAW_A_004 | Raw rollback timestamp feeding rollback lineage. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Confirmation_cd | varchar | POS confirmation code | CNF908812 | Backend | LINK_RAW_A_005 | Direct source for POS confirmation. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | ReservedSeats_tx | varchar | Reserved seats payload | A10,A11 | Backend | LINK_RAW_A_006 | Raw reserved seat text used for seating outputs. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Loyalty_cd | varchar | Loyalty code | LOY123 | Backend | LINK_RAW_A_007 | Raw loyalty value forwarded through layers. |
| raw_import_cache.transaction | one row per transaction_guid | DeviceType_cd | varchar | Device type code | Mobile | Backend | LINK_RAW_A_008 | Raw device classification. |
| raw_import_cache.transactionPerformanceAttribute | one row per tx-performance | Performance_dm | timestamp | Performance local datetime | 2026-10-05 19:30:00 | Backend | LINK_RAW_A_009 | Raw local showtime value. |
| raw_import_cache.transactionAttribute | one row per tx attribute | value_tx (CharityName) | varchar | Charity name attribute | Save The Arts | Backend | LINK_RAW_A_010 | Source attribute for charity name. |
| raw_import_cache.transactionAttribute | one row per tx attribute | value_tx (PartnerDefinedData) | varchar | Partner-defined payload | campaign=fall | Backend | LINK_RAW_A_011 | Source attribute for partner payload. |
| raw_import_cache.transactionAttribute | one row per tx attribute | value_tx (IsBuyNowOrder) | varchar | Buy-now marker | true | Backend | LINK_RAW_A_012 | Source value for upstream booleanization. |
| raw_import_cache.transactionAttribute | one row per tx attribute | value_tx (NetworkToken) | varchar | Network token marker | true | Backend | LINK_RAW_A_013 | Source for network token usage derivation. |
| raw_import_cache.transactionItem | one row per transaction item | Item_qy / Item_at / fee-tax fields | numeric/int | Item and monetary components | 2 / 35.00 | Backend | LINK_RAW_A_014 | Financial primitives aggregated upstream into public transaction measures. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_guid | varchar | Transaction identifier | TXN-123 | Backend | LINK_STG_A_001 | Direct pass-through key from raw transaction GUID. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_start_datetime | timestamp | Start datetime UTC | 2026-10-01 16:10:00 | Backend | LINK_STG_A_002 | Cast from raw transaction create timestamp. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_commit_datetime | timestamp | Commit datetime UTC | 2026-10-01 16:11:05 | Backend | LINK_STG_A_003 | Sourced from committed transaction create timestamp. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_rollback_datetime | timestamp | Rollback datetime UTC | NULL | Backend | LINK_STG_A_004 | Sourced from rollback transaction timestamp when present. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_end_datetime | timestamp | Final lifecycle timestamp | 2026-10-01 16:11:05 | Derived | LINK_STG_A_005 | Derived via CASE prioritizing rollback/commit/start timeline. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_STG_A_006 | Passes confirmation code downstream. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating indicator | true | Derived | LINK_STG_A_007 | Boolean derived from reserved seats text existence. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | reserved_seats | varchar | Reserved seats payload | A10,A11 | Backend | LINK_STG_A_008 | Trimmed/standardized reserved seat text. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY123 | Backend | LINK_STG_A_009 | Trimmed loyalty code from committed record. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-05 19:30:00 | Backend | LINK_STG_A_010 | Pulled from performance attribute timestamp. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_STG_A_011 | Direct map from raw device code. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_test | boolean | Test transaction flag | false | Derived | LINK_STG_A_012 | Multi-condition test classification logic in staging SQL. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived | LINK_STG_A_013 | Derived from fanclub attribute values. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived | LINK_STG_A_014 | Derived from buy-now attribute values. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_STG_A_015 | Attribute extraction pass-through. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_defined_data_value | varchar | Partner payload | campaign=fall | Backend | LINK_STG_A_016 | Attribute extraction pass-through. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | network_token_usage | boolean | Network token usage | true | Derived | LINK_STG_A_017 | Normalized true/false derivation from attribute text. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_plf / is_imax / is_3d / is_higher_fee_amenity | boolean | Amenity indicators | true/false | Derived | LINK_STG_A_018 | Amenity-aggregate temp-table logic populates these flags. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | transaction_guid | varchar | Transaction identifier | TXN-123 | Backend | LINK_PUB_A_001 | Directly retained transaction key. |
| public.fact_transaction | one row per transaction_guid | is_successful | boolean | Success flag | true | Derived | LINK_PUB_A_002 | Upstream outcome derivation retained in public fact. |
| public.fact_transaction | one row per transaction_guid | transaction_start_datetime | timestamp | Start datetime UTC | 2026-10-01 16:10:00 | Backend | LINK_PUB_A_003 | Pass-through from staging start datetime. |
| public.fact_transaction | one row per transaction_guid | transaction_commit_datetime | timestamp | Commit datetime UTC | 2026-10-01 16:11:05 | Backend | LINK_PUB_A_004 | Pass-through from staging commit datetime. |
| public.fact_transaction | one row per transaction_guid | transaction_rollback_datetime | timestamp | Rollback datetime UTC | NULL | Backend | LINK_PUB_A_005 | Pass-through from staging rollback datetime. |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime | timestamp | Final lifecycle datetime | 2026-10-01 16:11:05 | Derived | LINK_PUB_A_006 | Pass-through of upstream derived end timestamp. |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime_local | timestamp | Local end datetime | 2026-10-01 09:11:05 | Derived | LINK_PUB_A_007 | Localized timestamp retained in public model. |
| public.fact_transaction | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_PUB_A_008 | Direct pass-through. |
| public.fact_transaction | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating indicator | true | Derived | LINK_PUB_A_009 | Pass-through of staging boolean derivation. |
| public.fact_transaction | one row per transaction_guid | reserved_seats | varchar | Reserved seats payload | A10,A11 | Backend | LINK_PUB_A_010 | Pass-through reserved seat values. |
| public.fact_transaction | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY123 | Backend | LINK_PUB_A_011 | Pass-through loyalty value. |
| public.fact_transaction | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime | 2026-10-05 19:30:00 | Backend | LINK_PUB_A_012 | Pass-through showtime. |
| public.fact_transaction | one row per transaction_guid | payment_types | varchar | Payment method labels | VISA | Backend | LINK_PUB_A_013 | Public fact financial attribute. |
| public.fact_transaction | one row per transaction_guid | processor_order_ids | varchar | Processor order ids | ord_123 | Third-party | LINK_PUB_A_014 | Processor-generated references. |
| public.fact_transaction | one row per transaction_guid | item_count | int | Item count | 2 | Derived | LINK_PUB_A_015 | Aggregated count used as analytics `item_qty`. |
| public.fact_transaction | one row per transaction_guid | gross_transaction_amount + fee/return/charity fields | numeric(19,4) | Monetary metrics | 42.50 etc | Backend/Derived | LINK_PUB_A_016 | Public fact stores normalized financial outputs consumed directly by analytics. |
| public.fact_transaction | one row per transaction_guid | POS_reference_cd | varchar | POS reference code | POSREF1 | Backend | LINK_PUB_A_017 | Direct pass-through reference code. |
| public.fact_transaction | one row per transaction_guid | is_test / is_fanclub / is_buynow / network_token_usage | boolean | Behavioral flags | true/false | Derived | LINK_PUB_A_018 | Upstream staging derivations preserved. |
| public.fact_transaction | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_PUB_A_019 | Pass-through from staging. |
| public.fact_transaction | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_PUB_A_020 | Pass-through from staging attribute extraction. |
| public.fact_transaction | one row per transaction_guid | partner_defined_data_value | varchar | Partner payload | campaign=fall | Backend | LINK_PUB_A_021 | Pass-through from staging attribute extraction. |
| public.fact_transaction | one row per transaction_guid | is_plf / is_imax / is_3d / is_higher_fee_amenity | boolean | Amenity flags | true/false | Derived | LINK_PUB_A_022 | Pass-through of staging amenity derivations. |

</details>

<details open>
<summary><strong>Analytics (Normalized: 48 rows)</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_guid | varchar | Transaction identifier | TXN-123 | Backend | LINK_AN_A_001 | Direct select from public.fact_transaction.transaction_guid. |
| analytics.transactions_data_source | one row per transaction_guid | is_successful | boolean | Success flag | true | Derived | LINK_AN_A_002 | Direct select from public.fact_transaction.is_successful. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_start_datetime_utc | timestamp | UTC start datetime | 2026-10-01 16:10:00 | Backend | LINK_AN_A_003 | Alias-only projection from public.fact_transaction.transaction_start_datetime. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_commit_datetime_utc | timestamp | UTC commit datetime | 2026-10-01 16:11:05 | Backend | LINK_AN_A_004 | Alias-only projection from public.fact_transaction.transaction_commit_datetime. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_rollback_datetime_utc | timestamp | UTC rollback datetime | NULL | Backend | LINK_AN_A_005 | Alias-only projection from public.fact_transaction.transaction_rollback_datetime. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_utc | timestamp | UTC end datetime | 2026-10-01 16:11:05 | Derived | LINK_AN_A_006 | Alias-only projection from public.fact_transaction.transaction_end_datetime. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_local | timestamp | Local end datetime | 2026-10-01 09:11:05 | Derived | LINK_AN_A_007 | Direct pass-through projection from public.fact_transaction.transaction_end_datetime_local. |
| analytics.transactions_data_source | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_AN_A_008 | Direct pass-through from public.fact_transaction.point_of_sale_confirmation_code. |
| analytics.transactions_data_source | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating indicator | true | Derived | LINK_AN_A_009 | Direct pass-through from public.fact_transaction.is_reserved_seating. |
| analytics.transactions_data_source | one row per transaction_guid | reserved_seats | varchar | Reserved seats payload | A10,A11 | Backend | LINK_AN_A_010 | Direct pass-through from public.fact_transaction.reserved_seats. |
| analytics.transactions_data_source | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY123 | Backend | LINK_AN_A_011 | Direct pass-through from public.fact_transaction.loyalty_code. |
| analytics.transactions_data_source | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-05 19:30:00 | Backend | LINK_AN_A_012 | Direct pass-through from public.fact_transaction.performance_datetime_local. |
| analytics.transactions_data_source | one row per transaction_guid | payment_types | varchar | Payment method labels | VISA | Backend | LINK_AN_A_013 | Direct pass-through from public.fact_transaction.payment_types. |
| analytics.transactions_data_source | one row per transaction_guid | processor_order_ids | varchar | Processor order ids | ord_123 | Third-party | LINK_AN_A_014 | Direct pass-through from public.fact_transaction.processor_order_ids. |
| analytics.transactions_data_source | one row per transaction_guid | item_qty | int | Item quantity | 2 | Derived | LINK_AN_A_015 | Rename-only projection of public.fact_transaction.item_count as item_qty. |
| analytics.transactions_data_source | one row per transaction_guid | gross_transaction_amount | numeric(19,4) | Gross transaction amount | 42.50 | Backend | LINK_AN_A_016 | Direct pass-through from public.fact_transaction.gross_transaction_amount. |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee amount | 5.00 | Backend | LINK_AN_A_017 | Direct pass-through from public.fact_transaction.sales_fee_amount. |
| analytics.transactions_data_source | one row per transaction_guid | waived_sales_fee_ticket_count | int | Waived sales fee ticket count | 0 | Derived | LINK_AN_A_018 | Direct pass-through from public.fact_transaction.waived_sales_fee_ticket_count. |
| analytics.transactions_data_source | one row per transaction_guid | waived_fanclub_fee_ticket_count | int | Waived fanclub fee ticket count | 0 | Derived | LINK_AN_A_019 | Direct pass-through from public.fact_transaction.waived_fanclub_fee_ticket_count. |
| analytics.transactions_data_source | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount | 0.00 | Derived | LINK_AN_A_020 | Insert-time pass-through from public.fact_transaction.returned_amount (also refreshed in Group E). |
| analytics.transactions_data_source | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | 0.00 | Derived | LINK_AN_A_021 | Insert-time pass-through from public.fact_transaction.returned_fee_amount (also refreshed in Group E). |
| analytics.transactions_data_source | one row per transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Backend | LINK_AN_A_022 | Direct pass-through from public.fact_transaction.ticket_sales_amount. |
| analytics.transactions_data_source | one row per transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Backend | LINK_AN_A_023 | Direct pass-through from public.fact_transaction.waived_sales_fee_amount. |
| analytics.transactions_data_source | one row per transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Backend | LINK_AN_A_024 | Direct pass-through from public.fact_transaction.waived_fanclub_fee_amount. |
| analytics.transactions_data_source | one row per transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee amount | 1.50 | Backend | LINK_AN_A_025 | Direct pass-through from public.fact_transaction.surcharge_fee_amount. |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_return_amount | numeric(19,4) | Sales fee return amount | 0.00 | Derived | LINK_AN_A_026 | Direct pass-through from public.fact_transaction.sales_fee_return_amount. |
| analytics.transactions_data_source | one row per transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Surcharge fee return amount | 0.00 | Derived | LINK_AN_A_027 | Direct pass-through from public.fact_transaction.surcharge_fee_return_amount. |
| analytics.transactions_data_source | one row per transaction_guid | reservation_fee_return_amount | numeric(19,4) | Reservation fee return amount | 0.00 | Derived | LINK_AN_A_028 | Direct pass-through from public.fact_transaction.reservation_fee_return_amount. |
| analytics.transactions_data_source | one row per transaction_guid | tax_return_amount | numeric(19,4) | Tax return amount | 0.00 | Derived | LINK_AN_A_029 | Direct pass-through from public.fact_transaction.tax_return_amount. |
| analytics.transactions_data_source | one row per transaction_guid | card_fee_return_amount | numeric(19,4) | Card fee return amount | 0.00 | Derived | LINK_AN_A_030 | Direct pass-through from public.fact_transaction.card_fee_return_amount. |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee amount for returned purchase | 0.00 | Derived | LINK_AN_A_031 | Direct pass-through from public.fact_transaction.sales_fee_amount_for_returned_purchase. |
| analytics.transactions_data_source | one row per transaction_guid | POS_reference_cd | varchar | POS reference code | POSREF1 | Backend | LINK_AN_A_032 | Direct pass-through from public.fact_transaction.POS_reference_cd. |
| analytics.transactions_data_source | one row per transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived | LINK_AN_A_033 | Direct pass-through from public.fact_transaction.taxable_sales_fee_amount. |
| analytics.transactions_data_source | one row per transaction_guid | is_test | boolean | Test transaction indicator | false | Derived | LINK_AN_A_034 | Direct pass-through from public.fact_transaction.is_test. |
| analytics.transactions_data_source | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_AN_A_035 | Direct pass-through from public.fact_transaction.deviceType. |
| analytics.transactions_data_source | one row per transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived | LINK_AN_A_036 | Direct pass-through from public.fact_transaction.is_fanclub. |
| analytics.transactions_data_source | one row per transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Backend | LINK_AN_A_037 | Direct pass-through from public.fact_transaction.charity_amount. |
| analytics.transactions_data_source | one row per transaction_guid | true_gross_transaction_amount | numeric(19,4) | True gross transaction amount | 43.50 | Derived | LINK_AN_A_038 | Direct pass-through from public.fact_transaction.true_gross_transaction_amount. |
| analytics.transactions_data_source | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | 0.00 | Derived | LINK_AN_A_039 | Insert-time pass-through from public.fact_transaction.returned_charity_amount (also refreshed in Group E). |
| analytics.transactions_data_source | one row per transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | 0.00 | Derived | LINK_AN_A_040 | Direct pass-through from public.fact_transaction.charity_return_amount. |
| analytics.transactions_data_source | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_AN_A_041 | Direct pass-through from public.fact_transaction.charity_name. |
| analytics.transactions_data_source | one row per transaction_guid | partner_defined_data_value | varchar | Partner-defined payload | campaign=fall | Backend | LINK_AN_A_042 | Direct pass-through from public.fact_transaction.partner_defined_data_value. |
| analytics.transactions_data_source | one row per transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived | LINK_AN_A_043 | Direct pass-through from public.fact_transaction.is_buynow. |
| analytics.transactions_data_source | one row per transaction_guid | network_token_usage | boolean | Network token usage flag | true | Derived | LINK_AN_A_044 | Direct pass-through from public.fact_transaction.network_token_usage. |
| analytics.transactions_data_source | one row per transaction_guid | is_plf | boolean | PLF indicator | true | Derived | LINK_AN_A_045 | Direct pass-through from public.fact_transaction.is_plf. |
| analytics.transactions_data_source | one row per transaction_guid | is_imax | boolean | IMAX indicator | false | Derived | LINK_AN_A_046 | Direct pass-through from public.fact_transaction.is_imax. |
| analytics.transactions_data_source | one row per transaction_guid | is_3d | boolean | 3D indicator | false | Derived | LINK_AN_A_047 | Direct pass-through from public.fact_transaction.is_3d. |
| analytics.transactions_data_source | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity indicator | true | Derived | LINK_AN_A_048 | Direct pass-through from public.fact_transaction.is_higher_fee_amenity. |

</details>
</details>

<details>
<summary><strong>Group B — Derived in Analytics</strong></summary>

### Flow Diagram (Analytics-layer calculations using public fact + movie/affiliate/channel context)
```text
Raw (raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute)
-> Staging (warehouse_loader.fact_transaction_base)
-> Public (public.fact_transaction, public.dim_movie, public.dim_affiliate, public.dim_channel)
-> Analytics (analytics.transactions_data_source)
```

<details><summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Create_dm | timestamp | Base UTC event timestamp | 2026-10-01 16:10:00 | Backend | LINK_RAW_B_001 | Seeds UTC lifecycle timestamps later transformed in analytics. |
| raw_import_cache.transactionPerformanceAttribute | one row per tx-performance | Performance_dm | timestamp | Local showtime timestamp | 2026-10-05 19:30:00 | Backend | LINK_RAW_B_002 | Input for purchase-to-showtime and release-to-showtime deltas. |
| raw_import_cache.transaction | one row per transaction_guid | Affiliate_id | int | Affiliate id | 11879 | Backend | LINK_RAW_B_003 | Drives CASE mapping for grouping/apps/platform outputs. |
| raw_import_cache.transaction | one row per transaction_guid | Channel_cd | varchar | Channel code | WAP | Backend | LINK_RAW_B_004 | Drives channel-aware CASE logic for platform mapping. |

</details>

<details><summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_start_datetime / transaction_commit_datetime / transaction_rollback_datetime / transaction_end_datetime | timestamp | UTC lifecycle timestamps | 2026-10-01 ... | Backend/Derived | LINK_STG_B_001 | Staging standardizes lifecycle timestamps consumed by public fact and then converted in analytics. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | performance_datetime_local | timestamp | Local performance datetime | 2026-10-05 19:30:00 | Backend | LINK_STG_B_002 | Passes local performance timestamp downstream for date arithmetic. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | affiliate_key / channel_key | bigint | Dimension keys | 8123 / 291 | ETL | LINK_STG_B_003 | Enables analytics CASE logic by resolving affiliate/channel context through dimensions. |

</details>

<details><summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | transaction_start_datetime / transaction_commit_datetime / transaction_rollback_datetime / transaction_end_datetime | timestamp | UTC timestamps used in analytics derivations | 2026-10-01 ... | Backend/Derived | LINK_PUB_B_001 | Inputs for timezone conversions and purchase-day truncation logic. |
| public.fact_transaction | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-05 19:30:00 | Backend | LINK_PUB_B_002 | Input to showtime-related delta calculations. |
| public.dim_movie | one row per movie_key | movie_release_date | date | Movie release date | 2026-09-14 | Backend | LINK_PUB_B_003 | Input to all release-based DATEDIFF calculations. |
| public.dim_affiliate | one row per affiliate_key | affiliate_id | int | Affiliate identifier | 11879 | Backend | LINK_PUB_B_004 | Input to CASE buckets for grouping and app buckets. |
| public.dim_channel | one row per channel_key | channel_name | varchar | Channel detail | WAP | Backend | LINK_PUB_B_005 | Input to platform CASE branching. |

</details>

<details open><summary><strong>Analytics (Normalized: 11 rows)</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_start_datetime_los_angeles | timestamp | Pacific start datetime | 2026-10-01 09:10:00 | Derived | LINK_AN_B_001 | `CONVERT_TIMEZONE('US/Pacific', public.fact_transaction.transaction_start_datetime)`. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_commit_datetime_los_angeles | timestamp | Pacific commit datetime | 2026-10-01 09:11:05 | Derived | LINK_AN_B_002 | Timezone conversion from UTC commit timestamp. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_rollback_datetime_los_angeles | timestamp | Pacific rollback datetime | NULL | Derived | LINK_AN_B_003 | Timezone conversion from UTC rollback timestamp. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_los_angeles | timestamp | Pacific end datetime | 2026-10-01 09:11:05 | Derived | LINK_AN_B_004 | Timezone conversion from UTC end timestamp. |
| analytics.transactions_data_source | one row per transaction_guid | deltareleasetopurchasedate | int | Days release to purchase day | 17 | Derived | LINK_AN_B_005 | DATEDIFF between movie release date and Pacific purchase date. |
| analytics.transactions_data_source | one row per transaction_guid | deltareleasetoshowtimedate | int | Days release to showtime day | 21 | Derived | LINK_AN_B_006 | DATEDIFF between release date and local performance date. |
| analytics.transactions_data_source | one row per transaction_guid | deltapurchasetoshowtimedate | int | Days purchase day to showtime day | 4 | Derived | LINK_AN_B_007 | DATEDIFF between Pacific purchase day and local showtime day. |
| analytics.transactions_data_source | one row per transaction_guid | Affiliate Grouping | varchar | Detailed affiliate bucket | iPhone App | Derived | LINK_AN_B_008 | CASE map by affiliate identifiers into named affiliate buckets. |
| analytics.transactions_data_source | one row per transaction_guid | Affiliate Grouping - HIGH LEVEL BINS | varchar | High-level affiliate bucket | Fandango O&O | Derived | LINK_AN_B_009 | CASE map consolidating affiliate detail into high-level bins. |
| analytics.transactions_data_source | one row per transaction_guid | apps | varchar | App label bucket | Fandango App: iPhone | Derived | LINK_AN_B_010 | CASE map assigning app-centric reporting label. |
| analytics.transactions_data_source | one row per transaction_guid | platform | varchar | Platform reporting bucket | Mobile Apps | Derived | LINK_AN_B_011 | CASE map using affiliate/channel logic for platform rollup. |

</details>
</details>

<details>
<summary><strong>Group C — Dimension-Sourced</strong></summary>

### Flow Diagram (Dimension key mapping, then dimension attribute projection)
```text
Raw (raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionAttribute)
-> Staging (warehouse_loader.fact_transaction_base)
-> Public (public.fact_transaction + public.dim_*)
-> Analytics (analytics.transactions_data_source)
```

<details><summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Affiliate_id / Channel_cd / Transaction_tp / UserAgent_nm / Property_nm | mixed | Raw ids/codes for dimension resolution | 11879 / WAP / Purchase | Backend | LINK_RAW_C_001 | Source identifiers mapped to conformed surrogate keys in staging. |
| raw_import_cache.transactionPerformanceAttribute | one row per tx-performance | Movie_detail_id / Theater_pkey | int | Raw movie and theater identifiers | 10293 / 4451 | Backend | LINK_RAW_C_002 | Raw identifiers used for movie/theater dimension key lookup. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Confirmation_cd | varchar | Confirmation code for return-type mapping | REF123 | Backend | LINK_RAW_C_003 | Prefix logic supports return type key assignment upstream. |
| raw_import_cache.transactionPurchaseFunnel | one row per transaction_guid + funnel | Funnel_cd | varchar | Purchase funnel code | 14 | Backend | LINK_RAW_C_004 | Raw funnel id mapped to purchase funnel key. |
| raw_import_cache.transactionAttribute | one row per tx attribute | value_tx (partner-channel / partner-platform) | varchar | Partner channel/platform values | TMO / WEBVIEW | Third-party | LINK_RAW_C_005 | Attribute values mapped to partner dimension keys. |

</details>

<details><summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | affiliate_key / channel_key / movie_key / theater_key / chain_key | bigint | Core conformed keys | 8123 / 291 / 129384 | ETL | LINK_STG_C_001 | Staging joins raw ids to conformed dimensions and stores surrogate keys. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_type_key / return_type_key | bigint | Type keys | 82 / 16 | ETL | LINK_STG_C_002 | Keys assigned via transaction type and return-type mapping logic. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | user_agent_key / purchase_funnel_key / property_key | bigint | Additional conformed keys | 298771 / 7231 / 45 | ETL | LINK_STG_C_003 | Key mapping for UA, funnel, and property dimensions. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_channel_key / partner_platform_key | bigint | Partner dimension keys | 912 / 117 | ETL | LINK_STG_C_004 | Attribute-based partner values mapped to partner dims. |

</details>

<details><summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_affiliate | one row per affiliate_key | affiliate_name / affiliate_group_name / affiliate_category_name / affiliate_id | mixed | Affiliate attributes | Fandango iOS App / 11879 | Backend/Derived | LINK_PUB_C_001 | Affiliate columns selected directly into analytics from affiliate join. |
| public.dim_channel | one row per channel_key | channel_group / channel_name | varchar | Channel attributes | App / WAP | Backend/Derived | LINK_PUB_C_002 | Channel columns projected into analytics fields. |
| public.dim_movie | one row per movie_key | movie_id / movie_title / movie_parent_id / movie_parent_title / movie_release_date / movie_tms_id | mixed | Movie attributes | 10293 / Example Movie | Backend/Derived/Third-party | LINK_PUB_C_003 | Movie columns populate film and release/TMS analytics outputs. |
| public.dim_chain | one row per chain_key | chain_name / chain_id / point_of_sale_vendor / awards_program_name | mixed | Chain attributes | Example Cinemas / CHN019 | Backend/Derived/Third-party | LINK_PUB_C_004 | Chain columns projected into analytics chain outputs. |
| public.dim_theater | one row per theater_key | theater/location/POS/terminal/capacity/timezone columns | mixed | Theater attributes | theater_id 4451, zip 90028 | Backend/Derived/Third-party | LINK_PUB_C_005 | Theater columns projected into analytics location outputs. |
| public.dim_transaction_type | one row per transaction_type_key | transaction_type_id / transaction_type_name | varchar | Transaction type attributes | Purchase / Ticket Purchase | Backend/Derived | LINK_PUB_C_006 | Type attributes projected into analytics type outputs. |
| public.dim_user_agent | one row per user_agent_key | user_agent_id / user_agent_description | varchar | User agent attributes | UA-1 / iOS Safari | Backend/Derived | LINK_PUB_C_007 | UA attributes projected into analytics customer outputs. |
| public.dim_purchase_funnel | one row per purchase_funnel_key | purchase_funnel_id / purchase_funnel_name | varchar | Funnel attributes | 14 / Guest Checkout | Backend/Derived | LINK_PUB_C_008 | Funnel attributes projected into analytics customer outputs. |
| public.dim_return_type | one row per return_type_key | return_method / return_type_code / return_type_description | varchar | Return type attributes | Self-service / REF | Backend/Derived | LINK_PUB_C_009 | Return-type attributes projected into analytics outputs. |
| public.dim_property | one row per property_key | property_id / property_group | varchar | Property attributes | Fandango USA / Fandango | Backend/Derived | LINK_PUB_C_010 | Property attributes projected into analytics outputs. |
| public.dim_partner_channel | one row per partner_channel_key | partner_channel_id / partner_display_name | varchar | Partner channel attributes | TMO / T-Mobile Tuesdays | Third-party | LINK_PUB_C_011 | Partner channel attributes projected into analytics outputs. |
| public.dim_partner_platform | one row per partner_platform_key | partner_platform_id | varchar | Partner platform attribute | WEBVIEW | Third-party | LINK_PUB_C_012 | Partner platform attribute projected into analytics outputs. |

</details>

<details open><summary><strong>Analytics (Normalized: 65 rows)</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | affiliate_name | varchar | Affiliate name | Fandango iOS App | Derived | LINK_AN_C_001 | Selected from public.dim_affiliate via affiliate key join path. |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_group_name | varchar | Affiliate group name | Mobile App | Derived | LINK_AN_C_002 | Selected from public.dim_affiliate. |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_category_name | varchar | Affiliate category name | Owned & Operated | Derived | LINK_AN_C_003 | Selected from public.dim_affiliate. |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_id | int | Affiliate id | 11879 | Backend | LINK_AN_C_004 | Selected from public.dim_affiliate. |
| analytics.transactions_data_source | one row per transaction_guid | channel | varchar | Channel group | App | Derived | LINK_AN_C_005 | Selected from public.dim_channel.channel_group. |
| analytics.transactions_data_source | one row per transaction_guid | channel_ungrouped | varchar | Channel name | WAP | Backend | LINK_AN_C_006 | Selected from public.dim_channel.channel_name. |
| analytics.transactions_data_source | one row per transaction_guid | filmid_child | int | Child movie id | 10293 | Backend | LINK_AN_C_007 | Selected from public.dim_movie.movie_id. |
| analytics.transactions_data_source | one row per transaction_guid | filmname_child | varchar | Child movie title | Example Movie | Derived | LINK_AN_C_008 | Selected from public.dim_movie.movie_title. |
| analytics.transactions_data_source | one row per transaction_guid | filmid_parent | int | Parent movie id | 872 | Derived | LINK_AN_C_009 | Selected from public.dim_movie.movie_parent_id. |
| analytics.transactions_data_source | one row per transaction_guid | filmname_parent | varchar | Parent movie title | Example Franchise | Derived | LINK_AN_C_010 | Selected from public.dim_movie.movie_parent_title. |
| analytics.transactions_data_source | one row per transaction_guid | releasedate | date | Release date | 2026-09-14 | Backend | LINK_AN_C_011 | Selected from public.dim_movie.movie_release_date. |
| analytics.transactions_data_source | one row per transaction_guid | movie_tms_id | varchar | Movie TMS id | MV123456 | Third-party | LINK_AN_C_012 | Selected from public.dim_movie.movie_tms_id. |
| analytics.transactions_data_source | one row per transaction_guid | chainname | varchar | Chain name | Example Cinemas | Derived | LINK_AN_C_013 | Selected from public.dim_chain.chain_name. |
| analytics.transactions_data_source | one row per transaction_guid | chainid | varchar | Chain id | CHN019 | Backend | LINK_AN_C_014 | Selected from public.dim_chain.chain_id. |
| analytics.transactions_data_source | one row per transaction_guid | pos_vendor | varchar | POS vendor | Vista | Third-party | LINK_AN_C_015 | Selected from public.dim_chain.point_of_sale_vendor. |
| analytics.transactions_data_source | one row per transaction_guid | awards_program_name | varchar | Awards program name | Rewards+ | Derived | LINK_AN_C_016 | Selected from public.dim_chain.awards_program_name. |
| analytics.transactions_data_source | one row per transaction_guid | theater_id | int | Theater id | 4451 | Backend | LINK_AN_C_017 | Selected from public.dim_theater.theater_id. |
| analytics.transactions_data_source | one row per transaction_guid | theater_code | varchar | Theater code | TH4451 | Derived | LINK_AN_C_018 | Selected from public.dim_theater.theater_code. |
| analytics.transactions_data_source | one row per transaction_guid | theater_name | varchar | Theater name | Example Hollywood | Derived | LINK_AN_C_019 | Selected from public.dim_theater.theater_name. |
| analytics.transactions_data_source | one row per transaction_guid | pdi_id | varchar | PDI id | PDI9981 | Backend | LINK_AN_C_020 | Selected from public.dim_theater.pdi_id. |
| analytics.transactions_data_source | one row per transaction_guid | pos_theater_id | varchar | POS theater id | POS-4451 | Backend | LINK_AN_C_021 | Selected from public.dim_theater.point_of_sale_theater_id. |
| analytics.transactions_data_source | one row per transaction_guid | latitude | numeric | Latitude | 34.10 | Backend | LINK_AN_C_022 | Selected from public.dim_theater.latitude. |
| analytics.transactions_data_source | one row per transaction_guid | longitude | numeric | Longitude | -118.34 | Backend | LINK_AN_C_023 | Selected from public.dim_theater.longitude. |
| analytics.transactions_data_source | one row per transaction_guid | point_of_sale_id | varchar | POS id | POS01 | Backend | LINK_AN_C_024 | Selected from public.dim_theater.point_of_sale_id. |
| analytics.transactions_data_source | one row per transaction_guid | address_1 | varchar | Address line 1 | 123 Main St | Backend | LINK_AN_C_025 | Selected from public.dim_theater.address_1. |
| analytics.transactions_data_source | one row per transaction_guid | address_2 | varchar | Address line 2 | Suite 200 | Backend | LINK_AN_C_026 | Selected from public.dim_theater.address_2. |
| analytics.transactions_data_source | one row per transaction_guid | city_name | varchar | City name | Los Angeles | Backend | LINK_AN_C_027 | Selected from public.dim_theater.city_name. |
| analytics.transactions_data_source | one row per transaction_guid | state_code | varchar | State code | CA | Backend | LINK_AN_C_028 | Selected from public.dim_theater.state_code. |
| analytics.transactions_data_source | one row per transaction_guid | zip_code | varchar | ZIP code | 90028 | Backend | LINK_AN_C_029 | Selected from public.dim_theater.zip_code. |
| analytics.transactions_data_source | one row per transaction_guid | country_code | varchar | Country code | US | Backend | LINK_AN_C_030 | Selected from public.dim_theater.country_code. |
| analytics.transactions_data_source | one row per transaction_guid | tht_alternate_theater_phone_name | varchar | Alternate theater phone/name | Main Box Office | Derived | LINK_AN_C_031 | Selected from public.dim_theater.alternate_theater_phone_name. |
| analytics.transactions_data_source | one row per transaction_guid | chain_specific_site_id | varchar | Chain-specific site id | SITE-22 | Backend | LINK_AN_C_032 | Selected from public.dim_theater.chain_specific_site_id. |
| analytics.transactions_data_source | one row per transaction_guid | paymentech_terminal_id | varchar | Paymentech terminal id | PT-100 | Third-party | LINK_AN_C_033 | Selected from public.dim_theater.paymentech_terminal_id. |
| analytics.transactions_data_source | one row per transaction_guid | cybersource_terminal_id | varchar | Cybersource terminal id | CS-200 | Third-party | LINK_AN_C_034 | Selected from public.dim_theater.cybersource_terminal_id. |
| analytics.transactions_data_source | one row per transaction_guid | screen_count | int | Screen count | 12 | Backend | LINK_AN_C_035 | Selected from public.dim_theater.screen_count. |
| analytics.transactions_data_source | one row per transaction_guid | seat_count | int | Seat count | 1800 | Backend | LINK_AN_C_036 | Selected from public.dim_theater.seat_count. |
| analytics.transactions_data_source | one row per transaction_guid | theater_time_zone_name | varchar | Theater timezone name | America/Los_Angeles | Backend | LINK_AN_C_037 | Selected from public.dim_theater.theater_time_zone_name. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_type_id | varchar | Transaction type id | Purchase | Backend | LINK_AN_C_038 | Selected from public.dim_transaction_type.transaction_type_id. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_type_name | varchar | Transaction type name | Ticket Purchase | Derived | LINK_AN_C_039 | Selected from public.dim_transaction_type.transaction_type_name. |
| analytics.transactions_data_source | one row per transaction_guid | user_agent_id | varchar | User agent id | UA-1 | Backend | LINK_AN_C_040 | Selected from public.dim_user_agent.user_agent_id. |
| analytics.transactions_data_source | one row per transaction_guid | user_agent_description | varchar | User agent description | iOS Safari | Derived | LINK_AN_C_041 | Selected from public.dim_user_agent.user_agent_description. |
| analytics.transactions_data_source | one row per transaction_guid | purchase_funnel_id | varchar | Purchase funnel id | 14 | Backend | LINK_AN_C_042 | Selected from public.dim_purchase_funnel.purchase_funnel_id. |
| analytics.transactions_data_source | one row per transaction_guid | purchase_funnel_name | varchar | Purchase funnel name | Guest Checkout | Derived | LINK_AN_C_043 | Selected from public.dim_purchase_funnel.purchase_funnel_name. |
| analytics.transactions_data_source | one row per transaction_guid | return_method | varchar | Return method | Self-service | Derived | LINK_AN_C_044 | Selected from public.dim_return_type.return_method. |
| analytics.transactions_data_source | one row per transaction_guid | return_type_code | varchar | Return type code | REF | Backend | LINK_AN_C_045 | Selected from public.dim_return_type.return_type_code. |
| analytics.transactions_data_source | one row per transaction_guid | return_type_description | varchar | Return type description | Customer refund | Derived | LINK_AN_C_046 | Selected from public.dim_return_type.return_type_description. |
| analytics.transactions_data_source | one row per transaction_guid | property_id | varchar | Property id | Fandango USA | Backend | LINK_AN_C_047 | Selected from public.dim_property.property_id. |
| analytics.transactions_data_source | one row per transaction_guid | property_group | varchar | Property group | Fandango | Derived | LINK_AN_C_048 | Selected from public.dim_property.property_group. |
| analytics.transactions_data_source | one row per transaction_guid | partner_channel | varchar | Partner channel id | TMO | Third-party | LINK_AN_C_049 | Selected from public.dim_partner_channel.partner_channel_id. |
| analytics.transactions_data_source | one row per transaction_guid | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party | LINK_AN_C_050 | Selected from public.dim_partner_channel.partner_display_name. |
| analytics.transactions_data_source | one row per transaction_guid | partner_platform | varchar | Partner platform id | WEBVIEW | Third-party | LINK_AN_C_051 | Selected from public.dim_partner_platform.partner_platform_id. |
| analytics.transactions_data_source | one row per transaction_guid | braintree_merchant_id | varchar | Braintree merchant id | bt_merch_1 | Third-party | LINK_AN_C_052 | Selected from public.dim_theater.braintree_merchant_id. |
| analytics.transactions_data_source | one row per transaction_guid | convenience_fee_tax_amount | numeric(19,4) | Convenience fee tax amount | 0.75 | Derived | LINK_AN_C_053 | Selected from conformed transaction tax fields joined via public.fact_transaction context. |
| analytics.transactions_data_source | one row per transaction_guid | sales_tax_amount | numeric(19,4) | Sales tax amount | 2.10 | Derived | LINK_AN_C_054 | Selected from conformed tax outputs associated to transaction context. |
| analytics.transactions_data_source | one row per transaction_guid | reservation_fee_amount | numeric(19,4) | Reservation fee amount | 1.25 | Derived | LINK_AN_C_055 | Selected from conformed fee attributes in public-layer transaction outputs. |
| analytics.transactions_data_source | one row per transaction_guid | card_fee_amount | numeric(19,4) | Card fee amount | 0.40 | Derived | LINK_AN_C_056 | Selected from conformed card-fee outputs in public-layer transaction outputs. |
| analytics.transactions_data_source | one row per transaction_guid | ticket_qty | int | Ticket quantity | 2 | Derived | LINK_AN_C_057 | Selected from conformed ticket quantity output used for reporting. |
| analytics.transactions_data_source | one row per transaction_guid | package_qty | int | Package quantity | 0 | Derived | LINK_AN_C_058 | Selected from conformed package quantity output used for reporting. |
| analytics.transactions_data_source | one row per transaction_guid | concession_qty | int | Concession quantity | 0 | Derived | LINK_AN_C_059 | Selected from conformed concession quantity output used for reporting. |
| analytics.transactions_data_source | one row per transaction_guid | gift_card_amount | numeric(19,4) | Gift card amount applied | 0.00 | Derived | LINK_AN_C_060 | Selected from payment allocation outputs tied to transaction context. |
| analytics.transactions_data_source | one row per transaction_guid | discount_amount | numeric(19,4) | Discount amount | 3.00 | Derived | LINK_AN_C_061 | Selected from conformed discount outputs associated with transaction context. |
| analytics.transactions_data_source | one row per transaction_guid | promotion_code | varchar | Promotion code | FALL2026 | Backend | LINK_AN_C_062 | Selected from conformed promotion attributes joined by transaction context. |
| analytics.transactions_data_source | one row per transaction_guid | partner_order_id | varchar | Partner order id | partner-9981 | Third-party | LINK_AN_C_063 | Selected from partner-linked transaction attributes in conformed outputs. |
| analytics.transactions_data_source | one row per transaction_guid | customer_zip_code | varchar | Customer zip code | 90028 | Backend | LINK_AN_C_064 | Selected from customer/location conformed attributes tied to transaction context. |
| analytics.transactions_data_source | one row per transaction_guid | country_name | varchar | Country name | United States | Derived | LINK_AN_C_065 | Selected from conformed geography dimension attributes. |

</details>
</details>

<details>
<summary><strong>Group D — Raw-Enriched in Analytics</strong></summary>

### Flow Diagram (ZIP enrichment from raw DMA table at analytics step)
```text
Raw (raw_import_cache.designatedMarketArea)
-> Staging (warehouse_loader.fact_transaction_base)
-> Public (public.fact_transaction, public.dim_theater)
-> Analytics (analytics.transactions_data_source)
```

<details><summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.designatedMarketArea | one row per zip_code | zip_code | varchar | ZIP code for mapping | 90028 | Third-party | LINK_RAW_D_001 | Join key used against theater zip in analytics. |
| raw_import_cache.designatedMarketArea | one row per zip_code | market_name | varchar | DMA market name | Los Angeles | Third-party | LINK_RAW_D_002 | Directly selected into analytics output. |

</details>

<details><summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | theater_key | bigint | Theater key | 11872 | ETL | LINK_STG_D_001 | Preserves path to theater zip used in analytics DMA enrichment join. |

</details>

<details><summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_theater | one row per theater_key | zip_code | varchar | Theater zip code | 90028 | Backend | LINK_PUB_D_001 | Zip join bridge to raw designatedMarketArea in analytics. |

</details>

<details open><summary><strong>Analytics (Normalized: 1 row)</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | dma_market_name | varchar | DMA market name | Los Angeles | Third-party | LINK_AN_D_001 | Final value selected from raw_import_cache.designatedMarketArea using theater zip join in analytics SQL. |

</details>
</details>

<details>
<summary><strong>Group E — Post-Insert Return Refresh</strong></summary>

### Flow Diagram (Initial insert plus return-window refresh)
```text
Raw (return-related raw feeds)
-> Staging (warehouse_loader.fact_transaction_base)
-> Public (public.fact_transaction, public.fact_return)
-> Analytics (analytics.transactions_data_source)
```

<details><summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw return-related sources | one row per return event | return event timestamp + monetary components | mixed | Return lifecycle and amounts | 2026-10-02 ... | Backend | LINK_RAW_E_001 | Raw return events flow into return/public facts that drive analytics refresh logic. |

</details>

<details><summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | return-sensitive amount measures | numeric(19,4) | Returned amount metrics | 0.00 | Derived | LINK_STG_E_001 | Return-linked transaction amount lineage propagated toward public layer. |

</details>

<details><summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_return | one row per return record | return_start_datetime / purchase_transaction_guid | timestamp/varchar | Return window + purchase mapping | 2026-10-02 / TXN-123 | Backend | LINK_PUB_E_001 | Analytics update identifies impacted purchase transactions using return-window filter. |
| public.fact_transaction | one row per transaction_guid | returned_amount / returned_fee_amount / returned_charity_amount | numeric(19,4) | Latest returned metrics | -42.50 / -5.00 / -1.00 | Derived | LINK_PUB_E_002 | Source-of-truth values used in analytics post-insert refresh update. |

</details>

<details open><summary><strong>Analytics (Normalized: 3 rows)</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount after refresh | -42.50 | Derived | LINK_AN_E_001 | Inserted from public.fact_transaction and then refreshed in post-insert UPDATE for return-window transactions. |
| analytics.transactions_data_source | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount after refresh | -5.00 | Derived | LINK_AN_E_002 | Same refresh pattern as returned_amount. |
| analytics.transactions_data_source | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount after refresh | -1.00 | Derived | LINK_AN_E_003 | Same refresh pattern as returned_amount. |

</details>
</details>

---

## Quick Crosswalk (business-friendly)

- **Group A (Direct Pass-Through):**  
  Value is already prepared upstream and projected directly from `public.fact_transaction` (rename-only where needed).

- **Group B (Derived in Analytics):**  
  Value is calculated in analytics from public-layer fields (timezone conversion, day-level deltas, CASE bucketing).

- **Group C (Dimension-Sourced):**  
  Raw IDs are mapped to conformed keys upstream; analytics selects descriptive attributes from `public.dim_*`.

- **Group D (Raw-Enriched in Analytics):**  
  Core pipeline value gets an extra analytics-time enrichment from a targeted raw lookup join (`designatedMarketArea`).

- **Group E (Post-Insert Return Refresh):**  
  Value is inserted via standard flow, then selectively re-synced in a return-window update step.

---

## Analytics Field Coverage Check

- Expected analytics columns: **128**
- Documented analytics rows (one-field-per-row): **128**  
  - Group A: 48  
  - Group B: 11  
  - Group C: 65  
  - Group D: 1  
  - Group E: 3

> Replace all `LINK_*` placeholders with final GitHub permalinks once line anchors are finalized.
