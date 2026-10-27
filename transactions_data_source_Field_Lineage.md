# analytics.transactions_data_source Field Lineage (Grouped by Flow Pattern)

This document maps all **128 fields** in `analytics.transactions_data_source` across:

```text
MSSQL -> raw_import_cache -> warehouse_loader -> public -> analytics
```

## Conventions

- **Transform tags** used in **Logic Explanation**:
  - `[passthrough]` — carries a value forward without changing it.
  - `[rename/cast]` — renames a value, converts its data type, or both.
  - `[lookup]` — gets a value by matching a key to a lookup table or dimension.
  - `[derived]` — calculates a value from one or more inputs.
  - `[aggregate]` — combines multiple rows into a summarized result, such as a sum or count.
  - `[conditional]` — chooses or adjusts a value based on a condition, such as a `CASE` rule.
  - Tags can be combined when a transformation does more than one of these things.
- **Consistent field IDs**: `AN_001` … `AN_128` for analytics fields; `RAW_###`, `STG_###`, and `PUB_###` uniquely identify Raw, Staging, and Public rows.
- **Previous Stage ID(s)** in the Staging rows follow the transaction-level Raw → Stage → Public → Analytics overview. When a staging aggregate reads a lower-grain `public` fact internally, its raw/stage ancestry is shown in the ID cell and the physical `public` input is named in Logic Explanation. Public and Analytics rows continue to point to their immediate preceding lineage IDs.
- Grouping is by **flow pattern**, not business domain.

---

## Coverage Summary (128 / 128)

- Group A — Staging-derived pass-through: **31**
- Group B — Aggregated commercial measures: **13**
- Group C — Return/refund enrichment and sync: **11**
- Group D — Dimension lookup projection: **52**
- Group E — Analytics-derived calcs and CASE buckets: **11**
- Group F — Raw enrichment at analytics step: **1**
- Group G — Sales-tax enrichment: **9**

---

<details open>
<summary><strong>Group A — Staging-derived pass-through (31 fields)</strong></summary>

### Flow Diagram
```text
Raw (transaction, transactionCommitted, transactionRolledBack, transactionCustomer, transactionPerformanceAttribute, transactionAttribute, test-email lookup tables)
-> Staging (warehouse_loader.fact_transaction_base)
-> Public (public.fact_transaction)
-> Analytics (analytics.transactions_data_source)
```

