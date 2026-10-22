# analytics.transactions_data_source Field Lineage (Grouped by Flow Pattern)

This document maps all **128 fields** in `analytics.transactions_data_source` across:

```text
MSSQL -> raw_import_cache -> warehouse_loader -> public -> analytics
```

## Conventions

- **Transform tags** used in **Logic Explanation**:
  - `[passthrough]`
  - `[rename/cast]`
  - `[lookup]`
  - `[derived]`
  - `[aggregate]`
  - `[conditional]`
- **Consistent field IDs**: `AN_001` … `AN_128` (unique and stable; grouped tables use the canonical IDs listed below).
- Grouping is by **flow pattern**, not business domain.

---

## Coverage Summary (128 / 128)

- Group A — Staging-derived pass-through: **31**
- Group B — Aggregated commercial measures: **13**
- Group C — Return/refund enrichment and sync: **11**
- Group D — Dimension lookup projection: **52**
- Group E — Analytics-derived calcs and CASE buckets: **11**
- Group F — Raw enrichment at analytics step: **1**
- Group G — Post-insert sales-tax enrichment: **9**

---

<details open>
<summary><strong>Group A — Staging-derived pass-through (31 fields)</strong></summary>

### Flow Diagram
```text
Raw (transaction, transactionCommitted, transactionRolledBack, transactionCustomer, transactionPerformanceAttribute, transactionAttribute, transactionPurchaseFunnel, test-email lookup tables)
-> Staging (warehouse_loader.fact_transaction_base)
-> Public (public.fact_transaction)
-> Analytics (analytics.transactions_data_source)
```

