# analytics.transactions_data_source — Flow-Pattern Lineage (Raw → Staging → Public → Analytics)

## Purpose

This version groups by **flow pattern**, while preserving the same four layer categories in every group:

1. Raw
2. Staging
3. Public
4. Analytics

Each table uses exactly these columns:

- Redshift Table
- Table Grain
- Column Name
- Data Type
- Description
- Example Value
- Source
- GitHub Link
- Logic Explanation

---

## Group A — Direct Pass-Through from `public.fact_transaction`

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Transaction_GUID | varchar | Transaction identifier | TXN-123 | Backend | LINK_RAW_A_001 | Extracted from `tblTransaction.Transaction_GUID`; foundational key for all downstream joins. |
| raw_import_cache.transaction | one row per transaction_guid | Create_dm | timestamp | Transaction start timestamp | 2026-10-01 16:10:00 | Backend | LINK_RAW_A_002 | Feeds staging `transaction_start_datetime`. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Create_dm | timestamp | Commit timestamp | 2026-10-01 16:11:05 | Backend | LINK_RAW_A_003 | Feeds staging `transaction_commit_datetime`. |
| raw_import_cache.transactionRolledBack | one row per rolled-back transaction_guid | Create_dm | timestamp | Rollback timestamp | NULL | Backend | LINK_RAW_A_004 | Feeds staging `transaction_rollback_datetime`. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Confirmation_cd | varchar | POS confirmation code | CNF908812 | Backend | LINK_RAW_A_005 | Feeds `point_of_sale_confirmation_code`. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | ReservedSeats_tx | varchar | Reserved seat text | A10,A11 | Backend | LINK_RAW_A_006 | Feeds `is_reserved_seating` and `reserved_seats`. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Loyalty_cd | varchar | Loyalty code | LOY123 | Backend | LINK_RAW_A_007 | Trimmed in staging to `loyalty_code`. |
| raw_import_cache.transaction | one row per transaction_guid | DeviceType_cd | varchar | Device type | Mobile | Backend | LINK_RAW_A_008 | Passes to `deviceType`. |
| raw_import_cache.transactionAttribute | one row per transaction_guid + attribute | value_tx (`CharityName`) | varchar | Charity name | Save The Arts | Backend | LINK_RAW_A_009 | Extracted via attribute filter into staging `charity_name`. |
| raw_import_cache.transactionAttribute | one row per transaction_guid + attribute | value_tx (`PartnerDefinedData`) | varchar | Partner-defined value | campaign=fall | Backend | LINK_RAW_A_010 | Feeds `partner_defined_data_value`. |
| raw_import_cache.transactionAttribute | one row per transaction_guid + attribute | value_tx (`IsBuyNowOrder`) | varchar | Buy-now marker | true | Backend | LINK_RAW_A_011 | CASE-derived to boolean in staging. |
| raw_import_cache.transactionAttribute | one row per transaction_guid + attribute | value_tx (`NetworkToken`) | varchar | Network token marker | true | Backend | LINK_RAW_A_012 | CASE-derived to `network_token_usage`. |
| raw_import_cache.transactionPerformanceAttribute | one row per transaction-performance | Performance_dm | timestamp | Local performance datetime | 2026-10-05 19:30:00 | Backend | LINK_RAW_A_013 | Feeds `performance_datetime_local`. |
| raw_import_cache.transactionItem | one row per transaction item | Item_qy / Item_at / Fee/Tax fields | numeric/int | Item and amount components | 2 / 35.00 | Backend | LINK_RAW_A_014 | Upstream components for public financial rollups (gross/fees/returns). |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_guid | varchar | Transaction identifier | TXN-123 | Backend | LINK_STG_A_001 | Direct from `raw_import_cache.transaction.Transaction_GUID`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_start_datetime | timestamp | UTC start datetime | 2026-10-01 16:10:00 | Backend | LINK_STG_A_002 | `t.Create_dm::TIMESTAMP`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_commit_datetime | timestamp | UTC commit datetime | 2026-10-01 16:11:05 | Backend | LINK_STG_A_003 | `tc.Create_dm`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_rollback_datetime | timestamp | UTC rollback datetime | NULL | Backend | LINK_STG_A_004 | `tr.Create_dm`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_end_datetime | timestamp | Final lifecycle timestamp | 2026-10-01 16:11:05 | Derived | LINK_STG_A_005 | CASE selects latest commit/rollback/start. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_STG_A_006 | `tc.Confirmation_cd`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating indicator | true | Derived | LINK_STG_A_007 | CASE on `ReservedSeats_tx` non-empty. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | reserved_seats | varchar | Reserved seat payload | A10,A11 | Backend | LINK_STG_A_008 | Trimmed reserved seats string. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY123 | Backend | LINK_STG_A_009 | Trimmed from `tc.Loyalty_cd`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-05 19:30:00 | Backend | LINK_STG_A_010 | `tpa.Performance_dm`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_STG_A_011 | `t.DeviceType_cd`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_test | boolean | Test transaction flag | false | Derived | LINK_STG_A_012 | Multi-condition CASE (test bit, email patterns, transaction type). |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_fanclub | boolean | Fanclub status flag | true | Derived | LINK_STG_A_013 | CASE from `FanClubStatus` attribute values. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived | LINK_STG_A_014 | CASE from `IsBuyNowOrder` attribute. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_STG_A_015 | `ta3.value_tx`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_defined_data_value | varchar | Partner payload | campaign=fall | Backend | LINK_STG_A_016 | `ta4.value_tx`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | network_token_usage | boolean | Network token usage | true | Derived | LINK_STG_A_017 | CASE lower(trim(value)) = 'true'. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_plf | boolean | PLF indicator | true | Derived | LINK_STG_A_018 | From aggregated amenity flags temp table. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_imax | boolean | IMAX indicator | false | Derived | LINK_STG_A_019 | From aggregated amenity flags temp table. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_3d | boolean | 3D indicator | false | Derived | LINK_STG_A_020 | From aggregated amenity flags temp table. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity indicator | true | Derived | LINK_STG_A_021 | From aggregated amenity flags temp table. |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | transaction_guid | varchar | Transaction identifier | TXN-123 | Backend | LINK_PUB_A_001 | Pass-through from staging key. |
| public.fact_transaction | one row per transaction_guid | is_successful | boolean | Success flag | true | Derived | LINK_PUB_A_002 | Derived upstream from commit/rollback existence. |
| public.fact_transaction | one row per transaction_guid | transaction_start_datetime | timestamp | UTC start datetime | 2026-10-01 16:10:00 | Backend | LINK_PUB_A_003 | Pass-through from staging. |
| public.fact_transaction | one row per transaction_guid | transaction_commit_datetime | timestamp | UTC commit datetime | 2026-10-01 16:11:05 | Backend | LINK_PUB_A_004 | Pass-through from staging. |
| public.fact_transaction | one row per transaction_guid | transaction_rollback_datetime | timestamp | UTC rollback datetime | NULL | Backend | LINK_PUB_A_005 | Pass-through from staging. |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime | timestamp | Final lifecycle datetime | 2026-10-01 16:11:05 | Derived | LINK_PUB_A_006 | Pass-through derived staging value. |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime_local | timestamp | Local final datetime | 2026-10-01 09:11:05 | Derived | LINK_PUB_A_007 | Localized value maintained in public. |
| public.fact_transaction | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_PUB_A_008 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived | LINK_PUB_A_009 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | reserved_seats | varchar | Reserved seats | A10,A11 | Backend | LINK_PUB_A_010 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY123 | Backend | LINK_PUB_A_011 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-05 19:30:00 | Backend | LINK_PUB_A_012 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | payment_types | varchar | Payment type list | VISA | Backend | LINK_PUB_A_013 | Maintained in public fact. |
| public.fact_transaction | one row per transaction_guid | processor_order_ids | varchar | Processor order ids | ord_123 | Third-party | LINK_PUB_A_014 | Processor-provided identifiers. |
| public.fact_transaction | one row per transaction_guid | item_count | int | Item count | 2 | Derived | LINK_PUB_A_015 | Aggregated upstream; renamed in analytics as `item_qty`. |
| public.fact_transaction | one row per transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | Backend | LINK_PUB_A_016 | Financial fact measure. |
| public.fact_transaction | one row per transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee amount | 5.00 | Backend | LINK_PUB_A_017 | Financial fact measure. |
| public.fact_transaction | one row per transaction_guid | waived_sales_fee_ticket_count | int | Waived sales fee ticket count | 0 | Derived | LINK_PUB_A_018 | Aggregated count measure. |
| public.fact_transaction | one row per transaction_guid | waived_fanclub_fee_ticket_count | int | Waived fanclub fee ticket count | 0 | Derived | LINK_PUB_A_019 | Aggregated count measure. |
| public.fact_transaction | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount | 0.00 | Derived | LINK_PUB_A_020 | Return-adjusted measure. |
| public.fact_transaction | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | 0.00 | Derived | LINK_PUB_A_021 | Return-adjusted measure. |
| public.fact_transaction | one row per transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Backend | LINK_PUB_A_022 | Financial measure. |
| public.fact_transaction | one row per transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Backend | LINK_PUB_A_023 | Financial measure. |
| public.fact_transaction | one row per transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Backend | LINK_PUB_A_024 | Financial measure. |
| public.fact_transaction | one row per transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee amount | 1.50 | Backend | LINK_PUB_A_025 | Financial measure. |
| public.fact_transaction | one row per transaction_guid | sales_fee_return_amount | numeric(19,4) | Sales fee return amount | 0.00 | Derived | LINK_PUB_A_026 | Return metric. |
| public.fact_transaction | one row per transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Surcharge fee return amount | 0.00 | Derived | LINK_PUB_A_027 | Return metric. |
| public.fact_transaction | one row per transaction_guid | reservation_fee_return_amount | numeric(19,4) | Reservation fee return amount | 0.00 | Derived | LINK_PUB_A_028 | Return metric. |
| public.fact_transaction | one row per transaction_guid | tax_return_amount | numeric(19,4) | Tax return amount | 0.00 | Derived | LINK_PUB_A_029 | Return metric. |
| public.fact_transaction | one row per transaction_guid | card_fee_return_amount | numeric(19,4) | Card fee return amount | 0.00 | Derived | LINK_PUB_A_030 | Return metric. |
| public.fact_transaction | one row per transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee for returned purchase | 0.00 | Derived | LINK_PUB_A_031 | Return attribution metric. |
| public.fact_transaction | one row per transaction_guid | POS_reference_cd | varchar | POS reference code | POSREF1 | Backend | LINK_PUB_A_032 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived | LINK_PUB_A_033 | Taxability metric. |
| public.fact_transaction | one row per transaction_guid | is_test | boolean | Test flag | false | Derived | LINK_PUB_A_034 | Pass-through from staging logic. |
| public.fact_transaction | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_PUB_A_035 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived | LINK_PUB_A_036 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Backend | LINK_PUB_A_037 | Financial measure. |
| public.fact_transaction | one row per transaction_guid | true_gross_transaction_amount | numeric(19,4) | True gross amount | 43.50 | Derived | LINK_PUB_A_038 | Derived gross metric. |
| public.fact_transaction | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | 0.00 | Derived | LINK_PUB_A_039 | Return-adjusted charity metric. |
| public.fact_transaction | one row per transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | 0.00 | Derived | LINK_PUB_A_040 | Return metric. |
| public.fact_transaction | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_PUB_A_041 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | partner_defined_data_value | varchar | Partner-defined payload | campaign=fall | Backend | LINK_PUB_A_042 | Pass-through. |
| public.fact_transaction | one row per transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived | LINK_PUB_A_043 | Pass-through from staging CASE. |
| public.fact_transaction | one row per transaction_guid | network_token_usage | boolean/varchar | Network token usage flag | true | Derived | LINK_PUB_A_044 | Pass-through from staging CASE. |
| public.fact_transaction | one row per transaction_guid | is_plf | boolean | PLF flag | true | Derived | LINK_PUB_A_045 | Pass-through from amenity aggregation. |
| public.fact_transaction | one row per transaction_guid | is_imax | boolean | IMAX flag | false | Derived | LINK_PUB_A_046 | Pass-through from amenity aggregation. |
| public.fact_transaction | one row per transaction_guid | is_3d | boolean | 3D flag | false | Derived | LINK_PUB_A_047 | Pass-through from amenity aggregation. |
| public.fact_transaction | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | true | Derived | LINK_PUB_A_048 | Pass-through from amenity aggregation. |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_guid | varchar | Transaction identifier | TXN-123 | Backend | LINK_AN_A_001 | `public.fact_transaction.transaction_guid` direct pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | is_successful | boolean | Success indicator | true | Derived | LINK_AN_A_002 | Direct pass-through from `public.fact_transaction.is_successful`. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_start_datetime_utc | timestamp | UTC start datetime | 2026-10-01 16:10:00 | Backend | LINK_AN_A_003 | Alias of `public.fact_transaction.transaction_start_datetime`. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_commit_datetime_utc | timestamp | UTC commit datetime | 2026-10-01 16:11:05 | Backend | LINK_AN_A_004 | Alias of `public.fact_transaction.transaction_commit_datetime`. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_rollback_datetime_utc | timestamp | UTC rollback datetime | NULL | Backend | LINK_AN_A_005 | Alias of `public.fact_transaction.transaction_rollback_datetime`. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_utc | timestamp | UTC final datetime | 2026-10-01 16:11:05 | Derived | LINK_AN_A_006 | Alias of upstream-derived `public.fact_transaction.transaction_end_datetime`. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_local | timestamp | Local final datetime | 2026-10-01 09:11:05 | Derived | LINK_AN_A_007 | Direct from `public.fact_transaction.transaction_end_datetime_local`. |
| analytics.transactions_data_source | one row per transaction_guid | point_of_sale_confirmation_code | varchar | POS confirmation code | CNF908812 | Backend | LINK_AN_A_008 | Pass-through from `public.fact_transaction`. |
| analytics.transactions_data_source | one row per transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived | LINK_AN_A_009 | Pass-through from upstream derived boolean. |
| analytics.transactions_data_source | one row per transaction_guid | reserved_seats | varchar | Reserved seats | A10,A11 | Backend | LINK_AN_A_010 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | loyalty_code | varchar | Loyalty code | LOY123 | Backend | LINK_AN_A_011 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-05 19:30:00 | Backend | LINK_AN_A_012 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | payment_types | varchar | Payment types | VISA | Backend | LINK_AN_A_013 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | processor_order_ids | varchar | Processor order ids | ord_123 | Third-party | LINK_AN_A_014 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | item_qty | int | Item quantity | 2 | Derived | LINK_AN_A_015 | Rename-only `public.fact_transaction.item_count AS item_qty`. |
| analytics.transactions_data_source | one row per transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | Backend | LINK_AN_A_016 | Pass-through from public fact. |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee amount | 5.00 | Backend | LINK_AN_A_017 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | waived_sales_fee_ticket_count | int | Waived sales fee ticket count | 0 | Derived | LINK_AN_A_018 | Pass-through aggregated metric. |
| analytics.transactions_data_source | one row per transaction_guid | waived_fanclub_fee_ticket_count | int | Waived fanclub fee ticket count | 0 | Derived | LINK_AN_A_019 | Pass-through aggregated metric. |
| analytics.transactions_data_source | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount | 0.00 | Derived | LINK_AN_A_020 | Pass-through at insert; also refreshed in Group E update. |
| analytics.transactions_data_source | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | 0.00 | Derived | LINK_AN_A_021 | Pass-through at insert; also refreshed in Group E update. |
| analytics.transactions_data_source | one row per transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Backend | LINK_AN_A_022 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Backend | LINK_AN_A_023 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Backend | LINK_AN_A_024 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee amount | 1.50 | Backend | LINK_AN_A_025 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_return_amount | numeric(19,4) | Sales fee return amount | 0.00 | Derived | LINK_AN_A_026 | Pass-through return metric. |
| analytics.transactions_data_source | one row per transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Surcharge fee return amount | 0.00 | Derived | LINK_AN_A_027 | Pass-through return metric. |
| analytics.transactions_data_source | one row per transaction_guid | reservation_fee_return_amount | numeric(19,4) | Reservation fee return amount | 0.00 | Derived | LINK_AN_A_028 | Pass-through return metric. |
| analytics.transactions_data_source | one row per transaction_guid | tax_return_amount | numeric(19,4) | Tax return amount | 0.00 | Derived | LINK_AN_A_029 | Pass-through return metric. |
| analytics.transactions_data_source | one row per transaction_guid | card_fee_return_amount | numeric(19,4) | Card fee return amount | 0.00 | Derived | LINK_AN_A_030 | Pass-through return metric. |
| analytics.transactions_data_source | one row per transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee for returned purchase | 0.00 | Derived | LINK_AN_A_031 | Pass-through return attribution metric. |
| analytics.transactions_data_source | one row per transaction_guid | POS_reference_cd | varchar | POS reference code | POSREF1 | Backend | LINK_AN_A_032 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived | LINK_AN_A_033 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | is_test | boolean | Test indicator | false | Derived | LINK_AN_A_034 | Pass-through from staging CASE logic. |
| analytics.transactions_data_source | one row per transaction_guid | deviceType | varchar | Device type | Mobile | Backend | LINK_AN_A_035 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived | LINK_AN_A_036 | Pass-through from staging CASE logic. |
| analytics.transactions_data_source | one row per transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Backend | LINK_AN_A_037 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | true_gross_transaction_amount | numeric(19,4) | True gross amount | 43.50 | Derived | LINK_AN_A_038 | Pass-through upstream derived metric. |
| analytics.transactions_data_source | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | 0.00 | Derived | LINK_AN_A_039 | Pass-through at insert; also refreshed in Group E. |
| analytics.transactions_data_source | one row per transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | 0.00 | Derived | LINK_AN_A_040 | Pass-through return metric. |
| analytics.transactions_data_source | one row per transaction_guid | charity_name | varchar | Charity name | Save The Arts | Backend | LINK_AN_A_041 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | partner_defined_data_value | varchar | Partner-defined payload | campaign=fall | Backend | LINK_AN_A_042 | Pass-through. |
| analytics.transactions_data_source | one row per transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived | LINK_AN_A_043 | Pass-through from staging CASE logic. |
| analytics.transactions_data_source | one row per transaction_guid | network_token_usage | boolean/varchar | Network token usage | true | Derived | LINK_AN_A_044 | Pass-through from staging CASE logic. |
| analytics.transactions_data_source | one row per transaction_guid | is_plf | boolean | PLF flag | true | Derived | LINK_AN_A_045 | Pass-through from staging amenity rollup logic. |
| analytics.transactions_data_source | one row per transaction_guid | is_imax | boolean | IMAX flag | false | Derived | LINK_AN_A_046 | Pass-through from staging amenity rollup logic. |
| analytics.transactions_data_source | one row per transaction_guid | is_3d | boolean | 3D flag | false | Derived | LINK_AN_A_047 | Pass-through from staging amenity rollup logic. |
| analytics.transactions_data_source | one row per transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | true | Derived | LINK_AN_A_048 | Pass-through from staging amenity rollup logic. |