<details>
<summary><strong>Raw</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| RAW_001 | raw_import_cache.transaction | 1 row / transaction_guid | Transaction_GUID | varchar(36) | Transaction identifier | 9F2A... | Backend |  | Base transaction key. |
| RAW_002 | raw_import_cache.transaction | 1 row / transaction_guid | Create_dm | timestamp | Start timestamp | 2026-10-01 16:10:00 | Backend |  | Source for start datetime. |
| RAW_003 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Create_dm | timestamp | Commit timestamp | 2026-10-01 16:11:05 | Backend |  | Source for commit/end logic. |
| RAW_004 | raw_import_cache.transactionRolledBack | 1 row / transaction_guid | Create_dm | timestamp | Rollback timestamp | NULL | Backend |  | Source for rollback/end logic. |
| RAW_005 | raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | Performance_dm | timestamp | Performance local datetime | 2026-10-05 19:30:00 | Backend |  | Source for performance_datetime_local. |
| RAW_006 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Confirmation_cd | varchar(40) | POS confirmation code | CNF908812 | Backend |  | Source for point_of_sale_confirmation_code. |
| RAW_007 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | ReservedSeats_tx | varchar(100) | Reserved seat payload | A10,A11 | Backend |  | Source for reserved seats outputs. |
| RAW_008 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Loyalty_cd | varchar(40) | Loyalty code | LOY123 | Backend |  | Source for loyalty_code. |
| RAW_009 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | POS_reference_cd | varchar(40) | POS reference code | POSREF1 | Backend |  | Source for pos_reference_cd. |
| RAW_010 | raw_import_cache.transaction | 1 row / transaction_guid | test_bt | boolean | Test marker | false | Backend |  | One input to is_test; the CASE also checks customer email, test-email tables/patterns, theater PDI, and transaction type. |
| RAW_011 | raw_import_cache.transactionCustomer + raw_import_cache.testEmailAddress + raw_import_cache.testTransactionEmailList | transaction/customer or email row | Email_nm, email_tx, activation/expiration, active/pattern flags | mixed | Test classification inputs | — | Backend |  | Used with transaction.test_bt to derive is_test. |
| RAW_012 | raw_import_cache.transaction | 1 row / transaction_guid | DeviceType_cd | varchar | Device type | Mobile | Backend |  | Source for devicetype. |
| RAW_013 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (FanClubStatus) | varchar | Fanclub status attr | Active | Backend |  | Input to is_fanclub. |
| RAW_014 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (IsBuyNowOrder) | varchar | Buy-now attr | true | Backend |  | Input to is_buynow. |
| RAW_015 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (CharityName) | varchar | Charity name attr | Save The Arts | Backend |  | Source for charity_name. |
| RAW_016 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (PartnerDefinedData) | varchar | Partner-defined attr | campaign=fall | Third-party |  | Source for partner_defined_data_value. |
| RAW_017 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (networktoken) | varchar | Network token attr | true | Backend |  | Input to network_token_usage. |
| RAW_018 | raw_import_cache.transactionPerformanceAttribute + mapping table | many-to-one tx | amenity_ids + mapped flags | varchar | Amenity rollup inputs | 3,10 | Backend/Derived |  | Inputs for is_plf/is_imax/is_3d/is_higher_fee_amenity. |
| RAW_019 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (UtmSource/UtmMedium/UtmCampaign/UtmContent/UtmTerm) | varchar(255) | UTM attrs | google/cpc/... | Backend |  | Source for utm_* fields. |
| RAW_020 | raw_import_cache.transaction | 1 row / transaction_guid | ticketTaxRate_rt | decimal(6,4) | PR ticket tax rate | 11.5000 | Backend |  | Source for ticket_tax_rate. |
| RAW_021 | raw_import_cache.transaction | 1 row / transaction_guid | ticketTaxTransaction_bt | boolean | PR ticket-tax flag | true | Backend |  | Source for is_ticket_tax_transaction. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| STG_001 | RAW_001 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction id | 9F2A... | Backend |  | `[passthrough]` Copies transaction_guid from raw_import_cache.transaction. |
| STG_002 | RAW_003, RAW_004 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_successful | boolean | Success flag | true | Derived |  | `[conditional]` Sets success when raw_import_cache.transactionCommitted has a row and raw_import_cache.transactionRolledBack does not. |
| STG_003 | RAW_002 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_start_datetime | timestamp | Start UTC | 2026-10-01 16:10:00 | Backend |  | `[rename/cast]` Casts raw_import_cache.transaction.Create_dm as transaction_start_datetime. |
| STG_004 | RAW_003 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_commit_datetime | timestamp | Commit UTC | 2026-10-01 16:11:05 | Backend |  | `[rename/cast]` Copies Create_dm from raw_import_cache.transactionCommitted. |
| STG_005 | RAW_004 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_rollback_datetime | timestamp | Rollback UTC | NULL | Backend |  | `[rename/cast]` Copies Create_dm from raw_import_cache.transactionRolledBack. |
| STG_006 | RAW_002, RAW_003, RAW_004 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_end_datetime | timestamp | End UTC | 2026-10-01 16:11:05 | Derived |  | `[conditional]` Chooses the latest available time from raw_import_cache.transaction, raw_import_cache.transactionCommitted, and raw_import_cache.transactionRolledBack; falls back to transaction start. |
| STG_007 | RAW_005 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | performance_datetime_local | timestamp | Local performance datetime | 2026-10-05 19:30:00 | Backend |  | `[passthrough]` Copies Performance_dm from raw_import_cache.transactionPerformanceAttribute. |
| STG_008 | RAW_006 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation | CNF908812 | Backend |  | `[rename/cast]` Copies Confirmation_cd from raw_import_cache.transactionCommitted. |
| STG_009 | RAW_007 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived |  | `[conditional]` Sets the seating flag when ReservedSeats_tx from raw_import_cache.transactionCommitted is nonblank. |
| STG_010 | RAW_007 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | reserved_seats | varchar(100) | Reserved seats | A10,A11 | Backend |  | `[conditional]` Trims ReservedSeats_tx from raw_import_cache.transactionCommitted; empty text becomes NULL. |
| STG_011 | RAW_008 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code | LOY123 | Backend |  | `[conditional]` Trims Loyalty_cd from raw_import_cache.transactionCommitted; empty text becomes NULL. |
| STG_012 | RAW_009 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | POS_reference_cd | varchar(40) | POS ref code | POSREF1 | Backend |  | `[passthrough]` Copies POS_reference_cd from raw_import_cache.transactionCommitted. |
| STG_013 | RAW_010, RAW_011, RAW_025, RAW_026 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_test | boolean | Test flag | false | Derived |  | `[conditional]` Derives is_test from raw_import_cache.transaction, raw_import_cache.transactionCustomer, raw_import_cache.testEmailAddress, and raw_import_cache.testTransactionEmailList; also checks public.dim_theater and public.dim_transaction_type. |
| STG_014 | RAW_012 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | deviceType | varchar | Device type | Mobile | Backend |  | `[rename/cast]` Copies DeviceType_cd from raw_import_cache.transaction as deviceType. |
| STG_015 | RAW_013 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived |  | `[conditional]` Maps listed active FanClubStatus values from raw_import_cache.transactionAttribute to true. |
| STG_016 | RAW_014 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived |  | `[conditional]` Sets the flag when IsBuyNowOrder in raw_import_cache.transactionAttribute is true. |
| STG_017 | RAW_015 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | charity_name | varchar(256) | Charity name | Save The Arts | Backend |  | `[lookup]` Reads CharityName from raw_import_cache.transactionAttribute. |
| STG_018 | RAW_016 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner-defined payload | campaign=fall | Third-party |  | `[lookup]` Reads PartnerDefinedData from raw_import_cache.transactionAttribute. |
| STG_019 | RAW_017 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | network_token_usage | boolean | Network token usage | true | Derived |  | `[conditional]` Reads NetworkToken from raw_import_cache.transactionAttribute; for refunds, raw_import_cache.refund identifies the parent transaction whose attribute is the fallback. |
| STG_020 | RAW_018 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_plf, is_imax, is_3d, is_higher_fee_amenity | boolean | Amenity flags | false | Derived |  | `[aggregate]` Matches amenity_ids from raw_import_cache.transactionPerformanceAttribute to analytics_manual.plf_amenity_mapping and groups flags by raw_import_cache.transaction.transaction_guid. |
| STG_021 | RAW_019 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_source, utm_medium, utm_campaign, utm_content, utm_term | varchar(255) | UTM params | google/cpc/... | Backend |  | `[lookup]` Joins five named UTM attributes from raw_import_cache.transactionAttribute by transaction GUID. |
| STG_022 | RAW_020 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | PR ticket tax rate | 11.5000 | Backend |  | `[rename/cast]` Copies ticketTaxRate_rt from raw_import_cache.transaction. |
| STG_023 | RAW_021 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | PR ticket-tax txn flag | true | Derived |  | `[conditional]` Copies ticketTaxTransaction_bt from raw_import_cache.transaction; NULL defaults to false. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| PUB_001 | STG_001, STG_002, STG_003, STG_004, STG_005, STG_006 | public.fact_transaction | 1 row / transaction_guid | transaction_guid, is_successful, transaction_start_datetime, transaction_commit_datetime, transaction_rollback_datetime, transaction_end_datetime | mixed | Lifecycle fields | — | Backend/Derived |  | `[passthrough]` Copies lifecycle fields from warehouse_loader.fact_transaction_base into public.fact_transaction. |
| PUB_002 | STG_006, STG_028 | public.fact_transaction | 1 row / transaction_guid | transaction_end_datetime_local | timestamp | End local datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` Converts warehouse_loader.fact_transaction_base.transaction_end_datetime using public.dim_theater.theater_time_zone_name. |
| PUB_003 | STG_008, STG_009, STG_010, STG_011, STG_007, STG_012 | public.fact_transaction | 1 row / transaction_guid | point_of_sale_confirmation_code, is_reserved_seating, reserved_seats, loyalty_code, performance_datetime_local, POS_reference_cd | mixed | Core transaction attrs | — | Backend/Derived |  | `[passthrough]` Copies confirmation, seating, loyalty, and performance fields from warehouse_loader.fact_transaction_base. |
| PUB_004 | STG_013, STG_014, STG_015, STG_016, STG_017, STG_018, STG_019 | public.fact_transaction | 1 row / transaction_guid | is_test, deviceType, is_fanclub, is_buynow, charity_name, partner_defined_data_value, network_token_usage | mixed | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` Copies test, device, fanclub, buy-now, charity, partner, and network-token fields from warehouse_loader.fact_transaction_base. |
| PUB_005 | STG_020 | public.fact_transaction | 1 row / transaction_guid | is_plf, is_imax, is_3d, is_higher_fee_amenity | boolean | Amenity flags | false | Derived |  | `[passthrough]` Copies amenity flags from warehouse_loader.fact_transaction_base. |
| PUB_006 | STG_022, STG_023, STG_021 | public.fact_transaction | 1 row / transaction_guid | ticket_tax_rate, is_ticket_tax_transaction, utm_source, utm_medium, utm_campaign, utm_content, utm_term | mixed | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` Copies UTM and ticket-tax fields from warehouse_loader.fact_transaction_base. |

</details>

<details open>
<summary><strong>Analytics (31 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| AN_001 | PUB_001 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction id | 9F2A... | Backend |  | `[passthrough]` Reads public.fact_transaction; public.fact_transaction.transaction_guid |
| AN_002 | PUB_001 | analytics.transactions_data_source | 1 row / transaction_guid | is_successful | boolean | Success flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; public.fact_transaction.is_successful |
| AN_003 | PUB_001 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_start_datetime_utc | timestamp | Start UTC | 2026-10-01 16:10:00 | Backend |  | `[rename/cast]` Reads public.fact_transaction; public.fact_transaction.transaction_start_datetime |
| AN_005 | PUB_001 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_commit_datetime_utc | timestamp | Commit UTC | 2026-10-01 16:11:05 | Backend |  | `[rename/cast]` Reads public.fact_transaction; public.fact_transaction.transaction_commit_datetime |
| AN_007 | PUB_001 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_rollback_datetime_utc | timestamp | Rollback UTC | NULL | Backend |  | `[rename/cast]` Reads public.fact_transaction; public.fact_transaction.transaction_rollback_datetime |
| AN_009 | PUB_001 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_utc | timestamp | End UTC | 2026-10-01 16:11:05 | Derived |  | `[rename/cast]` Reads public.fact_transaction; public.fact_transaction.transaction_end_datetime |
| AN_011 | PUB_002 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_local | timestamp | End local | 2026-10-01 09:11:05 | Derived |  | `[passthrough]` Reads public.fact_transaction; public.fact_transaction.transaction_end_datetime_local |
| AN_015 | PUB_003 | analytics.transactions_data_source | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation | CNF908812 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies point_of_sale_confirmation_code unchanged. |
| AN_016 | PUB_003 | analytics.transactions_data_source | 1 row / transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_reserved_seating unchanged. |
| AN_017 | PUB_003 | analytics.transactions_data_source | 1 row / transaction_guid | reserved_seats | varchar(100) | Reserved seats | A10,A11 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies reserved_seats unchanged. |
| AN_018 | PUB_003 | analytics.transactions_data_source | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code | LOY123 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies loyalty_code unchanged. |
| AN_019 | PUB_003 | analytics.transactions_data_source | 1 row / transaction_guid | performance_datetime_local | timestamp | Performance local | 2026-10-05 19:30:00 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies performance_datetime_local unchanged. |
| AN_089 | PUB_003 | analytics.transactions_data_source | 1 row / transaction_guid | pos_reference_cd | varchar(40) | POS reference code | POSREF1 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies pos_reference_cd unchanged. |
| AN_096 | PUB_004 | analytics.transactions_data_source | 1 row / transaction_guid | is_test | boolean | Test flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_test unchanged. |
| AN_097 | PUB_004 | analytics.transactions_data_source | 1 row / transaction_guid | devicetype | varchar | Device type | Mobile | Backend |  | `[passthrough]` Reads public.fact_transaction; copies devicetype unchanged. |
| AN_098 | PUB_004 | analytics.transactions_data_source | 1 row / transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_fanclub unchanged. |
| AN_105 | PUB_004 | analytics.transactions_data_source | 1 row / transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_buynow unchanged. |
| AN_106 | PUB_004 | analytics.transactions_data_source | 1 row / transaction_guid | charity_name | varchar(256) | Charity name | Save The Arts | Backend |  | `[passthrough]` Reads public.fact_transaction; copies charity_name unchanged. |
| AN_107 | PUB_004 | analytics.transactions_data_source | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner payload | campaign=fall | Third-party |  | `[passthrough]` Reads public.fact_transaction; copies partner_defined_data_value unchanged. |
| AN_111 | PUB_004 | analytics.transactions_data_source | 1 row / transaction_guid | network_token_usage | boolean | Network token usage | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies network_token_usage unchanged. |
| AN_113 | PUB_005 | analytics.transactions_data_source | 1 row / transaction_guid | is_plf | boolean | PLF flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_plf unchanged. |
| AN_114 | PUB_005 | analytics.transactions_data_source | 1 row / transaction_guid | is_imax | boolean | IMAX flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_imax unchanged. |
| AN_115 | PUB_005 | analytics.transactions_data_source | 1 row / transaction_guid | is_3d | boolean | 3D flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_3d unchanged. |
| AN_116 | PUB_005 | analytics.transactions_data_source | 1 row / transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_higher_fee_amenity unchanged. |
| AN_117 | PUB_006 | analytics.transactions_data_source | 1 row / transaction_guid | utm_source | varchar(255) | UTM source | google | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_source unchanged. |
| AN_118 | PUB_006 | analytics.transactions_data_source | 1 row / transaction_guid | utm_medium | varchar(255) | UTM medium | cpc | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_medium unchanged. |
| AN_119 | PUB_006 | analytics.transactions_data_source | 1 row / transaction_guid | utm_campaign | varchar(255) | UTM campaign | fall_promo | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_campaign unchanged. |
| AN_120 | PUB_006 | analytics.transactions_data_source | 1 row / transaction_guid | utm_content | varchar(255) | UTM content | banner_a | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_content unchanged. |
| AN_121 | PUB_006 | analytics.transactions_data_source | 1 row / transaction_guid | utm_term | varchar(255) | UTM term | movie+tickets | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_term unchanged. |
| AN_122 | PUB_006 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | PR ticket tax rate | 11.5000 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies ticket_tax_rate unchanged. |
| AN_123 | PUB_006 | analytics.transactions_data_source | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | PR tax txn flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_ticket_tax_transaction unchanged. |

</details>
</details>

---

<details>
<summary><strong>Group B — Aggregated commercial measures (13 fields)</strong></summary>

### Flow Diagram
```text
Raw-origin payment and item data
  -> Stage: transaction-level aggregates
     (warehouse_loader.fact_payment_aggregate,
      warehouse_loader.fact_transaction_item_aggregate)
  -> Public: transaction-level fact (public.fact_transaction)
  -> analytics.transactions_data_source