<details>
<summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | 1 row / transaction_guid | Transaction_GUID | varchar(36) | Transaction identifier | 9F2A... | Backend |  | Base transaction key. |
| raw_import_cache.transaction | 1 row / transaction_guid | Create_dm | timestamp | Start timestamp | 2026-10-01 16:10:00 | Backend |  | Source for start datetime. |
| raw_import_cache.transactionCommitted | 1 row / transaction_guid | Create_dm | timestamp | Commit timestamp | 2026-10-01 16:11:05 | Backend |  | Source for commit/end logic. |
| raw_import_cache.transactionRolledBack | 1 row / transaction_guid | Create_dm | timestamp | Rollback timestamp | NULL | Backend |  | Source for rollback/end logic. |
| raw_import_cache.transactionCommitted | 1 row / transaction_guid | Confirmation_cd | varchar(40) | POS confirmation code | CNF908812 | Backend |  | Source for point_of_sale_confirmation_code. |
| raw_import_cache.transactionCommitted | 1 row / transaction_guid | ReservedSeats_tx | varchar(100) | Reserved seat payload | A10,A11 | Backend |  | Source for reserved seats outputs. |
| raw_import_cache.transactionCommitted | 1 row / transaction_guid | Loyalty_cd | varchar(40) | Loyalty code | LOY123 | Backend |  | Source for loyalty_code. |
| raw_import_cache.transactionCommitted | 1 row / transaction_guid | POS_reference_cd | varchar(40) | POS reference code | POSREF1 | Backend |  | Source for pos_reference_cd. |
| raw_import_cache.transaction | 1 row / transaction_guid | DeviceType_cd | varchar | Device type | Mobile | Backend |  | Source for devicetype. |
| raw_import_cache.transaction | 1 row / transaction_guid | test_bt | boolean | Test marker | false | Backend |  | One input to is_test; the CASE also checks customer email, test-email tables/patterns, theater PDI, and transaction type. |
| raw_import_cache.transactionCustomer + testEmailAddress + testTransactionEmailList | transaction/customer or email row | Email_nm, email_tx, activation/expiration, active/pattern flags | mixed | Test classification inputs | — | Backend |  | Used with transaction.test_bt to derive is_test. |
| raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | Performance_dm | timestamp | Performance local datetime | 2026-10-05 19:30:00 | Backend |  | Source for performance_datetime_local. |
| raw_import_cache.transaction | 1 row / transaction_guid | ticketTaxRate_rt | decimal(6,4) | PR ticket tax rate | 11.5000 | Backend |  | Source for ticket_tax_rate. |
| raw_import_cache.transaction | 1 row / transaction_guid | ticketTaxTransaction_bt | boolean | PR ticket-tax flag | true | Backend |  | Source for is_ticket_tax_transaction. |
| raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (FanClubStatus) | varchar | Fanclub status attr | Active | Backend |  | Input to is_fanclub. |
| raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (IsBuyNowOrder) | varchar | Buy-now attr | true | Backend |  | Input to is_buynow. |
| raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (CharityName) | varchar | Charity name attr | Save The Arts | Backend |  | Source for charity_name. |
| raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (PartnerDefinedData) | varchar | Partner-defined attr | campaign=fall | Third-party |  | Source for partner_defined_data_value. |
| raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (networktoken) | varchar | Network token attr | true | Backend |  | Input to network_token_usage. |
| raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (UtmSource/UtmMedium/UtmCampaign/UtmContent/UtmTerm) | varchar(255) | UTM attrs | google/cpc/... | Backend |  | Source for utm_* fields. |
| raw_import_cache.transactionPerformanceAttribute + mapping table | many-to-one tx | amenity_ids + mapped flags | varchar | Amenity rollup inputs | 3,10 | Backend/Derived |  | Inputs for is_plf/is_imax/is_3d/is_higher_fee_amenity. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction id | 9F2A... | Backend |  | `[passthrough]` |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_successful | boolean | Success flag | true | Derived |  | `[conditional]` committed and not rolled back. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_start_datetime | timestamp | Start UTC | 2026-10-01 16:10:00 | Backend |  | `[rename/cast]` |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_commit_datetime | timestamp | Commit UTC | 2026-10-01 16:11:05 | Backend |  | `[rename/cast]` |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_rollback_datetime | timestamp | Rollback UTC | NULL | Backend |  | `[rename/cast]` |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_end_datetime | timestamp | End UTC | 2026-10-01 16:11:05 | Derived |  | `[conditional]` latest of commit/rollback/start fallback. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation | CNF908812 | Backend |  | `[rename/cast]` |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived |  | `[conditional]` non-empty seats string. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | reserved_seats | varchar(100) | Reserved seats | A10,A11 | Backend |  | `[conditional]` trim empty->NULL. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code | LOY123 | Backend |  | `[conditional]` trim empty->NULL. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | performance_datetime_local | timestamp | Local performance datetime | 2026-10-05 19:30:00 | Backend |  | `[passthrough]` |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | POS_reference_cd | varchar(40) | POS ref code | POSREF1 | Backend |  | `[passthrough]` |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_test | boolean | Test flag | false | Derived |  | `[conditional]` multi-rule test classification. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | deviceType | varchar | Device type | Mobile | Backend |  | `[rename/cast]` |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived |  | `[conditional]` FanClubStatus list. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived |  | `[conditional]` IsBuyNowOrder=true. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | charity_name | varchar(256) | Charity name | Save The Arts | Backend |  | `[lookup]` from transactionAttribute. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner-defined payload | campaign=fall | Third-party |  | `[lookup]` from transactionAttribute. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | network_token_usage | boolean | Network token usage | true | Derived |  | `[conditional]` normalized true/false (with refund fallback in current DAG). |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_plf, is_imax, is_3d, is_higher_fee_amenity | boolean | Amenity flags | false | Derived |  | `[aggregate]` grouped amenity mapping rollup. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | PR ticket tax rate | 11.5000 | Backend |  | `[rename/cast]` from ticketTaxRate_rt. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | PR ticket-tax txn flag | true | Derived |  | `[conditional]` COALESCE to false. |
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_source, utm_medium, utm_campaign, utm_content, utm_term | varchar(255) | UTM params | google/cpc/... | Backend |  | `[lookup]` attribute joins. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | 1 row / transaction_guid | transaction_guid, is_successful, transaction_start_datetime, transaction_commit_datetime, transaction_rollback_datetime, transaction_end_datetime | mixed | Lifecycle fields | — | Backend/Derived |  | `[passthrough]` from base. |
| public.fact_transaction | 1 row / transaction_guid | transaction_end_datetime_local | timestamp | End local datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` timezone conversion by theater TZ. |
| public.fact_transaction | 1 row / transaction_guid | point_of_sale_confirmation_code, is_reserved_seating, reserved_seats, loyalty_code, performance_datetime_local, POS_reference_cd | mixed | Core transaction attrs | — | Backend/Derived |  | `[passthrough]` |
| public.fact_transaction | 1 row / transaction_guid | is_test, deviceType, is_fanclub, is_buynow, charity_name, partner_defined_data_value, network_token_usage | mixed | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` |
| public.fact_transaction | 1 row / transaction_guid | is_plf, is_imax, is_3d, is_higher_fee_amenity | boolean | Amenity flags | false | Derived |  | `[passthrough]` |
| public.fact_transaction | 1 row / transaction_guid | ticket_tax_rate, is_ticket_tax_transaction, utm_source, utm_medium, utm_campaign, utm_content, utm_term | mixed | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` from base. |

</details>

<details open>
<summary><strong>Analytics (31 fields)</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| AN_001 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction id | 9F2A... | Backend |  | `[passthrough]` ft.transaction_guid |
| AN_002 | analytics.transactions_data_source | 1 row / transaction_guid | is_successful | boolean | Success flag | true | Derived |  | `[passthrough]` ft.is_successful |
| AN_003 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_start_datetime_utc | timestamp | Start UTC | 2026-10-01 16:10:00 | Backend |  | `[rename/cast]` ft.transaction_start_datetime |
| AN_005 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_commit_datetime_utc | timestamp | Commit UTC | 2026-10-01 16:11:05 | Backend |  | `[rename/cast]` ft.transaction_commit_datetime |
| AN_007 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_rollback_datetime_utc | timestamp | Rollback UTC | NULL | Backend |  | `[rename/cast]` ft.transaction_rollback_datetime |
| AN_009 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_utc | timestamp | End UTC | 2026-10-01 16:11:05 | Derived |  | `[rename/cast]` ft.transaction_end_datetime |
| AN_011 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_local | timestamp | End local | 2026-10-01 09:11:05 | Derived |  | `[passthrough]` ft.transaction_end_datetime_local |
| AN_015 | analytics.transactions_data_source | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation | CNF908812 | Backend |  | `[passthrough]` |
| AN_016 | analytics.transactions_data_source | 1 row / transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived |  | `[passthrough]` |
| AN_017 | analytics.transactions_data_source | 1 row / transaction_guid | reserved_seats | varchar(100) | Reserved seats | A10,A11 | Backend |  | `[passthrough]` |
| AN_018 | analytics.transactions_data_source | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code | LOY123 | Backend |  | `[passthrough]` |
| AN_019 | analytics.transactions_data_source | 1 row / transaction_guid | performance_datetime_local | timestamp | Performance local | 2026-10-05 19:30:00 | Backend |  | `[passthrough]` |
| AN_089 | analytics.transactions_data_source | 1 row / transaction_guid | pos_reference_cd | varchar(40) | POS reference code | POSREF1 | Backend |  | `[passthrough]` |
| AN_096 | analytics.transactions_data_source | 1 row / transaction_guid | is_test | boolean | Test flag | false | Derived |  | `[passthrough]` |
| AN_097 | analytics.transactions_data_source | 1 row / transaction_guid | devicetype | varchar | Device type | Mobile | Backend |  | `[passthrough]` |
| AN_098 | analytics.transactions_data_source | 1 row / transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived |  | `[passthrough]` |
| AN_105 | analytics.transactions_data_source | 1 row / transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived |  | `[passthrough]` |
| AN_106 | analytics.transactions_data_source | 1 row / transaction_guid | charity_name | varchar(256) | Charity name | Save The Arts | Backend |  | `[passthrough]` |
| AN_107 | analytics.transactions_data_source | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner payload | campaign=fall | Third-party |  | `[passthrough]` |
| AN_111 | analytics.transactions_data_source | 1 row / transaction_guid | network_token_usage | boolean | Network token usage | true | Derived |  | `[passthrough]` |
| AN_113 | analytics.transactions_data_source | 1 row / transaction_guid | is_plf | boolean | PLF flag | false | Derived |  | `[passthrough]` |
| AN_114 | analytics.transactions_data_source | 1 row / transaction_guid | is_imax | boolean | IMAX flag | false | Derived |  | `[passthrough]` |
| AN_115 | analytics.transactions_data_source | 1 row / transaction_guid | is_3d | boolean | 3D flag | false | Derived |  | `[passthrough]` |
| AN_116 | analytics.transactions_data_source | 1 row / transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | false | Derived |  | `[passthrough]` |
| AN_122 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | PR ticket tax rate | 11.5000 | Backend |  | `[passthrough]` |
| AN_123 | analytics.transactions_data_source | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | PR tax txn flag | true | Derived |  | `[passthrough]` |
| AN_117 | analytics.transactions_data_source | 1 row / transaction_guid | utm_source | varchar(255) | UTM source | google | Backend |  | `[passthrough]` |
| AN_118 | analytics.transactions_data_source | 1 row / transaction_guid | utm_medium | varchar(255) | UTM medium | cpc | Backend |  | `[passthrough]` |
| AN_119 | analytics.transactions_data_source | 1 row / transaction_guid | utm_campaign | varchar(255) | UTM campaign | fall_promo | Backend |  | `[passthrough]` |
| AN_120 | analytics.transactions_data_source | 1 row / transaction_guid | utm_content | varchar(255) | UTM content | banner_a | Backend |  | `[passthrough]` |
| AN_121 | analytics.transactions_data_source | 1 row / transaction_guid | utm_term | varchar(255) | UTM term | movie+tickets | Backend |  | `[passthrough]` |

</details>
</details>

---

<details>
<summary><strong>Group B — Aggregated commercial measures (13 fields)</strong></summary>

### Flow Diagram
```text
Raw (raw_import_cache.transactionItem, raw_import_cache.transactionPayment)
-> Staging (warehouse_loader.fact_transaction_item_aggregate, warehouse_loader.fact_payment_aggregate)
-> Public (public.fact_transaction)
-> Analytics (analytics.transactions_data_source)
```

<details>
<summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transactionItem | 1 row / item | Item_qy, Item_at, SalesFee_at, WaivedSalesFee_at, SurchargeFee_at, Tax_at | numeric | Item-level quantities and amounts | 2 / 35.00 | Backend |  | Upstream item inputs to aggregate tables. |
| raw_import_cache.transactionPayment | 1 row / payment row | payment type/order refs/amounts | mixed | Payment-level data | VISA / ord_123 | Backend/Third-party |  | Upstream payment inputs to aggregates. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_count, item_sales_amount, item_fees_amount, item_surcharge_fee_amount, item_waived_sales_fee_amount, fanclub_waived_sales_fee_amount, fee_waiver_item_count, fanclub_fee_waiver_item_count, item_taxable_sales_fee_amount, charity_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` consumed by factTransaction.sql. |
| warehouse_loader.fact_payment_aggregate | 1 row / transaction_guid | payment_types, processor_order_ids, gross_transaction_amount | mixed | Payment aggregates at tx grain | — | Derived/Third-party |  | `[aggregate]` consumed by factTransaction.sql. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | 1 row / transaction_guid | item_count | int | Item quantity | 2 | Derived |  | `[rename/cast]` from ftia.item_count. |
| public.fact_transaction | 1 row / transaction_guid | payment_types, processor_order_ids | varchar | Payment labels and processor refs | VISA / ord_123 | Derived/Third-party |  | `[passthrough]` from fpa. |
| public.fact_transaction | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross minus charity | 42.50 | Derived |  | `[derived]` fpa.gross - coalesce(ftia.charity_amount,0). |
| public.fact_transaction | 1 row / transaction_guid | sales_fee_amount, surcharge_fee_amount, waived_sales_fee_amount, waived_fanclub_fee_amount, waived_sales_fee_ticket_count, waived_fanclub_fee_ticket_count, ticket_sales_amount, taxable_sales_fee_amount, charity_amount, true_gross_transaction_amount | numeric(19,4) | Aggregated commercial metrics | — | Derived |  | `[rename/cast]` from aggregate outputs. |