---

## Group B — Derived in Analytics (Timezone / Date Deltas / CASE Buckets)

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Create_dm | timestamp | Raw transaction creation datetime | 2026-10-01 16:10:00 | Backend | LINK_RAW_B_001 | Feeds upstream `public.fact_transaction.transaction_*` datetimes used in analytics timezone conversions. |
| raw_import_cache.transactionPerformanceAttribute | one row per transaction-performance | Performance_dm | timestamp | Local performance datetime | 2026-10-05 19:30:00 | Backend | LINK_RAW_B_002 | Feeds `public.fact_transaction.performance_datetime_local` used in date-delta calculations. |
| raw_import_cache.transaction | one row per transaction_guid | Affiliate_id | int | Affiliate id | 11879 | Backend | LINK_RAW_B_003 | Drives CASE logic for `Affiliate Grouping`, `HIGH LEVEL BINS`, `apps`, and part of `platform`. |
| raw_import_cache.transaction | one row per transaction_guid | Channel_cd | varchar | Channel code | WAP | Backend | LINK_RAW_B_004 | Drives CASE logic for `platform` (via dim channel resolution). |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_start_datetime / transaction_commit_datetime / transaction_rollback_datetime / transaction_end_datetime | timestamps | Core UTC lifecycle timestamps | 2026-10-01 ... | Backend/Derived | LINK_STG_B_001 | Passed to public and then converted to Pacific in analytics. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-05 19:30:00 | Backend | LINK_STG_B_002 | Used in analytics date-delta calculations. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | affiliate_key / channel_key | bigint | Keys for affiliate/channel dimensions | 8123 / 291 | ETL | LINK_STG_B_003 | Enables analytics CASE logic using joined dim ids/names. |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | one row per transaction_guid | transaction_start_datetime | timestamp | UTC start datetime | 2026-10-01 16:10:00 | Backend | LINK_PUB_B_001 | Converted in analytics to Los Angeles timezone. |
| public.fact_transaction | one row per transaction_guid | transaction_commit_datetime | timestamp | UTC commit datetime | 2026-10-01 16:11:05 | Backend | LINK_PUB_B_002 | Converted in analytics. |
| public.fact_transaction | one row per transaction_guid | transaction_rollback_datetime | timestamp | UTC rollback datetime | NULL | Backend | LINK_PUB_B_003 | Converted in analytics. |
| public.fact_transaction | one row per transaction_guid | transaction_end_datetime | timestamp | UTC final datetime | 2026-10-01 16:11:05 | Derived | LINK_PUB_B_004 | Converted in analytics and used in date-delta logic. |
| public.fact_transaction | one row per transaction_guid | performance_datetime_local | timestamp | Local showtime datetime | 2026-10-05 19:30:00 | Backend | LINK_PUB_B_005 | Used in date-delta logic. |
| public.dim_movie | one row per movie_key | movie_release_date | date | Movie release date | 2026-09-14 | Backend | LINK_PUB_B_006 | Used in all delta date calculations. |
| public.dim_affiliate | one row per affiliate_key | affiliate_id | int | Affiliate id | 11879 | Backend | LINK_PUB_B_007 | Input to analytics CASE bucket logic. |
| public.dim_channel | one row per channel_key | channel_name | varchar | Channel name | WAP | Backend | LINK_PUB_B_008 | Input to platform CASE logic. |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | transaction_start_datetime_los_angeles | timestamp | Pacific start datetime | 2026-10-01 09:10:00 | Derived | LINK_AN_B_001 | `CONVERT_TIMEZONE('US/Pacific', public.fact_transaction.transaction_start_datetime)`. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_commit_datetime_los_angeles | timestamp | Pacific commit datetime | 2026-10-01 09:11:05 | Derived | LINK_AN_B_002 | Timezone conversion in analytics SELECT. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_rollback_datetime_los_angeles | timestamp | Pacific rollback datetime | NULL | Derived | LINK_AN_B_003 | Timezone conversion in analytics SELECT. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_end_datetime_los_angeles | timestamp | Pacific final datetime | 2026-10-01 09:11:05 | Derived | LINK_AN_B_004 | Timezone conversion in analytics SELECT. |
| analytics.transactions_data_source | one row per transaction_guid | deltareleasetopurchasedate | int | Days release-to-purchase | 17 | Derived | LINK_AN_B_005 | `DATEDIFF(day, movie_release_date, DATE_TRUNC('day', Pacific transaction_end_datetime))`. |
| analytics.transactions_data_source | one row per transaction_guid | deltareleasetoshowtimedate | int | Days release-to-showtime | 21 | Derived | LINK_AN_B_006 | `DATEDIFF(day, movie_release_date, DATE_TRUNC('day', performance_datetime_local))`. |
| analytics.transactions_data_source | one row per transaction_guid | deltapurchasetoshowtimedate | int | Days purchase-to-showtime | 4 | Derived | LINK_AN_B_007 | `DATEDIFF(day, DATE_TRUNC('day', Pacific transaction_end_datetime), DATE_TRUNC('day', performance_datetime_local))`. |
| analytics.transactions_data_source | one row per transaction_guid | Affiliate Grouping | varchar | Detailed affiliate grouping | iPhone App | Derived | LINK_AN_B_008 | Large CASE by `affiliate_id`. |
| analytics.transactions_data_source | one row per transaction_guid | Affiliate Grouping - HIGH LEVEL BINS | varchar | High-level grouping | Fandango O&O | Derived | LINK_AN_B_009 | CASE by `affiliate_id`. |
| analytics.transactions_data_source | one row per transaction_guid | apps | varchar | App grouping | Fandango App: iPhone | Derived | LINK_AN_B_010 | CASE by `affiliate_id`. |
| analytics.transactions_data_source | one row per transaction_guid | platform | varchar | Platform grouping | Mobile Apps | Derived | LINK_AN_B_011 | CASE by `channel_name` + `affiliate_id`. |