Side note: the aggregates read row-level public payment/item facts and dimensions, which trace back to raw inputs; details are listed below.
```

<details>
<summary><strong>Raw</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| RAW_022 | raw_import_cache.transactionPayment + raw_import_cache.paypaltrrdetail + raw_import_cache.paypalstldetail + raw_import_cache.processorbraintreeauthorizationrequest + raw_import_cache.processorbraintreeauthorizationresponse + raw_import_cache.processorbraintreerefundrequest + raw_import_cache.processorbraintreerefundresponse | payment row / transaction | Payment_tp, Account_cd, Transaction_at, Payment_GUID, processor transaction IDs | mixed | Payment amounts, types, and processor order IDs | VISA / ord_123 | Backend/Third-party |  | `factPayment.sql` reads transactionPayment and processor response/request feeds; payment aggregation reads the resulting fact_payment. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | Item_qy, Item_at, SalesFee_at, WaivedSalesFee_at, SurchargeFee_at, ReservedFee_at, CardFee_at, Shipping_at, taxable_at, totaladmissions_qy, Item_tp | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_035 | raw_import_cache.ticketTypeMapping + raw_import_cache.ticketCategory | mapping/category row | pos_description, chain_id, category_id, category_description, display_description | mixed | Item description lookups used by aggregate | — | Backend |  | `factTransactionItemAggregate.sql` uses these mappings to determine item_type labels before the transaction-level rollup. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| STG_024 | RAW_022 | warehouse_loader.fact_payment_aggregate | 1 row / transaction_guid | payment_types, processor_order_ids, gross_transaction_amount | mixed | Payment aggregates at tx grain | — | Derived/Third-party |  | `[aggregate]` Transaction-level rollup of raw payment lineage. SQL reads public.fact_payment and joins public.dim_payment_instrument and public.dim_payment_type for payment type labels. |
| STG_025 | RAW_023, RAW_035 | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_count, item_sales_amount, item_fees_amount, item_surcharge_fee_amount, item_waived_sales_fee_amount, fanclub_waived_sales_fee_amount, fee_waiver_item_count, fanclub_fee_waiver_item_count, item_taxable_sales_fee_amount, charity_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item lineage. SQL reads public.fact_transaction_item and uses public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory for item labels. Donation amounts feed charity separately; subscriptions are excluded from item_count. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| PUB_020 | STG_001, STG_002, STG_003, STG_006, STG_028 | public.fact_payment + public.dim_payment_instrument + public.dim_payment_type | payment grain | transaction_guid, transaction_amount, payment_instrument_key, processor_order_id, payment_type_id | mixed | Payment facts and type lookup used by payment aggregate | — | ETL |  | `[lookup]` Builds public.fact_payment by joining warehouse_loader.fact_transaction_base, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.paypaltrrdetail, raw_import_cache.paypalstldetail, raw_import_cache.processorbraintreeauthorizationrequest, raw_import_cache.processorbraintreeauthorizationresponse, raw_import_cache.processorbraintreerefundrequest, and raw_import_cache.processorbraintreerefundresponse; it also uses public.dim_payment_instrument, public.dim_payment_type, public.dim_processor, public.dim_processor_error_code, and public.dim_credit_card_type. The aggregate reads public.fact_payment and the payment instrument/type dimensions. |
| PUB_007 | STG_024 | public.fact_transaction | 1 row / transaction_guid | payment_types, processor_order_ids | varchar | Payment labels and processor refs | VISA / ord_123 | Derived/Third-party |  | `[passthrough]` Copies payment labels and processor IDs from warehouse_loader.fact_payment_aggregate. |
| PUB_021 | STG_001, STG_002, STG_003, STG_006, STG_028 | public.fact_transaction_item + public.dim_chain + public.dim_purchase_type + public.dim_theater | item grain | transaction_guid, item_count, item_amount, sales_fee_amount, surcharge_fee_amount, waived_sales_fee_amount, taxable_sales_fee_amount | mixed | Item facts used by item aggregate | — | ETL |  | `[lookup]` Joins raw_import_cache.transactionItem to warehouse_loader.fact_transaction_base, public.dim_purchase_type, and public.dim_theater to build public.fact_transaction_item. Its aggregate also reads public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory. |
| PUB_008 | STG_025 | public.fact_transaction | 1 row / transaction_guid | item_count | int | Item quantity | 2 | Derived |  | `[rename/cast]` Copies item_count from warehouse_loader.fact_transaction_item_aggregate. |
| PUB_009 | STG_024, STG_025 | public.fact_transaction | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross minus charity | 42.50 | Derived |  | `[derived]` Sets gross amount to warehouse_loader.fact_payment_aggregate.gross_transaction_amount minus warehouse_loader.fact_transaction_item_aggregate.charity_amount. |
| PUB_010 | STG_025, STG_024 | public.fact_transaction | 1 row / transaction_guid | sales_fee_amount, surcharge_fee_amount, waived_sales_fee_amount, waived_fanclub_fee_amount, waived_sales_fee_ticket_count, waived_fanclub_fee_ticket_count, ticket_sales_amount, taxable_sales_fee_amount, charity_amount, true_gross_transaction_amount | numeric(19,4) | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate and fact_payment_aggregate. |

</details>

<details open>
<summary><strong>Analytics (13 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| AN_020 | PUB_007 | analytics.transactions_data_source | 1 row / transaction_guid | payment_types | varchar(200) | Payment types | VISA | Derived |  | `[passthrough]` Reads public.fact_transaction; copies payment_types from public.fact_transaction, assembled by warehouse_loader.fact_payment_aggregate. |
| AN_021 | PUB_007 | analytics.transactions_data_source | 1 row / transaction_guid | processor_order_ids | varchar(124) | Processor order ids | ord_123 | Third-party |  | `[passthrough]` Reads public.fact_transaction; copies processor_order_ids from public.fact_transaction, assembled by warehouse_loader.fact_payment_aggregate. |
| AN_073 | PUB_008 | analytics.transactions_data_source | 1 row / transaction_guid | item_qty | int | Item qty | 2 | Derived |  | `[rename/cast]` Reads public.fact_transaction; renames item_count to item_qty; public.fact_transaction gets the count from warehouse_loader.fact_transaction_item_aggregate. |
| AN_074 | PUB_009 | analytics.transactions_data_source | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies gross_transaction_amount from public.fact_transaction; factTransaction.sql derives it from warehouse_loader.fact_payment_aggregate less aggregate charity. |
| AN_075 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee | 5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies sales_fee_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_076 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | waived_sales_fee_ticket_count | numeric(19,4) | Waived sales fee ticket count | 0 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies waived sales-fee ticket count from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_079 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies ticket_sales_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_080 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies waived_sales_fee_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_081 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee | 1.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies surcharge_fee_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_099 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies waived_fanclub_fee_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_100 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | waived_fanclub_fee_ticket_count | numeric(19,4) | Waived fanclub fee ticket count | 0 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies waived_fanclub_fee_ticket_count from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_101 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies charity_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_102 | PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | true_gross_transaction_amount | numeric(19,4) | Gross including charity | 43.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies true_gross_transaction_amount from public.fact_transaction, built from warehouse_loader.fact_payment_aggregate. |

</details>
</details>

---

<details>
<summary><strong>Group C — Return/refund enrichment and sync (11 fields)</strong></summary>

### Flow Diagram
```text
Raw-origin refund data
  -> Stage: transaction-level return calculations
  -> Public: transaction-level fact updates (public.fact_transaction)
  -> Analytics (analytics.transactions_data_source)