</details>

<details open>
<summary><strong>Analytics (13 fields)</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| AN_020 | analytics.transactions_data_source | 1 row / transaction_guid | payment_types | varchar(200) | Payment types | VISA | Derived |  | `[passthrough]` ft.payment_types |
| AN_021 | analytics.transactions_data_source | 1 row / transaction_guid | processor_order_ids | varchar(124) | Processor order ids | ord_123 | Third-party |  | `[passthrough]` |
| AN_073 | analytics.transactions_data_source | 1 row / transaction_guid | item_qty | int | Item qty | 2 | Derived |  | `[rename/cast]` ft.item_count AS item_qty |
| AN_074 | analytics.transactions_data_source | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | Derived |  | `[passthrough]` |
| AN_075 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee | 5.00 | Derived |  | `[passthrough]` |
| AN_076 | analytics.transactions_data_source | 1 row / transaction_guid | waived_sales_fee_ticket_count | numeric(19,4) | Waived sales fee ticket count | 0 | Derived |  | `[passthrough]` |
| AN_100 | analytics.transactions_data_source | 1 row / transaction_guid | waived_fanclub_fee_ticket_count | numeric(19,4) | Waived fanclub fee ticket count | 0 | Derived |  | `[passthrough]` |
| AN_079 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Derived |  | `[passthrough]` |
| AN_080 | analytics.transactions_data_source | 1 row / transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Derived |  | `[passthrough]` |
| AN_099 | analytics.transactions_data_source | 1 row / transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Derived |  | `[passthrough]` |
| AN_081 | analytics.transactions_data_source | 1 row / transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee | 1.50 | Derived |  | `[passthrough]` |
| AN_101 | analytics.transactions_data_source | 1 row / transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Derived |  | `[passthrough]` |
| AN_102 | analytics.transactions_data_source | 1 row / transaction_guid | true_gross_transaction_amount | numeric(19,4) | Gross including charity | 43.50 | Derived |  | `[passthrough]` |

</details>
</details>

---

<details>
<summary><strong>Group C — Return/refund enrichment and sync (11 fields)</strong></summary>

### Flow Diagram
```text
Raw (raw_import_cache.refund)
-> Staging (public.fact_return, warehouse_loader.fact_return_aggregate, #fact_return_lookup temp)
-> Public (public.fact_transaction UPDATEs)
-> Analytics (insert + returned_* sync update)
```