---

## Group C — Dimension-Sourced Fields (Joined via `public.fact_transaction` Keys)

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | one row per transaction_guid | Affiliate_id | int | Affiliate identifier | 11879 | Backend | LINK_RAW_C_001 | Mapped to `affiliate_key` in staging, then to `public.dim_affiliate`. |
| raw_import_cache.transaction | one row per transaction_guid | Channel_cd | varchar | Channel code | WAP | Backend | LINK_RAW_C_002 | Mapped to `channel_key`, then `public.dim_channel`. |
| raw_import_cache.transactionPerformanceAttribute | one row per transaction-performance | Movie_detail_id | int | Movie id | 10293 | Backend | LINK_RAW_C_003 | Mapped to `movie_key`, then `public.dim_movie`. |
| raw_import_cache.transactionPerformanceAttribute | one row per transaction-performance | Theater_pkey | int | Theater id | 4451 | Backend | LINK_RAW_C_004 | Mapped to `theater_key`/`chain_key`, then theater/chain dims. |
| raw_import_cache.transaction | one row per transaction_guid | Transaction_tp | varchar | Transaction type code | Purchase | Backend | LINK_RAW_C_005 | Mapped to `transaction_type_key`, then dim transaction type. |
| raw_import_cache.transactionCommitted | one row per committed transaction_guid | Confirmation_cd | varchar | Confirmation code | REF123 | Backend | LINK_RAW_C_006 | Prefix maps to return type dimension. |
| raw_import_cache.transaction | one row per transaction_guid | UserAgent_nm | varchar | User agent value | Mozilla/... | Backend | LINK_RAW_C_007 | Mapped to `user_agent_key`. |
| raw_import_cache.transactionPurchaseFunnel | one row per transaction_guid + funnel | Funnel_cd | varchar | Funnel code | 14 | Backend | LINK_RAW_C_008 | Mapped to `purchase_funnel_key`. |
| raw_import_cache.transaction | one row per transaction_guid | Property_nm | varchar | Property name | Fandango | Backend | LINK_RAW_C_009 | Normalized and mapped to `property_key`. |
| raw_import_cache.transactionAttribute | one row per transaction_guid + attribute | value_tx (`partner-channel`) | varchar | Partner channel text | TMO | Third-party | LINK_RAW_C_010 | Mapped to `partner_channel_key`. |
| raw_import_cache.transactionAttribute | one row per transaction_guid + attribute | value_tx (`partner-platform`) | varchar | Partner platform text | WEBVIEW | Third-party | LINK_RAW_C_011 | Mapped to `partner_platform_key`. |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | affiliate_key | bigint | Affiliate surrogate key | 8123 | ETL | LINK_STG_C_001 | From join `dim_affiliate.affiliate_id = raw Affiliate_id`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | channel_key | bigint | Channel surrogate key | 291 | ETL | LINK_STG_C_002 | From join `dim_channel.channel_id = raw Channel_cd`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | movie_key | bigint | Movie surrogate key | 129384 | ETL | LINK_STG_C_003 | From join `dim_movie.movie_id = raw Movie_detail_id`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | chain_key | bigint | Chain surrogate key | 392 | ETL | LINK_STG_C_004 | Derived via theater join / fallback coalesce logic. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | theater_key | bigint | Theater surrogate key | 11872 | ETL | LINK_STG_C_005 | From theater effective-dated join on raw theater id. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | transaction_type_key | bigint | Transaction type key | 82 | ETL | LINK_STG_C_006 | From join to `dim_transaction_type`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | return_type_key | bigint | Return type key | 16 | ETL | LINK_STG_C_007 | From confirmation prefix mapping to `dim_return_type`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | user_agent_key | bigint | User agent key | 298771 | ETL | LINK_STG_C_008 | From join to `dim_user_agent`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | purchase_funnel_key | bigint | Purchase funnel key | 7231 | ETL | LINK_STG_C_009 | From join to `dim_purchase_funnel`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | property_key | bigint | Property key | 45 | ETL | LINK_STG_C_010 | From normalized property join to `dim_property`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_channel_key | bigint | Partner channel key | 912 | ETL | LINK_STG_C_011 | From attribute value to `dim_partner_channel`. |
| warehouse_loader.fact_transaction_base | one row per transaction_guid | partner_platform_key | bigint | Partner platform key | 117 | ETL | LINK_STG_C_012 | From attribute value to `dim_partner_platform`. |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_affiliate | one row per affiliate_key | affiliate_name | varchar | Affiliate name | Fandango iOS App | Derived | LINK_PUB_C_001 | Selected in analytics via affiliate join. |
| public.dim_affiliate | one row per affiliate_key | affiliate_group_name | varchar | Affiliate group name | Mobile App | Derived | LINK_PUB_C_002 | Selected in analytics via affiliate join. |
| public.dim_affiliate | one row per affiliate_key | affiliate_category_name | varchar | Affiliate category name | Owned & Operated | Derived | LINK_PUB_C_003 | Selected in analytics via affiliate join. |
| public.dim_affiliate | one row per affiliate_key | affiliate_id | int | Affiliate id | 11879 | Backend | LINK_PUB_C_004 | Selected in analytics via affiliate join. |
| public.dim_channel | one row per channel_key | channel_group | varchar | Channel group | App | Derived | LINK_PUB_C_005 | Selected as analytics `channel`. |
| public.dim_channel | one row per channel_key | channel_name | varchar | Channel name | WAP | Backend | LINK_PUB_C_006 | Selected as analytics `channel_ungrouped`. |
| public.dim_movie | one row per movie_key | movie_id | int | Child movie id | 10293 | Backend | LINK_PUB_C_007 | Selected as `filmid_child`. |
| public.dim_movie | one row per movie_key | movie_title | varchar | Child movie title | Example Movie | Derived | LINK_PUB_C_008 | Selected as `filmname_child`. |
| public.dim_movie | one row per movie_key | movie_parent_id | int | Parent movie id | 872 | Derived | LINK_PUB_C_009 | Selected as `filmid_parent`. |
| public.dim_movie | one row per movie_key | movie_parent_title | varchar | Parent movie title | Example Franchise | Derived | LINK_PUB_C_010 | Selected as `filmname_parent`. |
| public.dim_movie | one row per movie_key | movie_release_date | date | Release date | 2026-09-14 | Backend | LINK_PUB_C_011 | Used directly and in date delta logic. |
| public.dim_movie | one row per movie_key | movie_tms_id | varchar | TMS id | MV123456 | Third-party | LINK_PUB_C_012 | Selected as `movie_tms_id`. |
| public.dim_chain | one row per chain_key | chain_name | varchar | Chain name | Example Cinemas | Derived | LINK_PUB_C_013 | Selected as `chainname`. |
| public.dim_chain | one row per chain_key | chain_id | varchar | Chain id | CHN019 | Backend | LINK_PUB_C_014 | Selected as `chainid`. |
| public.dim_chain | one row per chain_key | point_of_sale_vendor | varchar | POS vendor | Vista | Third-party | LINK_PUB_C_015 | Selected as `pos_vendor`. |
| public.dim_chain | one row per chain_key | awards_program_name | varchar | Awards program | Rewards+ | Derived | LINK_PUB_C_016 | Selected as `awards_program_name`. |
| public.dim_theater | one row per theater_key | theater_id | int | Theater id | 4451 | Backend | LINK_PUB_C_017 | Selected as `theater_id`. |
| public.dim_theater | one row per theater_key | theater_code | varchar | Theater code | TH4451 | Derived | LINK_PUB_C_018 | Selected as `theater_code`. |
| public.dim_theater | one row per theater_key | theater_name | varchar | Theater name | Example Hollywood | Derived | LINK_PUB_C_019 | Selected as `theater_name`. |
| public.dim_theater | one row per theater_key | pdi_id | varchar | PDI id | PDI9981 | Backend | LINK_PUB_C_020 | Selected as `pdi_id`. |
| public.dim_theater | one row per theater_key | point_of_sale_theater_id | varchar | POS theater id | POS-4451 | Backend | LINK_PUB_C_021 | Selected as `pos_theater_id`. |
| public.dim_theater | one row per theater_key | latitude | numeric | Latitude | 34.10 | Backend | LINK_PUB_C_022 | Selected as `latitude`. |
| public.dim_theater | one row per theater_key | longitude | numeric | Longitude | -118.34 | Backend | LINK_PUB_C_023 | Selected as `longitude`. |
| public.dim_theater | one row per theater_key | point_of_sale_id | varchar | POS id | POS01 | Backend | LINK_PUB_C_024 | Selected as `point_of_sale_id`. |
| public.dim_theater | one row per theater_key | address_1 | varchar | Address line 1 | 123 Main St | Backend | LINK_PUB_C_025 | Selected as `address_1`. |
| public.dim_theater | one row per theater_key | address_2 | varchar | Address line 2 | Suite 200 | Backend | LINK_PUB_C_026 | Selected as `address_2`. |
| public.dim_theater | one row per theater_key | city_name | varchar | City | Los Angeles | Backend | LINK_PUB_C_027 | Selected as `city_name`. |
| public.dim_theater | one row per theater_key | state_code | varchar | State | CA | Backend | LINK_PUB_C_028 | Selected as `state_code`. |
| public.dim_theater | one row per theater_key | zip_code | varchar | ZIP | 90028 | Backend | LINK_PUB_C_029 | Selected as `zip_code`. |
| public.dim_theater | one row per theater_key | country_code | varchar | Country | US | Backend | LINK_PUB_C_030 | Selected as `country_code`. |
| public.dim_theater | one row per theater_key | alternate_theater_phone_name | varchar | Alternate phone/name | Main Box Office | Derived | LINK_PUB_C_031 | Selected as `tht_alternate_theater_phone_name`. |
| public.dim_theater | one row per theater_key | chain_specific_site_id | varchar | Chain site id | SITE-22 | Backend | LINK_PUB_C_032 | Selected as `chain_specific_site_id`. |
| public.dim_theater | one row per theater_key | paymentech_terminal_id | varchar | Paymentech terminal id | PT-100 | Third-party | LINK_PUB_C_033 | Selected as `paymentech_terminal_id`. |
| public.dim_theater | one row per theater_key | cybersource_terminal_id | varchar | Cybersource terminal id | CS-200 | Third-party | LINK_PUB_C_034 | Selected as `cybersource_terminal_id`. |
| public.dim_theater | one row per theater_key | screen_count | int | Screen count | 12 | Backend | LINK_PUB_C_035 | Selected as `screen_count`. |
| public.dim_theater | one row per theater_key | seat_count | int | Seat count | 1800 | Backend | LINK_PUB_C_036 | Selected as `seat_count`. |
| public.dim_theater | one row per theater_key | theater_time_zone_name | varchar | Time zone | America/Los_Angeles | Backend | LINK_PUB_C_037 | Selected as `theater_time_zone_name`. |
| public.dim_theater | one row per theater_key | braintree_merchant_id | varchar | Braintree merchant id | bt_merch_1 | Third-party | LINK_PUB_C_038 | Selected as `braintree_merchant_id`. |
| public.dim_transaction_type | one row per transaction_type_key | transaction_type_id | varchar | Transaction type id | Purchase | Backend | LINK_PUB_C_039 | Selected as `transaction_type_id`. |
| public.dim_transaction_type | one row per transaction_type_key | transaction_type_name | varchar | Transaction type name | Ticket Purchase | Derived | LINK_PUB_C_040 | Selected as `transaction_type_name`. |
| public.dim_user_agent | one row per user_agent_key | user_agent_id | varchar | User agent id | UA-1 | Backend | LINK_PUB_C_041 | Selected as `user_agent_id`. |
| public.dim_user_agent | one row per user_agent_key | user_agent_description | varchar | User agent description | iOS Safari | Derived | LINK_PUB_C_042 | Selected as `user_agent_description`. |
| public.dim_purchase_funnel | one row per purchase_funnel_key | purchase_funnel_id | varchar | Funnel id | 14 | Backend | LINK_PUB_C_043 | Selected as `purchase_funnel_id`. |
| public.dim_purchase_funnel | one row per purchase_funnel_key | purchase_funnel_name | varchar | Funnel name | Guest Checkout | Derived | LINK_PUB_C_044 | Selected as `purchase_funnel_name`. |
| public.dim_return_type | one row per return_type_key | return_method | varchar | Return method | Self-service | Derived | LINK_PUB_C_045 | Selected as `return_method`. |
| public.dim_return_type | one row per return_type_key | return_type_code | varchar | Return code | REF | Backend | LINK_PUB_C_046 | Selected as `return_type_code`. |
| public.dim_return_type | one row per return_type_key | return_type_description | varchar | Return description | Customer refund | Derived | LINK_PUB_C_047 | Selected as `return_type_description`. |
| public.dim_property | one row per property_key | property_id | varchar | Property id | Fandango USA | Backend | LINK_PUB_C_048 | Selected as `property_id`. |
| public.dim_property | one row per property_key | property_group | varchar | Property group | Fandango | Derived | LINK_PUB_C_049 | Selected as `property_group`. |
| public.dim_partner_channel | one row per partner_channel_key | partner_channel_id | varchar | Partner channel id | TMO | Third-party | LINK_PUB_C_050 | Selected as `partner_channel`. |
| public.dim_partner_channel | one row per partner_channel_key | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party | LINK_PUB_C_051 | Selected as `partner_display_name`. |
| public.dim_partner_platform | one row per partner_platform_key | partner_platform_id | varchar | Partner platform id | WEBVIEW | Third-party | LINK_PUB_C_052 | Selected as `partner_platform`. |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | affiliate_name | varchar | Affiliate name | Fandango iOS App | Derived | LINK_AN_C_001 | Selected from `public.dim_affiliate.affiliate_name`. |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_group_name | varchar | Affiliate group name | Mobile App | Derived | LINK_AN_C_002 | Selected from affiliate dimension. |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_category_name | varchar | Affiliate category | Owned & Operated | Derived | LINK_AN_C_003 | Selected from affiliate dimension. |
| analytics.transactions_data_source | one row per transaction_guid | affiliate_id | int | Affiliate id | 11879 | Backend | LINK_AN_C_004 | Selected from affiliate dimension id. |
| analytics.transactions_data_source | one row per transaction_guid | channel | varchar | Channel group | App | Derived | LINK_AN_C_005 | Selected from `public.dim_channel.channel_group`. |
| analytics.transactions_data_source | one row per transaction_guid | channel_ungrouped | varchar | Raw channel bucket | WAP | Backend | LINK_AN_C_006 | Selected from `public.dim_channel.channel_name`. |
| analytics.transactions_data_source | one row per transaction_guid | filmid_child | int | Child movie id | 10293 | Backend | LINK_AN_C_007 | Selected from `public.dim_movie.movie_id`. |
| analytics.transactions_data_source | one row per transaction_guid | filmname_child | varchar | Child movie title | Example Movie | Derived | LINK_AN_C_008 | Selected from `public.dim_movie.movie_title`. |
| analytics.transactions_data_source | one row per transaction_guid | filmid_parent | int | Parent movie id | 872 | Derived | LINK_AN_C_009 | Selected from `public.dim_movie.movie_parent_id`. |
| analytics.transactions_data_source | one row per transaction_guid | filmname_parent | varchar | Parent movie title | Example Franchise | Derived | LINK_AN_C_010 | Selected from `public.dim_movie.movie_parent_title`. |
| analytics.transactions_data_source | one row per transaction_guid | releasedate | date | Release date | 2026-09-14 | Backend | LINK_AN_C_011 | Selected from `public.dim_movie.movie_release_date`. |
| analytics.transactions_data_source | one row per transaction_guid | movie_tms_id | varchar | TMS id | MV123456 | Third-party | LINK_AN_C_012 | Selected from `public.dim_movie.movie_tms_id`. |
| analytics.transactions_data_source | one row per transaction_guid | chainname | varchar | Chain name | Example Cinemas | Derived | LINK_AN_C_013 | Selected from `public.dim_chain.chain_name`. |
| analytics.transactions_data_source | one row per transaction_guid | chainid | varchar | Chain id | CHN019 | Backend | LINK_AN_C_014 | Selected from `public.dim_chain.chain_id`. |
| analytics.transactions_data_source | one row per transaction_guid | pos_vendor | varchar | POS vendor | Vista | Third-party | LINK_AN_C_015 | Selected from `public.dim_chain.point_of_sale_vendor`. |
| analytics.transactions_data_source | one row per transaction_guid | awards_program_name | varchar | Awards program name | Rewards+ | Derived | LINK_AN_C_016 | Selected from `public.dim_chain.awards_program_name`. |
| analytics.transactions_data_source | one row per transaction_guid | theater_id | int | Theater id | 4451 | Backend | LINK_AN_C_017 | Selected from `public.dim_theater.theater_id`. |
| analytics.transactions_data_source | one row per transaction_guid | theater_code | varchar | Theater code | TH4451 | Derived | LINK_AN_C_018 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | theater_name | varchar | Theater name | Example Hollywood | Derived | LINK_AN_C_019 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | pdi_id | varchar | PDI id | PDI9981 | Backend | LINK_AN_C_020 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | pos_theater_id | varchar | POS theater id | POS-4451 | Backend | LINK_AN_C_021 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | latitude | numeric | Latitude | 34.10 | Backend | LINK_AN_C_022 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | longitude | numeric | Longitude | -118.34 | Backend | LINK_AN_C_023 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | point_of_sale_id | varchar | POS id | POS01 | Backend | LINK_AN_C_024 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | address_1 | varchar | Address line 1 | 123 Main St | Backend | LINK_AN_C_025 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | address_2 | varchar | Address line 2 | Suite 200 | Backend | LINK_AN_C_026 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | city_name | varchar | City | Los Angeles | Backend | LINK_AN_C_027 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | state_code | varchar | State | CA | Backend | LINK_AN_C_028 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | zip_code | varchar | ZIP | 90028 | Backend | LINK_AN_C_029 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | country_code | varchar | Country | US | Backend | LINK_AN_C_030 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | tht_alternate_theater_phone_name | varchar | Alternate phone/name | Main Box Office | Derived | LINK_AN_C_031 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | chain_specific_site_id | varchar | Chain site id | SITE-22 | Backend | LINK_AN_C_032 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | paymentech_terminal_id | varchar | Paymentech terminal id | PT-100 | Third-party | LINK_AN_C_033 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | cybersource_terminal_id | varchar | Cybersource terminal id | CS-200 | Third-party | LINK_AN_C_034 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | screen_count | int | Screen count | 12 | Backend | LINK_AN_C_035 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | seat_count | int | Seat count | 1800 | Backend | LINK_AN_C_036 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | theater_time_zone_name | varchar | Theater timezone | America/Los_Angeles | Backend | LINK_AN_C_037 | Selected from theater dimension. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_type_id | varchar | Transaction type id | Purchase | Backend | LINK_AN_C_038 | Selected from transaction type dimension. |
| analytics.transactions_data_source | one row per transaction_guid | transaction_type_name | varchar | Transaction type name | Ticket Purchase | Derived | LINK_AN_C_039 | Selected from transaction type dimension. |
| analytics.transactions_data_source | one row per transaction_guid | user_agent_id | varchar | User agent id | UA-1 | Backend | LINK_AN_C_040 | Selected from user agent dimension. |
| analytics.transactions_data_source | one row per transaction_guid | user_agent_description | varchar | User agent description | iOS Safari | Derived | LINK_AN_C_041 | Selected from user agent dimension. |
| analytics.transactions_data_source | one row per transaction_guid | purchase_funnel_id | varchar | Funnel id | 14 | Backend | LINK_AN_C_042 | Selected from purchase funnel dimension. |
| analytics.transactions_data_source | one row per transaction_guid | purchase_funnel_name | varchar | Funnel name | Guest Checkout | Derived | LINK_AN_C_043 | Selected from purchase funnel dimension. |
| analytics.transactions_data_source | one row per transaction_guid | return_method | varchar | Return method | Self-service | Derived | LINK_AN_C_044 | Selected from return type dimension. |
| analytics.transactions_data_source | one row per transaction_guid | return_type_code | varchar | Return type code | REF | Backend | LINK_AN_C_045 | Selected from return type dimension. |
| analytics.transactions_data_source | one row per transaction_guid | return_type_description | varchar | Return type description | Customer refund | Derived | LINK_AN_C_046 | Selected from return type dimension. |
| analytics.transactions_data_source | one row per transaction_guid | property_id | varchar | Property id | Fandango USA | Backend | LINK_AN_C_047 | Selected from property dimension. |
| analytics.transactions_data_source | one row per transaction_guid | property_group | varchar | Property group | Fandango | Derived | LINK_AN_C_048 | Selected from property dimension. |
| analytics.transactions_data_source | one row per transaction_guid | partner_channel | varchar | Partner channel id | TMO | Third-party | LINK_AN_C_049 | Selected from partner channel dimension. |
| analytics.transactions_data_source | one row per transaction_guid | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party | LINK_AN_C_050 | Selected from partner channel dimension. |
| analytics.transactions_data_source | one row per transaction_guid | partner_platform | varchar | Partner platform id | WEBVIEW | Third-party | LINK_AN_C_051 | Selected from partner platform dimension. |
| analytics.transactions_data_source | one row per transaction_guid | braintree_merchant_id | varchar | Braintree merchant id | bt_merch_1 | Third-party | LINK_AN_C_052 | Selected from theater dimension. |