Side note: public.fact_return and related dimensions support the calculations; they trace the refund data to the transaction-level updates. Details are listed below.
```

<details>
<summary><strong>Raw</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| RAW_024 | raw_import_cache.refund | 1 row / refund line | Price_at, Fee_at, Tax_at, Surcharge_at, ReservedFee_at, CardFee_at, taxable_at, donation_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| STG_026 | RAW_024 | warehouse_loader.fact_return_aggregate | 1 row / purchase_transaction_guid | returned_ticket_amount, returned_fee_amount, returned_charity_amount | numeric | Purchase-side returned rollups | — | Derived |  | `[aggregate]` Transaction-level rollup of raw refund lineage. The ETL sums successful public.fact_return rows by purchase_transaction_guid. |
| STG_027 | RAW_024 | #fact_return_lookup (temp) | 1 row / return_transaction_guid | return_* + purchase_transaction_guid | mixed | Return lookup set in run window | — | ETL |  | `[lookup]` Stages raw refund lineage for refund-row updates by copying return details from public.fact_return into #fact_return_lookup. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| PUB_022 | STG_001, STG_002, STG_003, STG_004, STG_006, STG_007, STG_008, STG_024, STG_025, STG_028 | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_transaction_guid, purchase_transaction_guid, return_amount, return_fee_amount, return_tax_amount, return_surcharge_amount, return_reservation_fee_amount, return_card_fee_amount, return_charity_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_011 | STG_026 | public.fact_transaction | 1 row / purchase transaction_guid | returned_amount, returned_fee_amount, returned_charity_amount | numeric(19,4) | Returned values at purchase row | -42.50 | Derived |  | `[lookup]` Updates purchase rows from warehouse_loader.fact_return_aggregate, matched by purchase GUID. |
| PUB_012 | STG_027 | public.fact_transaction | 1 row / refund transaction_guid | sales_fee_return_amount, surcharge_fee_return_amount, reservation_fee_return_amount, tax_return_amount, card_fee_return_amount, taxable_sales_fee_amount, charity_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| PUB_013 | STG_027, STG_025 | public.fact_transaction | 1 row / refund transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Purchase fee amount tied to returned purchase | 5.00 | Derived |  | `[lookup]` Updates public.fact_transaction from warehouse_loader.fact_transaction_item_aggregate through #fact_return_lookup from public.fact_return. |

</details>

<details open>
<summary><strong>Analytics (11 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| AN_077 | PUB_011 | analytics.transactions_data_source | 1 row / transaction_guid | returned_amount | numeric(19,4) | Returned amount | -42.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies returned_amount from public.fact_transaction; factTransaction.sql updates that table from warehouse_loader.fact_return_aggregate, with delayed analytics sync from public.fact_transaction. |
| AN_078 | PUB_011 | analytics.transactions_data_source | 1 row / transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | -5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies returned_fee_amount from public.fact_transaction; it is updated from warehouse_loader.fact_return_aggregate, with delayed analytics sync from public.fact_transaction. |
| AN_082 | PUB_012 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_return_amount | numeric(19,4) | Sales fee return amount | -5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies sales_fee_return_amount from public.fact_transaction; factTransaction.sql negates the fee from public.fact_return via #fact_return_lookup. |
| AN_083 | PUB_012 | analytics.transactions_data_source | 1 row / transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Surcharge fee return amount | -1.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies surcharge_fee_return_amount from public.fact_transaction; factTransaction.sql negates the return surcharge from public.fact_return via #fact_return_lookup. |
| AN_084 | PUB_012 | analytics.transactions_data_source | 1 row / transaction_guid | reservation_fee_return_amount | numeric(19,4) | Reservation fee return amount | 0.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies reservation_fee_return_amount from public.fact_transaction; factTransaction.sql negates the return reservation fee from public.fact_return via #fact_return_lookup. |
| AN_085 | PUB_012 | analytics.transactions_data_source | 1 row / transaction_guid | tax_return_amount | numeric(19,4) | Tax return amount | -2.10 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies tax_return_amount from public.fact_transaction; factTransaction.sql negates the return tax from public.fact_return via #fact_return_lookup. |
| AN_086 | PUB_012 | analytics.transactions_data_source | 1 row / transaction_guid | card_fee_return_amount | numeric(19,4) | Card fee return amount | -0.40 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies card_fee_return_amount from public.fact_transaction; factTransaction.sql negates the return card fee from public.fact_return via #fact_return_lookup. |
| AN_087 | PUB_013 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee amount for returned purchase | 5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies sales_fee_amount_for_returned_purchase from public.fact_transaction; factTransaction.sql gets it from warehouse_loader.fact_transaction_item_aggregate through #fact_return_lookup from public.fact_return. |
| AN_091 | PUB_012 | analytics.transactions_data_source | 1 row / transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies taxable_sales_fee_amount from public.fact_transaction; factTransaction.sql updates refund rows from public.fact_return via #fact_return_lookup. |
| AN_103 | PUB_011 | analytics.transactions_data_source | 1 row / transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | -1.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies returned_charity_amount from public.fact_transaction; it is updated from warehouse_loader.fact_return_aggregate, with delayed analytics sync from public.fact_transaction. |
| AN_104 | PUB_012 | analytics.transactions_data_source | 1 row / transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | -1.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies charity_return_amount from public.fact_transaction; factTransaction.sql negates the value from public.fact_return via #fact_return_lookup. |

</details>
</details>

---

<details>
<summary><strong>Group D — Dimension lookup projection (52 fields)</strong></summary>

### Flow Diagram
```text
Raw transaction identifiers and codes
  -> Stage: transaction keys (warehouse_loader.fact_transaction_base)
  -> Public: transaction-level fact (public.fact_transaction)
  -> Analytics: transaction-level dimension attributes