<details>
<summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.refund | payment row, grouped by refund and purchase GUID | Price_at, Fee_at, Tax_at, Surcharge_at, ReservedFee_at, CardFee_at, taxable_at, donation_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund rows into public.fact_return; downstream aggregates and refund-row updates use those values. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_return_aggregate | 1 row / purchase_transaction_guid | returned_ticket_amount, returned_fee_amount, returned_charity_amount | numeric | Purchase-side returned rollups | — | Derived |  | `[aggregate]` source for returned_* on purchase rows. |
| #fact_return_lookup (temp) | 1 row / return_transaction_guid | return_* + purchase_transaction_guid | mixed | Return lookup set in run window | — | ETL |  | `[lookup]` temp table for refund-row updates. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | 1 row / purchase transaction_guid | returned_amount, returned_fee_amount, returned_charity_amount | numeric(19,4) | Returned values at purchase row | -42.50 | Derived |  | `[lookup]` update from fact_return_aggregate by purchase guid. |
| public.fact_transaction | 1 row / refund transaction_guid | sales_fee_return_amount, surcharge_fee_return_amount, reservation_fee_return_amount, tax_return_amount, card_fee_return_amount, taxable_sales_fee_amount, charity_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` update from #fact_return_lookup with negative sign transforms. |
| public.fact_transaction | 1 row / refund transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Purchase fee amount tied to returned purchase | 5.00 | Derived |  | `[lookup]` from purchase item aggregate via return lookup. |

</details>

<details open>
<summary><strong>Analytics (11 fields)</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| AN_077 | analytics.transactions_data_source | 1 row / transaction_guid | returned_amount | numeric(19,4) | Returned amount | -42.50 | Derived |  | `[passthrough]` insert + post-insert sync update for returned_* fields |
| AN_078 | analytics.transactions_data_source | 1 row / transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | -5.00 | Derived |  | `[passthrough]` + sync update |
| AN_082 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_return_amount | numeric(19,4) | Sales fee return amount | -5.00 | Derived |  | `[passthrough]` |
| AN_083 | analytics.transactions_data_source | 1 row / transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Surcharge fee return amount | -1.50 | Derived |  | `[passthrough]` |
| AN_084 | analytics.transactions_data_source | 1 row / transaction_guid | reservation_fee_return_amount | numeric(19,4) | Reservation fee return amount | 0.00 | Derived |  | `[passthrough]` |
| AN_085 | analytics.transactions_data_source | 1 row / transaction_guid | tax_return_amount | numeric(19,4) | Tax return amount | -2.10 | Derived |  | `[passthrough]` |
| AN_086 | analytics.transactions_data_source | 1 row / transaction_guid | card_fee_return_amount | numeric(19,4) | Card fee return amount | -0.40 | Derived |  | `[passthrough]` |
| AN_087 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee amount for returned purchase | 5.00 | Derived |  | `[passthrough]` |
| AN_091 | analytics.transactions_data_source | 1 row / transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived |  | `[passthrough]` (public update logic affects refund rows) |
| AN_103 | analytics.transactions_data_source | 1 row / transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | -1.00 | Derived |  | `[passthrough]` + sync update |
| AN_104 | analytics.transactions_data_source | 1 row / transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | -1.00 | Derived |  | `[passthrough]` |

</details>
</details>

---

<details>
<summary><strong>Group D — Dimension lookup projection (52 fields)</strong></summary>

### Flow Diagram
```text
Raw (business ids/codes in transaction + performance + attributes)
-> Staging (surrogate key resolution in fact_transaction_base)
-> Public (fact_transaction keys + dim tables)
-> Analytics (dimension attribute projection)
```

<details>
<summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction | 1 row / transaction_guid | Affiliate_id, Channel_cd, Transaction_tp, UserAgent_nm, Property_nm | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; Property_nm can fall back to the purchase transaction property for refunds. |
| raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | movie_detail_id, theater_pkey | int | Movie/theater ids | 10293/4451 | Backend |  | Inputs to movie/theater keys. |
| raw_import_cache.transactionCommitted | 1 row / transaction_guid | Confirmation_cd | varchar | Prefix for return type mapping | REF123 | Backend |  | REF/RET/EXH prefix mapping to return type dimension. |
| raw_import_cache.transactionPurchaseFunnel | 1 row / tx | Funnel_cd | varchar | Funnel code | 14 | Backend |  | Input to purchase_funnel_key. |
| raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (Channel / PartnerPlatform) | varchar | Partner mapping attrs | TMO/WEBVIEW | Third-party |  | Inputs to partner dimension keys; refund rows can fall back to parent-purchase attributes. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | movie_key, affiliate_key, chain_key, channel_key, purchase_funnel_key, return_type_key, theater_key, transaction_type_key, user_agent_key, property_key, partner_channel_key, partner_platform_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` dimension joins resolve natural keys; COALESCE uses an unknown/sentinel key when no match exists. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | 1 row / transaction_guid | all *_key join columns | bigint | Join keys for dim projection | — | ETL |  | `[passthrough]` keys retained from staging. |
| public.dim_* tables | 1 row / dim key | descriptive attributes used by analytics | mixed | Lookup attributes | — | Backend/Third-party |  | Joined and projected in analytics loader. |

</details>