---

## Group D — Raw Join in Analytics Query

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.designatedMarketArea | one row per zip_code | zip_code | varchar | ZIP used for DMA mapping | 90028 | Third-party | LINK_RAW_D_001 | Joined to theater zip in analytics query. |
| raw_import_cache.designatedMarketArea | one row per zip_code | market_name | varchar | DMA market name | Los Angeles | Third-party | LINK_RAW_D_002 | Selected directly into analytics. |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | theater_key | bigint | Theater key used to reach theater zip | 11872 | ETL | LINK_STG_D_001 | Enables path to `public.dim_theater.zip_code`, used in analytics raw DMA join. |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_theater | one row per theater_key | zip_code | varchar | Theater ZIP code | 90028 | Backend | LINK_PUB_D_001 | Analytics query joins raw DMA table on this ZIP. |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | dma_market_name | varchar | DMA market name | Los Angeles | Third-party | LINK_AN_D_001 | Selected from `raw_import_cache.designatedMarketArea.market_name` via `dma.zip_code = public.dim_theater.zip_code`. |

---

## Group E — Post-Insert Refresh Fields (Return Sync Update)

### Raw

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.return / return-related raw sources | one row per return event | return_start_datetime + return monetary components | timestamp/numeric | Return-triggering data | 2026-10-02 ... | Backend | LINK_RAW_E_001 | Upstream return events eventually materialize in `public.fact_return` and return amounts in `public.fact_transaction`. |