Side note: public dimensions supply descriptive lookup values; details are listed below.
```

<details>
<summary><strong>Raw</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| RAW_036 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | names, codes, dates, locations, parent IDs, vendor/terminal attributes, and partner labels | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| RAW_025 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | Affiliate_id, Channel_cd, Transaction_tp, UserAgent_nm, Property_nm, refund-to-purchase GUID | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| RAW_026 | raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | movie_detail_id, theater_pkey | int | Movie/theater ids | 10293/4451 | Backend |  | Inputs to movie/theater keys. |
| RAW_027 | raw_import_cache.transactionPurchaseFunnel | 1 row / tx | Funnel_cd | varchar | Funnel code | 14 | Backend |  | Input to purchase_funnel_key. |
| RAW_028 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Confirmation_cd | varchar | Prefix for return type mapping | REF123 | Backend |  | REF/RET/EXH prefix mapping to return type dimension. |
| RAW_029 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (Channel / PartnerPlatform) | varchar | Partner mapping attrs | TMO/WEBVIEW | Third-party |  | Inputs to partner dimension keys; refund rows can fall back to parent-purchase attributes. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| STG_028 | RAW_025, RAW_026, RAW_028, RAW_027, RAW_029 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | movie_key, affiliate_key, chain_key, channel_key, purchase_funnel_key, return_type_key, theater_key, transaction_type_key, user_agent_key, property_key, partner_channel_key, partner_platform_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| PUB_014 | STG_028 | public.fact_transaction | 1 row / transaction_guid | all *_key join columns | bigint | Join keys for dim projection | — | ETL |  | `[passthrough]` Copies surrogate keys from warehouse_loader.fact_transaction_base. |
| PUB_015 | RAW_036 | public.dim_* tables | 1 row / dim key | descriptive attributes used by analytics | mixed | Lookup attributes | — | Backend/Third-party |  | `[lookup]` Analytics reads public.dim_affiliate, public.dim_movie, public.dim_channel, public.dim_chain, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_purchase_funnel, public.dim_return_type, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform. Their tier_1_dimensions jobs load the raw inputs grouped in RAW_036. |

</details>

<details open>
<summary><strong>Analytics (52 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| AN_022 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_name | varchar(400) | Affiliate name | Fandango iOS App | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_affiliate; public.dim_affiliate.affiliate_name |
| AN_023 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_group_name | varchar(100) | Affiliate group name | Mobile | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_affiliate; public.dim_affiliate.affiliate_group_name |
| AN_024 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_category_name | varchar(480) | Affiliate category name | O&O | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_affiliate; public.dim_affiliate.affiliate_category_name |
| AN_025 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_id | int | Affiliate id | 11879 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_affiliate; public.dim_affiliate.affiliate_id |
| AN_026 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | channel | varchar(10) | Channel group | App | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_channel; public.dim_channel.channel_group |
| AN_027 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | channel_ungrouped | varchar(40) | Channel name | WAP | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_channel; public.dim_channel.channel_name |
| AN_028 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | filmid_child | int | Child movie id | 10293 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_id |
| AN_029 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | filmname_child | varchar(400) | Child movie title | Example Movie | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_title |
| AN_030 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | filmid_parent | varchar(32) | Parent movie id | 872 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_parent_id |
| AN_031 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | filmname_parent | varchar(400) | Parent movie title | Franchise X | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_parent_title |
| AN_032 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | releasedate | date | Release date | 2026-09-14 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_release_date |
| AN_033 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | movie_tms_id | int | Movie TMS id | 123456 | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_tms_id |
| AN_034 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | chainname | varchar(200) | Chain name | Example Cinemas | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_chain; public.dim_chain.chain_name |
| AN_035 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | chainid | varchar(10) | Chain id | CHN019 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_chain; public.dim_chain.chain_id |
| AN_036 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | pos_vendor | varchar(50) | POS vendor | Vista | Third-party |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_chain; public.dim_chain.point_of_sale_vendor |
| AN_037 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | awards_program_name | varchar(320) | Awards program | Rewards+ | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_chain; public.dim_chain.awards_program_name |
| AN_038 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | theater_id | int | Theater id | 4451 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.theater_id |
| AN_039 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | theater_code | varchar(10) | Theater code | TH4451 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.theater_code |
| AN_040 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | theater_name | varchar(200) | Theater name | Example Hollywood | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.theater_name |
| AN_041 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | pdi_id | varchar(6) | PDI id | AVSTA | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.pdi_id |
| AN_042 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | pos_theater_id | varchar(20) | POS theater id | POS-4451 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.point_of_sale_theater_id |
| AN_043 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | latitude | real | Latitude | 34.10 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.latitude |
| AN_044 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | longitude | real | Longitude | -118.34 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.longitude |
| AN_045 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | point_of_sale_id | smallint | POS id | 1 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.point_of_sale_id |
| AN_046 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | address_1 | varchar(200) | Address 1 | 123 Main St | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.address_1 |
| AN_047 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | address_2 | varchar(200) | Address 2 | Suite 200 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.address_2 |
| AN_048 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | city_name | varchar(160) | City | Los Angeles | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.city_name |
| AN_049 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | state_code | varchar(2) | State code | CA | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.state_code |
| AN_050 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | zip_code | varchar(10) | ZIP code | 90028 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.zip_code |
| AN_051 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | country_code | varchar(3) | Country code | US | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.country_code |
| AN_052 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | tht_alternate_theater_phone_name | varchar(200) | Alternate phone/name | Box Office | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.alternate_theater_phone_name |
| AN_053 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | chain_specific_site_id | varchar(10) | Chain site id | SITE-22 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.chain_specific_site_id |
| AN_054 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | paymentech_terminal_id | varchar(24) | Paymentech terminal id | PT-100 | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.paymentech_terminal_id |
| AN_055 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | cybersource_terminal_id | varchar(24) | Cybersource terminal id | CS-200 | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.cybersource_terminal_id |
| AN_056 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | screen_count | smallint | Screen count | 12 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.screen_count |
| AN_057 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | seat_count | int | Seat count | 1800 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.seat_count |
| AN_058 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | theater_time_zone_name | varchar(50) | Theater timezone | America/Los_Angeles | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.theater_time_zone_name |
| AN_060 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_type_id | varchar(40) | Transaction type id | Purchase | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_transaction_type; public.dim_transaction_type.transaction_type_id |
| AN_061 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_type_name | varchar(40) | Transaction type name | Ticket Purchase | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_transaction_type; public.dim_transaction_type.transaction_type_name |
| AN_062 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | user_agent_id | varchar(480) | User agent id | UA-1 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_user_agent; public.dim_user_agent.user_agent_id |
| AN_063 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | user_agent_description | varchar(480) | User agent description | iOS Safari | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_user_agent; public.dim_user_agent.user_agent_description |
| AN_064 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | purchase_funnel_id | varchar(40) | Funnel id | 14 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_purchase_funnel; public.dim_purchase_funnel.purchase_funnel_id |
| AN_065 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | purchase_funnel_name | varchar(100) | Funnel name | Guest Checkout | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_purchase_funnel; public.dim_purchase_funnel.purchase_funnel_name |
| AN_066 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | return_method | varchar(20) | Return method | Self-service | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_return_type; public.dim_return_type.return_method |
| AN_067 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | return_type_code | varchar(3) | Return type code | REF | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_return_type; public.dim_return_type.return_type_code |
| AN_068 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | return_type_description | varchar(20) | Return type description | Refund | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_return_type; public.dim_return_type.return_type_description |
| AN_088 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | property_id | varchar(60) | Property id | Fandango USA | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_property; public.dim_property.property_id |
| AN_090 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | braintree_merchant_id | varchar(60) | Merchant id | bt_merch_1 | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.braintree_merchant_id |
| AN_108 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | partner_channel | varchar | Partner channel id | TMO | Third-party |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_partner_channel; public.dim_partner_channel.partner_channel_id |
| AN_109 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_partner_channel; public.dim_partner_channel.partner_display_name |
| AN_110 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | partner_platform | varchar | Partner platform id | WEBVIEW | Third-party |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_partner_platform; public.dim_partner_platform.partner_platform_id |
| AN_112 | PUB_015 | analytics.transactions_data_source | 1 row / transaction_guid | property_group | varchar | Property group | Fandango | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_property; public.dim_property.property_group |


</details>
</details>

---

<details>
<summary><strong>Group E — Analytics-derived calcs and CASE buckets (11 fields)</strong></summary>

### Flow Diagram
```text
Raw transaction, lifecycle, and performance inputs
  -> Stage: transaction base (warehouse_loader.fact_transaction_base)
  -> Public: transaction-level fact (public.fact_transaction)
  -> Analytics: transaction-level time and category calculations