<details open>
<summary><strong>Analytics (52 fields)</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| AN_022 | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_name | varchar(400) | Affiliate name | Fandango iOS App | Backend |  | `[lookup]` aff.affiliate_name |
| AN_023 | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_group_name | varchar(100) | Affiliate group name | Mobile | Backend |  | `[lookup]` aff.affiliate_group_name |
| AN_024 | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_category_name | varchar(480) | Affiliate category name | O&O | Backend |  | `[lookup]` aff.affiliate_category_name |
| AN_025 | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_id | int | Affiliate id | 11879 | Backend |  | `[lookup]` aff.affiliate_id |
| AN_026 | analytics.transactions_data_source | 1 row / transaction_guid | channel | varchar(10) | Channel group | App | Backend |  | `[rename/cast][lookup]` channel.channel_group |
| AN_027 | analytics.transactions_data_source | 1 row / transaction_guid | channel_ungrouped | varchar(40) | Channel name | WAP | Backend |  | `[rename/cast][lookup]` channel.channel_name |
| AN_028 | analytics.transactions_data_source | 1 row / transaction_guid | filmid_child | int | Child movie id | 10293 | Backend |  | `[rename/cast][lookup]` film.movie_id |
| AN_029 | analytics.transactions_data_source | 1 row / transaction_guid | filmname_child | varchar(400) | Child movie title | Example Movie | Backend |  | `[rename/cast][lookup]` film.movie_title |
| AN_030 | analytics.transactions_data_source | 1 row / transaction_guid | filmid_parent | varchar(32) | Parent movie id | 872 | Backend |  | `[rename/cast][lookup]` film.movie_parent_id |
| AN_031 | analytics.transactions_data_source | 1 row / transaction_guid | filmname_parent | varchar(400) | Parent movie title | Franchise X | Backend |  | `[rename/cast][lookup]` film.movie_parent_title |
| AN_032 | analytics.transactions_data_source | 1 row / transaction_guid | releasedate | date | Release date | 2026-09-14 | Backend |  | `[rename/cast][lookup]` film.movie_release_date |
| AN_033 | analytics.transactions_data_source | 1 row / transaction_guid | movie_tms_id | int | Movie TMS id | 123456 | Third-party |  | `[lookup]` film.movie_tms_id |
| AN_034 | analytics.transactions_data_source | 1 row / transaction_guid | chainname | varchar(200) | Chain name | Example Cinemas | Backend |  | `[rename/cast][lookup]` chain.chain_name |
| AN_035 | analytics.transactions_data_source | 1 row / transaction_guid | chainid | varchar(10) | Chain id | CHN019 | Backend |  | `[rename/cast][lookup]` chain.chain_id |
| AN_036 | analytics.transactions_data_source | 1 row / transaction_guid | pos_vendor | varchar(50) | POS vendor | Vista | Third-party |  | `[rename/cast][lookup]` chain.point_of_sale_vendor |
| AN_037 | analytics.transactions_data_source | 1 row / transaction_guid | awards_program_name | varchar(320) | Awards program | Rewards+ | Backend |  | `[lookup]` chain.awards_program_name |
| AN_038 | analytics.transactions_data_source | 1 row / transaction_guid | theater_id | int | Theater id | 4451 | Backend |  | `[lookup]` tht.theater_id |
| AN_039 | analytics.transactions_data_source | 1 row / transaction_guid | theater_code | varchar(10) | Theater code | TH4451 | Backend |  | `[lookup]` tht.theater_code |
| AN_040 | analytics.transactions_data_source | 1 row / transaction_guid | theater_name | varchar(200) | Theater name | Example Hollywood | Backend |  | `[lookup]` tht.theater_name |
| AN_041 | analytics.transactions_data_source | 1 row / transaction_guid | pdi_id | varchar(6) | PDI id | AVSTA | Backend |  | `[lookup]` tht.pdi_id |
| AN_042 | analytics.transactions_data_source | 1 row / transaction_guid | pos_theater_id | varchar(20) | POS theater id | POS-4451 | Backend |  | `[rename/cast][lookup]` tht.point_of_sale_theater_id |
| AN_043 | analytics.transactions_data_source | 1 row / transaction_guid | latitude | real | Latitude | 34.10 | Backend |  | `[lookup]` tht.latitude |
| AN_044 | analytics.transactions_data_source | 1 row / transaction_guid | longitude | real | Longitude | -118.34 | Backend |  | `[lookup]` tht.longitude |
| AN_045 | analytics.transactions_data_source | 1 row / transaction_guid | point_of_sale_id | smallint | POS id | 1 | Backend |  | `[lookup]` tht.point_of_sale_id |
| AN_046 | analytics.transactions_data_source | 1 row / transaction_guid | address_1 | varchar(200) | Address 1 | 123 Main St | Backend |  | `[lookup]` tht.address_1 |
| AN_047 | analytics.transactions_data_source | 1 row / transaction_guid | address_2 | varchar(200) | Address 2 | Suite 200 | Backend |  | `[lookup]` tht.address_2 |
| AN_048 | analytics.transactions_data_source | 1 row / transaction_guid | city_name | varchar(160) | City | Los Angeles | Backend |  | `[lookup]` tht.city_name |
| AN_049 | analytics.transactions_data_source | 1 row / transaction_guid | state_code | varchar(2) | State code | CA | Backend |  | `[lookup]` tht.state_code |
| AN_050 | analytics.transactions_data_source | 1 row / transaction_guid | zip_code | varchar(10) | ZIP code | 90028 | Backend |  | `[lookup]` tht.zip_code |
| AN_051 | analytics.transactions_data_source | 1 row / transaction_guid | country_code | varchar(3) | Country code | US | Backend |  | `[lookup]` tht.country_code |
| AN_052 | analytics.transactions_data_source | 1 row / transaction_guid | tht_alternate_theater_phone_name | varchar(200) | Alternate phone/name | Box Office | Backend |  | `[rename/cast][lookup]` tht.alternate_theater_phone_name |
| AN_053 | analytics.transactions_data_source | 1 row / transaction_guid | chain_specific_site_id | varchar(10) | Chain site id | SITE-22 | Backend |  | `[lookup]` tht.chain_specific_site_id |
| AN_054 | analytics.transactions_data_source | 1 row / transaction_guid | paymentech_terminal_id | varchar(24) | Paymentech terminal id | PT-100 | Third-party |  | `[lookup]` tht.paymentech_terminal_id |
| AN_055 | analytics.transactions_data_source | 1 row / transaction_guid | cybersource_terminal_id | varchar(24) | Cybersource terminal id | CS-200 | Third-party |  | `[lookup]` tht.cybersource_terminal_id |
| AN_056 | analytics.transactions_data_source | 1 row / transaction_guid | screen_count | smallint | Screen count | 12 | Backend |  | `[lookup]` tht.screen_count |
| AN_057 | analytics.transactions_data_source | 1 row / transaction_guid | seat_count | int | Seat count | 1800 | Backend |  | `[lookup]` tht.seat_count |
| AN_058 | analytics.transactions_data_source | 1 row / transaction_guid | theater_time_zone_name | varchar(50) | Theater timezone | America/Los_Angeles | Backend |  | `[lookup]` tht.theater_time_zone_name |
| AN_060 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_type_id | varchar(40) | Transaction type id | Purchase | Backend |  | `[lookup]` tranxtype.transaction_type_id |
| AN_061 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_type_name | varchar(40) | Transaction type name | Ticket Purchase | Backend |  | `[lookup]` tranxtype.transaction_type_name |
| AN_062 | analytics.transactions_data_source | 1 row / transaction_guid | user_agent_id | varchar(480) | User agent id | UA-1 | Backend |  | `[lookup]` dua.user_agent_id |
| AN_063 | analytics.transactions_data_source | 1 row / transaction_guid | user_agent_description | varchar(480) | User agent description | iOS Safari | Backend |  | `[lookup]` dua.user_agent_description |
| AN_064 | analytics.transactions_data_source | 1 row / transaction_guid | purchase_funnel_id | varchar(40) | Funnel id | 14 | Backend |  | `[lookup]` dpf.purchase_funnel_id |
| AN_065 | analytics.transactions_data_source | 1 row / transaction_guid | purchase_funnel_name | varchar(100) | Funnel name | Guest Checkout | Backend |  | `[lookup]` dpf.purchase_funnel_name |
| AN_066 | analytics.transactions_data_source | 1 row / transaction_guid | return_method | varchar(20) | Return method | Self-service | Backend |  | `[lookup]` drt.return_method |
| AN_067 | analytics.transactions_data_source | 1 row / transaction_guid | return_type_code | varchar(3) | Return type code | REF | Backend |  | `[lookup]` drt.return_type_code |
| AN_068 | analytics.transactions_data_source | 1 row / transaction_guid | return_type_description | varchar(20) | Return type description | Refund | Backend |  | `[lookup]` drt.return_type_description |
| AN_088 | analytics.transactions_data_source | 1 row / transaction_guid | property_id | varchar(60) | Property id | Fandango USA | Backend |  | `[lookup]` dp.property_id |
| AN_112 | analytics.transactions_data_source | 1 row / transaction_guid | property_group | varchar | Property group | Fandango | Backend |  | `[lookup]` dp.property_group |
| AN_090 | analytics.transactions_data_source | 1 row / transaction_guid | braintree_merchant_id | varchar(60) | Merchant id | bt_merch_1 | Third-party |  | `[lookup]` tht.braintree_merchant_id |
| AN_108 | analytics.transactions_data_source | 1 row / transaction_guid | partner_channel | varchar | Partner channel id | TMO | Third-party |  | `[rename/cast][lookup]` dpc.partner_channel_id |
| AN_109 | analytics.transactions_data_source | 1 row / transaction_guid | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party |  | `[lookup]` dpc.partner_display_name |
| AN_110 | analytics.transactions_data_source | 1 row / transaction_guid | partner_platform | varchar | Partner platform id | WEBVIEW | Third-party |  | `[rename/cast][lookup]` dpp.partner_platform_id |