### Staging

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | one row per transaction_guid | returned_amount / returned_fee_amount / returned_charity_amount | numeric(19,4) | Return-adjusted measures | 0.00 | Derived | LINK_STG_E_001 | Upstream return-related measures propagated toward public layer. |

### Public

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_return | one row per return fact | return_start_datetime / purchase_transaction_guid | timestamp/varchar | Return facts used in analytics update filter | 2026-10-02 / TXN-123 | Backend | LINK_PUB_E_001 | Analytics update CTE filters returns in batch window and locates original purchase transaction. |
| public.fact_transaction | one row per transaction_guid | returned_amount | numeric(19,4) | Updated returned amount | -42.50 | Derived | LINK_PUB_E_002 | Source for analytics refresh SET clause. |
| public.fact_transaction | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Updated returned fee amount | -5.00 | Derived | LINK_PUB_E_003 | Source for analytics refresh SET clause. |
| public.fact_transaction | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Updated returned charity amount | -1.00 | Derived | LINK_PUB_E_004 | Source for analytics refresh SET clause. |

### Analytics

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| analytics.transactions_data_source | one row per transaction_guid | returned_amount | numeric(19,4) | Returned amount (post-refresh) | -42.50 | Derived | LINK_AN_E_001 | Initially inserted from `public.fact_transaction`, then updated when changed via post-insert UPDATE. |
| analytics.transactions_data_source | one row per transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount (post-refresh) | -5.00 | Derived | LINK_AN_E_002 | Same refresh pattern as above. |
| analytics.transactions_data_source | one row per transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount (post-refresh) | -1.00 | Derived | LINK_AN_E_003 | Same refresh pattern as above. |

---

## Notes

1. This document is flow-pattern grouped, so some fields appear in a **primary group** (A–D) and also in **Group E** when they have additional post-insert refresh behavior.
2. Source assignment follows pass-through inheritance:
   - Rename-only pass-through does **not** become ETL.
   - ETL is used for key mapping, joins, or computed transformations.
3. Replace all `LINK_*` placeholders with final permalinks once line numbers are locked.