Side note: public dimensions supply lookup attributes; details are listed below.
```

<details>
<summary><strong>Raw</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| RAW_030 | raw_import_cache.transaction + raw_import_cache.transactionCommitted + raw_import_cache.transactionRolledBack | 1 row / tx | Create_dm timestamps | timestamp | Lifecycle UTC timestamps | 2026-10-01... | Backend |  | Inputs to timezone derivations. |
| RAW_031 | raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | Performance_dm | timestamp | Performance local timestamp | 2026-10-05 19:30:00 | Backend |  | Input to DATEDIFF metrics. |
| RAW_032 | raw_import_cache.transaction | 1 row / tx | Affiliate_id, Channel_cd | mixed | Inputs to CASE bucketing | 11879/WAP | Backend |  | Feed dim-based CASE logic. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| STG_029 | RAW_030, RAW_031, RAW_032, RAW_026, RAW_036 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_* timestamps, performance_datetime_local, affiliate/channel/movie keys | mixed | Inputs to analytics derivations | — | Backend/ETL |  | `[passthrough]` Carries timestamps and performance values from raw_import_cache.transaction, raw_import_cache.transactionCommitted, raw_import_cache.transactionRolledBack, and raw_import_cache.transactionPerformanceAttribute; keys are resolved against public.dim_affiliate, public.dim_channel, public.dim_movie, and public.dim_theater. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| PUB_016 | STG_029 | public.fact_transaction | 1 row / transaction_guid | lifecycle timestamps + performance_datetime_local | timestamp | Inputs for timezone/delta calculations | — | Backend/Derived |  | `[passthrough]` Reads lifecycle and performance fields from public.fact_transaction; that table gets them from warehouse_loader.fact_transaction_base. |
| PUB_017 | RAW_036 | public.dim_movie / dim_affiliate / dim_channel | dim grain | movie_release_date / affiliate_id / channel_name | mixed | Inputs to DATEDIFF and CASE logic | — | Backend |  | `[lookup]` Reads attributes from public.dim_movie, public.dim_affiliate, and public.dim_channel for analytics calculations. |

</details>

<details open>
<summary><strong>Analytics (11 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| AN_004 | PUB_016 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_start_datetime_los_angeles | timestamp | Pacific start datetime | 2026-10-01 09:10:00 | Derived |  | `[derived]` Reads public.fact_transaction; CONVERT_TIMEZONE('US/Pacific', start_utc) |
| AN_006 | PUB_016 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_commit_datetime_los_angeles | timestamp | Pacific commit datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` Reads public.fact_transaction; timezone conversion |
| AN_008 | PUB_016 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_rollback_datetime_los_angeles | timestamp | Pacific rollback datetime | NULL | Derived |  | `[derived]` Reads public.fact_transaction; timezone conversion |
| AN_010 | PUB_016 | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_los_angeles | timestamp | Pacific end datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` Reads public.fact_transaction; timezone conversion |
| AN_012 | PUB_016, PUB_017 | analytics.transactions_data_source | 1 row / transaction_guid | deltareleasetopurchasedate | int | Days release->purchase day | 17 | Derived |  | `[derived]` Reads public.fact_transaction; calculates days from public.dim_movie.movie_release_date to the Pacific transaction date. |
| AN_013 | PUB_016, PUB_017 | analytics.transactions_data_source | 1 row / transaction_guid | deltareleasetoshowtimedate | int | Days release->showtime day | 21 | Derived |  | `[derived]` Reads public.fact_transaction; calculates days from public.dim_movie.movie_release_date to public.fact_transaction.performance_datetime_local. |
| AN_014 | PUB_016 | analytics.transactions_data_source | 1 row / transaction_guid | deltapurchasetoshowtimedate | int | Days purchase day->showtime day | 4 | Derived |  | `[derived]` Reads public.fact_transaction; calculates days between the Pacific transaction date and performance date in public.fact_transaction. |
| AN_069 | PUB_017 | analytics.transactions_data_source | 1 row / transaction_guid | "Affiliate Grouping" | varchar(256) | Detailed affiliate bucket | iPhone App | Derived |  | `[conditional]` Reads public.fact_transaction, public.dim_affiliate; uses public.dim_affiliate.affiliate_id in a CASE expression to assign the detailed affiliate bucket, including NULL handling. |
| AN_070 | PUB_017 | analytics.transactions_data_source | 1 row / transaction_guid | "Affiliate Grouping - HIGH LEVEL BINS" | varchar(256) | High-level bucket | Fandango O&O | Derived |  | `[conditional]` Reads public.fact_transaction, public.dim_affiliate; uses public.dim_affiliate.affiliate_id in a CASE expression to assign the high-level affiliate bucket. |
| AN_071 | PUB_017 | analytics.transactions_data_source | 1 row / transaction_guid | apps | varchar(256) | App bucket | Fandango App: iPhone | Derived |  | `[conditional]` Reads public.fact_transaction, public.dim_affiliate; uses public.dim_affiliate.affiliate_id in a CASE expression to assign the app bucket. |
| AN_072 | PUB_017 | analytics.transactions_data_source | 1 row / transaction_guid | platform | varchar(256) | Platform bucket | Mobile Apps | Derived |  | `[conditional]` Reads public.fact_transaction, public.dim_affiliate, public.dim_channel; uses public.dim_affiliate.affiliate_id and public.dim_channel.channel_name in a CASE expression; checks Platform first. |

</details>
</details>

---

<details>
<summary><strong>Group F — Raw enrichment at analytics step (1 field)</strong></summary>

### Flow Diagram
```text
Raw transaction and theater identifiers
  -> Stage: transaction base (warehouse_loader.fact_transaction_base)
  -> Public: transaction-level fact (public.fact_transaction)
  -> Analytics: transaction-level DMA enrichment