</details>
</details>

---

<details>
<summary><strong>Group E — Analytics-derived calcs and CASE buckets (11 fields)</strong></summary>

### Flow Diagram
```text
Raw (transaction lifecycle + performance + affiliate/channel ids)
-> Staging (fact_transaction_base)
-> Public (fact_transaction + dim_movie + dim_affiliate + dim_channel)
-> Analytics (CONVERT_TIMEZONE / DATEDIFF / CASE)
```

<details>
<summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.transaction + committed + rolledBack | 1 row / tx | Create_dm timestamps | timestamp | Lifecycle UTC timestamps | 2026-10-01... | Backend |  | Inputs to timezone derivations. |
| raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | Performance_dm | timestamp | Performance local timestamp | 2026-10-05 19:30:00 | Backend |  | Input to DATEDIFF metrics. |
| raw_import_cache.transaction | 1 row / tx | Affiliate_id, Channel_cd | mixed | Inputs to CASE bucketing | 11879/WAP | Backend |  | Feed dim-based CASE logic. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_* timestamps, performance_datetime_local, affiliate/channel/movie keys | mixed | Inputs to analytics derivations | — | Backend/ETL |  | `[passthrough]` inputs carried forward. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_transaction | 1 row / transaction_guid | lifecycle timestamps + performance_datetime_local | timestamp | Inputs for timezone/delta calculations | — | Backend/Derived |  | Used directly in analytics expressions. |
| public.dim_movie / dim_affiliate / dim_channel | dim grain | movie_release_date / affiliate_id / channel_name | mixed | Inputs to DATEDIFF and CASE logic | — | Backend |  | Joined and used in analytics calculations. |

</details>

<details open>
<summary><strong>Analytics (11 fields)</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| AN_004 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_start_datetime_los_angeles | timestamp | Pacific start datetime | 2026-10-01 09:10:00 | Derived |  | `[derived]` CONVERT_TIMEZONE('US/Pacific', start_utc) |
| AN_006 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_commit_datetime_los_angeles | timestamp | Pacific commit datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` timezone conversion |
| AN_008 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_rollback_datetime_los_angeles | timestamp | Pacific rollback datetime | NULL | Derived |  | `[derived]` timezone conversion |
| AN_010 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_los_angeles | timestamp | Pacific end datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` timezone conversion |
| AN_012 | analytics.transactions_data_source | 1 row / transaction_guid | deltareleasetopurchasedate | int | Days release->purchase day | 17 | Derived |  | `[derived]` DATEDIFF(release_date, purchase_day_pacific) |
| AN_013 | analytics.transactions_data_source | 1 row / transaction_guid | deltareleasetoshowtimedate | int | Days release->showtime day | 21 | Derived |  | `[derived]` DATEDIFF(release_date, performance_day) |
| AN_014 | analytics.transactions_data_source | 1 row / transaction_guid | deltapurchasetoshowtimedate | int | Days purchase day->showtime day | 4 | Derived |  | `[derived]` DATEDIFF(purchase_day_pacific, performance_day) |
| AN_069 | analytics.transactions_data_source | 1 row / transaction_guid | "Affiliate Grouping" | varchar(256) | Detailed affiliate bucket | iPhone App | Derived |  | `[conditional]` CASE by affiliate_id lists and NULL handling |
| AN_070 | analytics.transactions_data_source | 1 row / transaction_guid | "Affiliate Grouping - HIGH LEVEL BINS" | varchar(256) | High-level bucket | Fandango O&O | Derived |  | `[conditional]` CASE bins by affiliate_id |
| AN_071 | analytics.transactions_data_source | 1 row / transaction_guid | apps | varchar(256) | App bucket | Fandango App: iPhone | Derived |  | `[conditional]` CASE by affiliate set |
| AN_072 | analytics.transactions_data_source | 1 row / transaction_guid | platform | varchar(256) | Platform bucket | Mobile Apps | Derived |  | `[conditional]` CASE on channel and affiliate; channel_name = Platform is tested first |