Side note: ZIP-to-DMA lookup data supplies the enrichment; details are listed below.
```

<details>
<summary><strong>Raw</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| RAW_033 | raw_import_cache.designatedMarketArea | 1 row / zip_code | zip_code, market_name | varchar | ZIP to DMA mapping | 90028 / Los Angeles | Third-party |  | External DMA source used in analytics join. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| STG_030 | RAW_026 | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | theater_key | bigint | Theater lookup key | 11872 | ETL |  | `[lookup]` Resolves theater_pkey from raw_import_cache.transactionPerformanceAttribute against public.dim_theater. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| PUB_018 | STG_030 | public.dim_theater | 1 row / theater_key | zip_code | varchar(10) | Theater ZIP code | 90028 | Backend |  | `[lookup]` Reads ZIP from public.dim_theater; analytics joins it to raw_import_cache.designatedMarketArea by ZIP. |

</details>

<details open>
<summary><strong>Analytics (1 field)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| AN_059 | PUB_018 | analytics.transactions_data_source | 1 row / transaction_guid | dma_market_name | varchar(255) | DMA market name | Los Angeles | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_theater, raw_import_cache.designatedMarketArea; joins public.dim_theater.zip_code to raw_import_cache.designatedMarketArea.zip_code and selects market_name. |

</details>
</details>

---

<details>
<summary><strong>Group G — Sales-tax enrichment (9 fields)</strong></summary>

### Flow Diagram
```text
Raw (data_lake.salestaxtransaction)
  -> Stage (raw_import_cache.stgsalestaxtransaction)
  -> Public: transaction-level tax fact (public.fact_sales_fee_tax)
  -> Analytics (analytics.transactions_data_source)

Side note: recent-window refreshes use a temporary snapshot.
```

<details>
<summary><strong>Raw</strong></summary>

| Field ID | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|
| RAW_034 | data_lake.salestaxtransaction | line-item grain | document_number, lineitem_number, total_lineitem_tax, tax area/rate and status fields | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| STG_031 | RAW_034 | raw_import_cache.stgsalestaxtransaction | line-item staging grain | document_number, lineitem_number, total_lineitem_tax, tax area/rate and status fields | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| STG_032 | STG_031 | temp #transactionSalesTax | 1 row / transaction_guid | sales_fee_tax_amount, returned_sales_fee_tax_amount, sales_fee_tax_return_amount, sales_fee_tax_amount_for_returned_purchase, pr_ticket_tax_amount, returned_ticket_tax_amount, ticket_tax_return_amount, ticket_tax_amount_for_returned_purchase | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| PUB_019 | STG_031 | public.fact_sales_fee_tax | 1 row / transaction_guid | sales_fee_tax_amount, returned_sales_fee_tax_amount, sales_fee_tax_return_amount, sales_fee_tax_amount_for_returned_purchase, pr_ticket_tax_amount, returned_ticket_tax_amount, ticket_tax_return_amount, ticket_tax_amount_for_returned_purchase | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |

</details>

<details open>
<summary><strong>Analytics (9 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
|---|---|---|---|---|---|---|---|---|---|---|
| AN_092 | PUB_019 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_amount | decimal(19,4) | Sales fee tax amount | 0.35 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies sales_fee_tax_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_093 | PUB_019 | analytics.transactions_data_source | 1 row / transaction_guid | returned_sales_fee_tax_amount | decimal(19,4) | Returned sales fee tax amount | -0.35 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies returned_sales_fee_tax_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_094 | PUB_019 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_return_amount | decimal(19,4) | Sales fee tax return amount | -0.35 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies sales_fee_tax_return_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_095 | PUB_019 | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_amount_for_returned_purchase | decimal(19,4) | Sales fee tax amount for returned purchase | 0.35 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies sales_fee_tax_amount_for_returned_purchase from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_124 | PUB_019 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_amount | decimal(19,4) | Vertex ticket tax amount | 3.50 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction, public.fact_sales_fee_tax; renames public.fact_sales_fee_tax.pr_ticket_tax_amount to ticket_tax_amount. |
| AN_125 | PUB_019, PUB_010 | analytics.transactions_data_source | 1 row / transaction_guid | pre_tax_ticket_sales_amount | decimal(19,4) | Ticket sales amount before tax | 31.50 | Derived |  | `[derived]` Reads public.fact_transaction, public.fact_sales_fee_tax; calculates public.fact_transaction.ticket_sales_amount minus public.fact_sales_fee_tax.pr_ticket_tax_amount, using zero when tax is missing. |
| AN_126 | PUB_019 | analytics.transactions_data_source | 1 row / transaction_guid | returned_ticket_tax_amount | decimal(19,4) | Returned ticket tax amount | -3.50 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies returned_ticket_tax_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_127 | PUB_019 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_return_amount | decimal(19,4) | Ticket tax return amount | -3.50 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies ticket_tax_return_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_128 | PUB_019 | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_amount_for_returned_purchase | decimal(19,4) | Ticket tax amount for returned purchase | 3.50 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies ticket_tax_amount_for_returned_purchase from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |


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

## Quick Business Crosswalk

- **Group A:** direct pass-through style fields standardized in staging/public before analytics.
- **Group B:** aggregated commercial metrics from item/payment aggregates.
- **Group C:** return/refund derived fields and sync behavior.
- **Group D:** descriptive dim attributes projected from conformed keys.
- **Group E:** pure analytics-layer computations and CASE bucketing.
- **Group F:** targeted raw lookup enrichment (DMA by ZIP).
- **Group G:** post-insert sales-tax enrichment from salestax pipeline.

---