</details>
</details>

---

<details>
<summary><strong>Group F — Raw enrichment at analytics step (1 field)</strong></summary>

### Flow Diagram
```text
Raw (raw_import_cache.designatedMarketArea)
-> Staging (fact_transaction_base theater_key path)
-> Public (dim_theater.zip_code)
-> Analytics (zip join to designatedMarketArea)
```

<details>
<summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| raw_import_cache.designatedMarketArea | 1 row / zip_code | zip_code, market_name | varchar | ZIP to DMA mapping | 90028 / Los Angeles | Third-party |  | External DMA source used in analytics join. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| warehouse_loader.fact_transaction_base | 1 row / transaction_guid | theater_key | bigint | Theater lookup key | 11872 | ETL |  | Provides path to theater ZIP in dim_theater. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.dim_theater | 1 row / theater_key | zip_code | varchar(10) | Theater ZIP code | 90028 | Backend |  | Join bridge to raw DMA map. |

</details>

<details open>
<summary><strong>Analytics (1 field)</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| AN_059 | analytics.transactions_data_source | 1 row / transaction_guid | dma_market_name | varchar(255) | DMA market name | Los Angeles | Third-party |  | `[lookup]` dma.market_name via dma.zip_code = tht.zip_code |

</details>
</details>

---

<details>
<summary><strong>Group G — Sales-tax enrichment (9 fields)</strong></summary>

### Flow Diagram
```text
Raw (data_lake.salestaxtransaction)
-> Staging (raw_import_cache.stgsalestaxtransaction)
-> Public (public.fact_sales_fee_tax)
-> Analytics (INSERT join, then salestax ETL update via #transactionSalesTax)
```

<details>
<summary><strong>Raw</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| data_lake.salestaxtransaction | line-item grain | document_number, lineitem_number, total_lineitem_tax, tax area/rate and status fields | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| temp #transactionSalesTax | 1 row / transaction_guid | sales_fee_tax_amount, returned_sales_fee_tax_amount, sales_fee_tax_return_amount, sales_fee_tax_amount_for_returned_purchase, pr_ticket_tax_amount, returned_ticket_tax_amount, ticket_tax_return_amount, ticket_tax_amount_for_returned_purchase | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` analytics INSERT joins public.fact_sales_fee_tax by transaction_guid; the separate salestax load also updates lag-window rows through this temp table. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|
| public.fact_sales_fee_tax | 1 row / transaction_guid | sales_fee_tax_amount, returned_sales_fee_tax_amount, sales_fee_tax_return_amount, sales_fee_tax_amount_for_returned_purchase, pr_ticket_tax_amount, returned_ticket_tax_amount, ticket_tax_return_amount, ticket_tax_amount_for_returned_purchase | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | Joined by transaction_guid in the analytics INSERT; the salestax ETL separately refreshes recent analytics rows via UPDATE. |

</details>

<details open>
<summary><strong>Analytics (9 fields)</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| AN_092 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_amount | decimal(19,4) | Sales fee tax amount | 0.35 | Third-party/Derived |  | `[lookup]` fstax field from LEFT JOIN public.fact_sales_fee_tax; the salestax load also refreshes lag-window rows via #transactionSalesTax.sales_fee_tax_amount |
| AN_093 | analytics.transactions_data_source | 1 row / transaction_guid | returned_sales_fee_tax_amount | decimal(19,4) | Returned sales fee tax amount | -0.35 | Third-party/Derived |  | `[lookup]` fstax field from LEFT JOIN public.fact_sales_fee_tax; the salestax load also refreshes lag-window rows via #transactionSalesTax.returned_sales_fee_tax_amount |
| AN_094 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_return_amount | decimal(19,4) | Sales fee tax return amount | -0.35 | Third-party/Derived |  | `[lookup]` fstax field from LEFT JOIN public.fact_sales_fee_tax; the salestax load also refreshes lag-window rows via #transactionSalesTax.sales_fee_tax_return_amount |
| AN_095 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_amount_for_returned_purchase | decimal(19,4) | Sales fee tax amount for returned purchase | 0.35 | Third-party/Derived |  | `[lookup]` fstax field from LEFT JOIN public.fact_sales_fee_tax; the salestax load also refreshes lag-window rows via #transactionSalesTax.sales_fee_tax_amount_for_returned_purchase |
| AN_124 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_amount | decimal(19,4) | Vertex ticket tax amount | 3.50 | Third-party/Derived |  | `[lookup]` fstax.pr_ticket_tax_amount AS ticket_tax_amount; salestax load refreshes this value via #transactionSalesTax. |
| AN_125 | analytics.transactions_data_source | 1 row / transaction_guid | pre_tax_ticket_sales_amount | decimal(19,4) | Ticket sales amount before tax | 31.50 | Derived |  | `[derived]` ft.ticket_sales_amount - COALESCE(fstax.pr_ticket_tax_amount, 0) |
| AN_126 | analytics.transactions_data_source | 1 row / transaction_guid | returned_ticket_tax_amount | decimal(19,4) | Returned ticket tax amount | -3.50 | Third-party/Derived |  | `[lookup]` fstax field from LEFT JOIN public.fact_sales_fee_tax; the salestax load also refreshes lag-window rows via #transactionSalesTax.returned_ticket_tax_amount |
| AN_127 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_return_amount | decimal(19,4) | Ticket tax return amount | -3.50 | Third-party/Derived |  | `[lookup]` fstax field from LEFT JOIN public.fact_sales_fee_tax; the salestax load also refreshes lag-window rows via #transactionSalesTax.ticket_tax_return_amount |
| AN_128 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_amount_for_returned_purchase | decimal(19,4) | Ticket tax amount for returned purchase | 3.50 | Third-party/Derived |  | `[lookup]` fstax field from LEFT JOIN public.fact_sales_fee_tax; the salestax load also refreshes lag-window rows via #transactionSalesTax.ticket_tax_amount_for_returned_purchase |


</details>
</details>

---

## Canonical Field ID Map (AN_001–AN_128)

| Field ID | Column Name |
|---|---|
| AN_001 | transaction_guid |
| AN_002 | is_successful |
| AN_003 | transaction_start_datetime_utc |
| AN_004 | transaction_start_datetime_los_angeles |
| AN_005 | transaction_commit_datetime_utc |
| AN_006 | transaction_commit_datetime_los_angeles |
| AN_007 | transaction_rollback_datetime_utc |
| AN_008 | transaction_rollback_datetime_los_angeles |
| AN_009 | transaction_end_datetime_utc |
| AN_010 | transaction_end_datetime_los_angeles |
| AN_011 | transaction_end_datetime_local |
| AN_012 | deltareleasetopurchasedate |
| AN_013 | deltareleasetoshowtimedate |
| AN_014 | deltapurchasetoshowtimedate |
| AN_015 | point_of_sale_confirmation_code |
| AN_016 | is_reserved_seating |
| AN_017 | reserved_seats |
| AN_018 | loyalty_code |
| AN_019 | performance_datetime_local |
| AN_020 | payment_types |
| AN_021 | processor_order_ids |
| AN_022 | affiliate_name |
| AN_023 | affiliate_group_name |
| AN_024 | affiliate_category_name |
| AN_025 | affiliate_id |
| AN_026 | channel |
| AN_027 | channel_ungrouped |
| AN_028 | filmid_child |
| AN_029 | filmname_child |
| AN_030 | filmid_parent |
| AN_031 | filmname_parent |
| AN_032 | releasedate |
| AN_033 | movie_tms_id |
| AN_034 | chainname |
| AN_035 | chainid |
| AN_036 | pos_vendor |
| AN_037 | awards_program_name |
| AN_038 | theater_id |
| AN_039 | theater_code |
| AN_040 | theater_name |
| AN_041 | pdi_id |
| AN_042 | pos_theater_id |
| AN_043 | latitude |
| AN_044 | longitude |
| AN_045 | point_of_sale_id |
| AN_046 | address_1 |
| AN_047 | address_2 |
| AN_048 | city_name |
| AN_049 | state_code |
| AN_050 | zip_code |
| AN_051 | country_code |
| AN_052 | tht_alternate_theater_phone_name |
| AN_053 | chain_specific_site_id |
| AN_054 | paymentech_terminal_id |
| AN_055 | cybersource_terminal_id |
| AN_056 | screen_count |
| AN_057 | seat_count |
| AN_058 | theater_time_zone_name |
| AN_059 | dma_market_name |
| AN_060 | transaction_type_id |
| AN_061 | transaction_type_name |
| AN_062 | user_agent_id |
| AN_063 | user_agent_description |
| AN_064 | purchase_funnel_id |
| AN_065 | purchase_funnel_name |
| AN_066 | return_method |
| AN_067 | return_type_code |
| AN_068 | return_type_description |
| AN_069 | "Affiliate Grouping" |
| AN_070 | "Affiliate Grouping - HIGH LEVEL BINS" |
| AN_071 | apps |
| AN_072 | platform |
| AN_073 | item_qty |
| AN_074 | gross_transaction_amount |
| AN_075 | sales_fee_amount |
| AN_076 | waived_sales_fee_ticket_count |
| AN_077 | returned_amount |
| AN_078 | returned_fee_amount |
| AN_079 | ticket_sales_amount |
| AN_080 | waived_sales_fee_amount |
| AN_081 | surcharge_fee_amount |
| AN_082 | sales_fee_return_amount |
| AN_083 | surcharge_fee_return_amount |
| AN_084 | reservation_fee_return_amount |
| AN_085 | tax_return_amount |
| AN_086 | card_fee_return_amount |
| AN_087 | sales_fee_amount_for_returned_purchase |
| AN_088 | property_id |
| AN_089 | pos_reference_cd |
| AN_090 | braintree_merchant_id |
| AN_091 | taxable_sales_fee_amount |
| AN_092 | sales_fee_tax_amount |
| AN_093 | returned_sales_fee_tax_amount |
| AN_094 | sales_fee_tax_return_amount |
| AN_095 | sales_fee_tax_amount_for_returned_purchase |
| AN_096 | is_test |
| AN_097 | devicetype |
| AN_098 | is_fanclub |
| AN_099 | waived_fanclub_fee_amount |
| AN_100 | waived_fanclub_fee_ticket_count |
| AN_101 | charity_amount |
| AN_102 | true_gross_transaction_amount |
| AN_103 | returned_charity_amount |
| AN_104 | charity_return_amount |
| AN_105 | is_buynow |
| AN_106 | charity_name |
| AN_107 | partner_defined_data_value |
| AN_108 | partner_channel |
| AN_109 | partner_display_name |
| AN_110 | partner_platform |
| AN_111 | network_token_usage |
| AN_112 | property_group |
| AN_113 | is_plf |
| AN_114 | is_imax |
| AN_115 | is_3d |
| AN_116 | is_higher_fee_amenity |
| AN_117 | utm_source |
| AN_118 | utm_medium |
| AN_119 | utm_campaign |
| AN_120 | utm_content |
| AN_121 | utm_term |
| AN_122 | ticket_tax_rate |
| AN_123 | is_ticket_tax_transaction |
| AN_124 | ticket_tax_amount |
| AN_125 | pre_tax_ticket_sales_amount |
| AN_126 | returned_ticket_tax_amount |
| AN_127 | ticket_tax_return_amount |
| AN_128 | ticket_tax_amount_for_returned_purchase |

---

## Quick Business Crosswalk

- **Group A:** direct pass-through style fields standardized in staging/public before analytics.
- **Group B:** aggregated commercial metrics from item/payment aggregates.
- **Group C:** return/refund derived fields and sync behavior.
- **Group D:** descriptive dim attributes projected from conformed keys.
- **Group E:** pure analytics-layer computations and CASE bucketing.
- **Group F:** targeted raw lookup enrichment (DMA by ZIP).
- **Group G:** post-insert sales-tax enrichment from salestax pipeline.

---


