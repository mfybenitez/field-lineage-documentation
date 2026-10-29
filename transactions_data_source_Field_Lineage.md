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
- **Consistent field IDs**: `AN_001` … `AN_128` identify analytics fields; `RAW_###`, `STG_###`, and `PUB_###` identify Raw, Staging, and Public lineage fields. Every row has exactly one Field ID. A Field ID may appear again in another row when its fields have different predecessor IDs.
- **Previous Stage ID(s)** in the Staging rows follow the transaction-level Raw → Stage → Public → Analytics overview. When a staging aggregate reads a lower-grain `public` fact internally, its raw/stage ancestry is shown in the ID cell and the physical `public` input is named in Logic Explanation. Public and Analytics rows continue to point to their immediate preceding lineage IDs. For Groups A and B, Raw columns are grouped only when they come from the same source table. Each Column Name cell contains one column name. Group repeated references to the same column in the same table into one row and keep one Field ID in that row. List multiple Previous Stage IDs only when that field logic uses multiple direct predecessors.
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
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="raw-001"></a>RAW_001 | raw_import_cache.transaction | 1 row / transaction_guid | Transaction_GUID | mixed | Transaction identifier | — | Backend |  | Base transaction key. |
| <a id="raw-002"></a>RAW_002 | raw_import_cache.transaction | 1 row / transaction_guid | Create_dm | mixed | Start timestamp | — | Backend |  | Source for start datetime. |
| <a id="raw-010"></a>RAW_003 | raw_import_cache.transaction | 1 row / transaction_guid | test_bt | mixed | Test marker | — | Backend |  | One input to is_test; the CASE also checks customer email, test-email tables/patterns, theater PDI, and transaction type. |
| <a id="raw-012"></a>RAW_004 | raw_import_cache.transaction | 1 row / transaction_guid | DeviceType_cd | mixed | Device type | — | Backend |  | Source for devicetype. |
| <a id="raw-020"></a>RAW_005 | raw_import_cache.transaction | 1 row / transaction_guid | ticketTaxRate_rt | mixed | PR ticket tax rate | — | Backend |  | Source for ticket_tax_rate. |
| <a id="raw-021"></a>RAW_006 | raw_import_cache.transaction | 1 row / transaction_guid | ticketTaxTransaction_bt | mixed | PR ticket-tax flag | — | Backend |  | Source for is_ticket_tax_transaction. |
| <a id="raw-003"></a>RAW_007 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Create_dm | mixed | Commit timestamp | — | Backend |  | Source for commit/end logic. |
| <a id="raw-006"></a>RAW_008 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Confirmation_cd | mixed | POS confirmation code | — | Backend |  | Source for point_of_sale_confirmation_code. |
| <a id="raw-007"></a>RAW_009 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | ReservedSeats_tx | mixed | Reserved seat payload | — | Backend |  | Source for reserved seats outputs. |
| <a id="raw-008"></a>RAW_010 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Loyalty_cd | mixed | Loyalty code | — | Backend |  | Source for loyalty_code. |
| <a id="raw-009"></a>RAW_011 | raw_import_cache.transactionCommitted | 1 row / transaction_guid | POS_reference_cd | mixed | POS reference code | — | Backend |  | Source for pos_reference_cd. |
| <a id="raw-004"></a>RAW_012 | raw_import_cache.transactionRolledBack | 1 row / transaction_guid | Create_dm | timestamp | Rollback timestamp | NULL | Backend |  | **RAW_012**: Source for rollback/end logic. |
| <a id="raw-005"></a>RAW_013 | raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | Performance_dm | timestamp | Performance local datetime | 2026-10-05 19:30:00 | Backend |  | **RAW_013**: Source for performance_datetime_local. |
| <a id="raw-011"></a>RAW_014 | raw_import_cache.transactionCustomer + raw_import_cache.testEmailAddress + raw_import_cache.testTransactionEmailList | transaction/customer or email row | Email_nm | mixed | Test classification inputs | — | Backend |  | **RAW_014**: Used with transaction.test_bt to derive is_test. |
| RAW_014 | raw_import_cache.transactionCustomer + raw_import_cache.testEmailAddress + raw_import_cache.testTransactionEmailList | transaction/customer or email row | email_tx | mixed | Test classification inputs | — | Backend |  | **RAW_014**: Used with transaction.test_bt to derive is_test. |
| RAW_014 | raw_import_cache.transactionCustomer + raw_import_cache.testEmailAddress + raw_import_cache.testTransactionEmailList | transaction/customer or email row | activation | mixed | Test classification inputs | — | Backend |  | **RAW_014**: Used with transaction.test_bt to derive is_test. |
| RAW_014 | raw_import_cache.transactionCustomer + raw_import_cache.testEmailAddress + raw_import_cache.testTransactionEmailList | transaction/customer or email row | expiration | mixed | Test classification inputs | — | Backend |  | **RAW_014**: Used with transaction.test_bt to derive is_test. |
| RAW_014 | raw_import_cache.transactionCustomer + raw_import_cache.testEmailAddress + raw_import_cache.testTransactionEmailList | transaction/customer or email row | active | mixed | Test classification inputs | — | Backend |  | **RAW_014**: Used with transaction.test_bt to derive is_test. |
| RAW_014 | raw_import_cache.transactionCustomer + raw_import_cache.testEmailAddress + raw_import_cache.testTransactionEmailList | transaction/customer or email row | pattern flags | mixed | Test classification inputs | — | Backend |  | **RAW_014**: Used with transaction.test_bt to derive is_test. |
| <a id="raw-013"></a>RAW_015 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (FanClubStatus) | mixed | Fanclub status attr | — | Backend |  | Input to is_fanclub. |
| <a id="raw-014"></a>RAW_016 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (IsBuyNowOrder) | mixed | Buy-now attr | — | Backend |  | Input to is_buynow. |
| <a id="raw-015"></a>RAW_017 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (CharityName) | mixed | Charity name attr | — | Backend |  | Source for charity_name. |
| <a id="raw-016"></a>RAW_018 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (PartnerDefinedData) | mixed | Partner-defined attr | — | Third-party |  | Source for partner_defined_data_value. |
| <a id="raw-017"></a>RAW_019 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (networktoken) | mixed | Network token attr | — | Backend |  | Input to network_token_usage. |
| <a id="raw-019"></a>RAW_020 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (UtmSource) | mixed | UTM attrs | — | Third-party |  | Source for utm_* fields. |
| RAW_020 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (UtmMedium) | mixed | UTM attrs | — | Third-party |  | Source for utm_* fields. |
| RAW_020 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (UtmCampaign) | mixed | UTM attrs | — | Third-party |  | Source for utm_* fields. |
| RAW_020 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (UtmContent) | mixed | UTM attrs | — | Third-party |  | Source for utm_* fields. |
| RAW_020 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (UtmTerm) | mixed | UTM attrs | — | Third-party |  | Source for utm_* fields. |
| <a id="raw-018"></a>RAW_021 | raw_import_cache.transactionPerformanceAttribute + analytics_manual.plf_amenity_mapping | many amenity IDs / transaction | amenity_ids | mixed | Amenity flag lookup inputs | 3,10 | Backend/Derived |  | **RAW_021**: Matches each amenity_id in analytics_manual.plf_amenity_mapping to the comma-delimited amenity_ids in raw_import_cache.transactionPerformanceAttribute; rolls matching PLF, IMAX, 3D, and higher-fee flags up to transaction_guid. |
| RAW_021 | raw_import_cache.transactionPerformanceAttribute + analytics_manual.plf_amenity_mapping | many amenity IDs / transaction | amenity_id | mixed | Amenity flag lookup inputs | 3,10 | Backend/Derived |  | **RAW_021**: Matches each amenity_id in analytics_manual.plf_amenity_mapping to the comma-delimited amenity_ids in raw_import_cache.transactionPerformanceAttribute; rolls matching PLF, IMAX, 3D, and higher-fee flags up to transaction_guid. |
| RAW_021 | raw_import_cache.transactionPerformanceAttribute + analytics_manual.plf_amenity_mapping | many amenity IDs / transaction | is_imax | mixed | Amenity flag lookup inputs | 3,10 | Backend/Derived |  | **RAW_021**: Matches each amenity_id in analytics_manual.plf_amenity_mapping to the comma-delimited amenity_ids in raw_import_cache.transactionPerformanceAttribute; rolls matching PLF, IMAX, 3D, and higher-fee flags up to transaction_guid. |
| RAW_021 | raw_import_cache.transactionPerformanceAttribute + analytics_manual.plf_amenity_mapping | many amenity IDs / transaction | is_3d | mixed | Amenity flag lookup inputs | 3,10 | Backend/Derived |  | **RAW_021**: Matches each amenity_id in analytics_manual.plf_amenity_mapping to the comma-delimited amenity_ids in raw_import_cache.transactionPerformanceAttribute; rolls matching PLF, IMAX, 3D, and higher-fee flags up to transaction_guid. |
| RAW_021 | raw_import_cache.transactionPerformanceAttribute + analytics_manual.plf_amenity_mapping | many amenity IDs / transaction | is_higher_fee_amenity | mixed | Amenity flag lookup inputs | 3,10 | Backend/Derived |  | **RAW_021**: Matches each amenity_id in analytics_manual.plf_amenity_mapping to the comma-delimited amenity_ids in raw_import_cache.transactionPerformanceAttribute; rolls matching PLF, IMAX, 3D, and higher-fee flags up to transaction_guid. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-001"></a>STG_001 | [RAW_001](#raw-001) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction id | 9F2A... | Backend |  | `[passthrough]` Copies transaction_guid from raw_import_cache.transaction. |
| <a id="stg-002"></a>STG_002 | [RAW_007](#raw-007), [RAW_012](#raw-012) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_successful | boolean | Success flag | true | Derived |  | `[conditional]` Sets success when raw_import_cache.transactionCommitted has a row and raw_import_cache.transactionRolledBack does not. |
| <a id="stg-003"></a>STG_003 | [RAW_002](#raw-002) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_start_datetime | timestamp | Start UTC | 2026-10-01 16:10:00 | Backend |  | `[rename/cast]` Casts raw_import_cache.transaction.Create_dm as transaction_start_datetime. |
| <a id="stg-004"></a>STG_004 | [RAW_007](#raw-007) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_commit_datetime | timestamp | Commit UTC | 2026-10-01 16:11:05 | Backend |  | `[rename/cast]` Copies Create_dm from raw_import_cache.transactionCommitted. |
| <a id="stg-005"></a>STG_005 | [RAW_012](#raw-012) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_rollback_datetime | timestamp | Rollback UTC | NULL | Backend |  | `[rename/cast]` Copies Create_dm from raw_import_cache.transactionRolledBack. |
| <a id="stg-006"></a>STG_006 | [RAW_002](#raw-002), [RAW_007](#raw-007), [RAW_012](#raw-012) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_end_datetime | timestamp | End UTC | 2026-10-01 16:11:05 | Derived |  | `[conditional]` Chooses the latest available timestamp from raw_import_cache.transaction, raw_import_cache.transactionCommitted, and raw_import_cache.transactionRolledBack; falls back to the transaction start time. |
| <a id="stg-007"></a>STG_007 | [RAW_013](#raw-013) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | performance_datetime_local | timestamp | Local performance datetime | 2026-10-05 19:30:00 | Backend |  | `[passthrough]` Copies Performance_dm from raw_import_cache.transactionPerformanceAttribute. |
| <a id="stg-008"></a>STG_008 | [RAW_008](#raw-008) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation | CNF908812 | Backend |  | `[rename/cast]` Copies Confirmation_cd from raw_import_cache.transactionCommitted. |
| <a id="stg-009"></a>STG_009 | [RAW_009](#raw-009) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_reserved_seating | mixed | Reserved seating flag | true | Derived |  | `[conditional]` Sets the seating flag when ReservedSeats_tx from raw_import_cache.transactionCommitted is nonblank. |
| <a id="stg-010"></a>STG_010 | [RAW_009](#raw-009) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | reserved_seats | mixed | Reserved seats | A10,A11 | Backend |  | `[conditional]` Trims ReservedSeats_tx from raw_import_cache.transactionCommitted; empty text becomes NULL. |
| <a id="stg-011"></a>STG_011 | [RAW_010](#raw-010) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code | LOY123 | Backend |  | `[conditional]` Trims Loyalty_cd from raw_import_cache.transactionCommitted; empty text becomes NULL. |
| <a id="stg-012"></a>STG_012 | [RAW_011](#raw-011) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | POS_reference_cd | varchar(40) | POS ref code | POSREF1 | Backend |  | `[passthrough]` Copies POS_reference_cd from raw_import_cache.transactionCommitted. |
| <a id="stg-013"></a>STG_013 | [RAW_003](#raw-003), [RAW_014](#raw-014), [RAW_027](#raw-027), [RAW_028](#raw-028) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_test | boolean | Test flag | false | Derived |  | `[conditional]` Derives is_test from raw_import_cache.transaction, raw_import_cache.transactionCustomer, raw_import_cache.testEmailAddress, and raw_import_cache.testTransactionEmailList; also checks public.dim_theater and public.dim_transaction_type. |
| <a id="stg-014"></a>STG_014 | [RAW_004](#raw-004) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | deviceType | varchar | Device type | Mobile | Backend |  | `[rename/cast]` Copies DeviceType_cd from raw_import_cache.transaction as deviceType. |
| <a id="stg-015"></a>STG_015 | [RAW_015](#raw-015) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived |  | `[conditional]` Maps listed active FanClubStatus values from raw_import_cache.transactionAttribute to true. |
| <a id="stg-016"></a>STG_016 | [RAW_016](#raw-016) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived |  | `[conditional]` Sets the flag when IsBuyNowOrder in raw_import_cache.transactionAttribute is true. |
| <a id="stg-017"></a>STG_017 | [RAW_017](#raw-017) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | charity_name | varchar(256) | Charity name | Save The Arts | Backend |  | `[lookup]` Reads CharityName from raw_import_cache.transactionAttribute. |
| <a id="stg-018"></a>STG_018 | [RAW_018](#raw-018) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner-defined payload | campaign=fall | Third-party |  | `[lookup]` Reads PartnerDefinedData from raw_import_cache.transactionAttribute. |
| <a id="stg-019"></a>STG_019 | [RAW_019](#raw-019) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | network_token_usage | boolean | Network token usage | true | Derived |  | `[conditional]` Reads NetworkToken from raw_import_cache.transactionAttribute; for refunds, raw_import_cache.refund identifies the parent transaction whose attribute is the fallback. |
| <a id="stg-020"></a>STG_020 | [RAW_021](#raw-021) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_plf | boolean | Amenity flags | false | Derived |  | `[aggregate]` Matches amenity_ids from raw_import_cache.transactionPerformanceAttribute to analytics_manual.plf_amenity_mapping and groups flags by raw_import_cache.transaction.transaction_guid. |
| STG_020 | [RAW_021](#raw-021) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_imax | boolean | Amenity flags | false | Derived |  | `[aggregate]` Matches amenity_ids from raw_import_cache.transactionPerformanceAttribute to analytics_manual.plf_amenity_mapping and groups flags by raw_import_cache.transaction.transaction_guid. |
| STG_020 | [RAW_021](#raw-021) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_3d | boolean | Amenity flags | false | Derived |  | `[aggregate]` Matches amenity_ids from raw_import_cache.transactionPerformanceAttribute to analytics_manual.plf_amenity_mapping and groups flags by raw_import_cache.transaction.transaction_guid. |
| STG_020 | [RAW_021](#raw-021) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_higher_fee_amenity | boolean | Amenity flags | false | Derived |  | `[aggregate]` Matches amenity_ids from raw_import_cache.transactionPerformanceAttribute to analytics_manual.plf_amenity_mapping and groups flags by raw_import_cache.transaction.transaction_guid. |
| <a id="stg-021"></a>STG_021 | [RAW_020](#raw-020) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_source | varchar(255) | UTM params | google/cpc/... | Backend |  | `[lookup]` Joins five named UTM attributes from raw_import_cache.transactionAttribute by transaction GUID. |
| STG_021 | [RAW_020](#raw-020) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_medium | varchar(255) | UTM params | google/cpc/... | Backend |  | `[lookup]` Joins five named UTM attributes from raw_import_cache.transactionAttribute by transaction GUID. |
| STG_021 | [RAW_020](#raw-020) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_campaign | varchar(255) | UTM params | google/cpc/... | Backend |  | `[lookup]` Joins five named UTM attributes from raw_import_cache.transactionAttribute by transaction GUID. |
| STG_021 | [RAW_020](#raw-020) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_content | varchar(255) | UTM params | google/cpc/... | Backend |  | `[lookup]` Joins five named UTM attributes from raw_import_cache.transactionAttribute by transaction GUID. |
| STG_021 | [RAW_020](#raw-020) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_term | varchar(255) | UTM params | google/cpc/... | Backend |  | `[lookup]` Joins five named UTM attributes from raw_import_cache.transactionAttribute by transaction GUID. |
| <a id="stg-022"></a>STG_022 | [RAW_005](#raw-005) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | PR ticket tax rate | 11.5000 | Backend |  | `[rename/cast]` Copies ticketTaxRate_rt from raw_import_cache.transaction. |
| <a id="stg-023"></a>STG_023 | [RAW_006](#raw-006) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | PR ticket-tax txn flag | true | Derived |  | `[conditional]` Copies ticketTaxTransaction_bt from raw_import_cache.transaction; NULL defaults to false. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-001"></a>PUB_001 | [STG_001](#stg-001) | public.fact_transaction | 1 row / transaction_guid | transaction_guid | varchar(36) | Lifecycle fields | — | Backend/Derived |  | `[passthrough]` Copies lifecycle fields from warehouse_loader.fact_transaction_base into public.fact_transaction. |
| PUB_001 | [STG_002](#stg-002) | public.fact_transaction | 1 row / transaction_guid | is_successful | boolean | Lifecycle fields | — | Backend/Derived |  | `[passthrough]` Copies lifecycle fields from warehouse_loader.fact_transaction_base into public.fact_transaction. |
| PUB_001 | [STG_003](#stg-003) | public.fact_transaction | 1 row / transaction_guid | transaction_start_datetime | timestamp | Lifecycle fields | — | Backend/Derived |  | `[passthrough]` Copies lifecycle fields from warehouse_loader.fact_transaction_base into public.fact_transaction. |
| PUB_001 | [STG_004](#stg-004) | public.fact_transaction | 1 row / transaction_guid | transaction_commit_datetime | timestamp | Lifecycle fields | — | Backend/Derived |  | `[passthrough]` Copies lifecycle fields from warehouse_loader.fact_transaction_base into public.fact_transaction. |
| PUB_001 | [STG_005](#stg-005) | public.fact_transaction | 1 row / transaction_guid | transaction_rollback_datetime | timestamp | Lifecycle fields | — | Backend/Derived |  | `[passthrough]` Copies lifecycle fields from warehouse_loader.fact_transaction_base into public.fact_transaction. |
| PUB_001 | [STG_006](#stg-006) | public.fact_transaction | 1 row / transaction_guid | transaction_end_datetime | timestamp | Lifecycle fields | — | Backend/Derived |  | `[passthrough]` Copies lifecycle fields from warehouse_loader.fact_transaction_base into public.fact_transaction. |
| <a id="pub-002"></a>PUB_002 | [STG_006](#stg-006), [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | transaction_end_datetime_local | timestamp | End local datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` Converts warehouse_loader.fact_transaction_base.transaction_end_datetime using public.dim_theater.theater_time_zone_name. |
| PUB_003 | [STG_007](#stg-007) | public.fact_transaction | 1 row / transaction_guid | performance_datetime_local | timestamp | Core transaction attrs | — | Backend/Derived |  | `[passthrough]` Copies confirmation, seating, loyalty, and performance fields from warehouse_loader.fact_transaction_base. |
| <a id="pub-003"></a>PUB_003 | [STG_008](#stg-008) | public.fact_transaction | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | Core transaction attrs | — | Backend/Derived |  | `[passthrough]` Copies confirmation, seating, loyalty, and performance fields from warehouse_loader.fact_transaction_base. |
| PUB_003 | [STG_009](#stg-009) | public.fact_transaction | 1 row / transaction_guid | is_reserved_seating | boolean | Core transaction attrs | — | Backend/Derived |  | `[passthrough]` Copies confirmation, seating, loyalty, and performance fields from warehouse_loader.fact_transaction_base. |
| PUB_003 | [STG_010](#stg-010) | public.fact_transaction | 1 row / transaction_guid | reserved_seats | varchar(100) | Core transaction attrs | — | Backend/Derived |  | `[passthrough]` Copies confirmation, seating, loyalty, and performance fields from warehouse_loader.fact_transaction_base. |
| PUB_003 | [STG_011](#stg-011) | public.fact_transaction | 1 row / transaction_guid | loyalty_code | varchar(40) | Core transaction attrs | — | Backend/Derived |  | `[passthrough]` Copies confirmation, seating, loyalty, and performance fields from warehouse_loader.fact_transaction_base. |
| PUB_003 | [STG_012](#stg-012) | public.fact_transaction | 1 row / transaction_guid | POS_reference_cd | varchar(40) | Core transaction attrs | — | Backend/Derived |  | `[passthrough]` Copies confirmation, seating, loyalty, and performance fields from warehouse_loader.fact_transaction_base. |
| <a id="pub-004"></a>PUB_004 | [STG_013](#stg-013) | public.fact_transaction | 1 row / transaction_guid | is_test | boolean | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` Copies test, device, fanclub, buy-now, charity, partner, and network-token fields from warehouse_loader.fact_transaction_base. |
| PUB_004 | [STG_014](#stg-014) | public.fact_transaction | 1 row / transaction_guid | deviceType | varchar | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` Copies test, device, fanclub, buy-now, charity, partner, and network-token fields from warehouse_loader.fact_transaction_base. |
| PUB_004 | [STG_015](#stg-015) | public.fact_transaction | 1 row / transaction_guid | is_fanclub | boolean | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` Copies test, device, fanclub, buy-now, charity, partner, and network-token fields from warehouse_loader.fact_transaction_base. |
| PUB_004 | [STG_016](#stg-016) | public.fact_transaction | 1 row / transaction_guid | is_buynow | boolean | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` Copies test, device, fanclub, buy-now, charity, partner, and network-token fields from warehouse_loader.fact_transaction_base. |
| PUB_004 | [STG_017](#stg-017) | public.fact_transaction | 1 row / transaction_guid | charity_name | varchar(256) | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` Copies test, device, fanclub, buy-now, charity, partner, and network-token fields from warehouse_loader.fact_transaction_base. |
| PUB_004 | [STG_018](#stg-018) | public.fact_transaction | 1 row / transaction_guid | partner_defined_data_value | varchar | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` Copies test, device, fanclub, buy-now, charity, partner, and network-token fields from warehouse_loader.fact_transaction_base. |
| PUB_004 | [STG_019](#stg-019) | public.fact_transaction | 1 row / transaction_guid | network_token_usage | boolean | Behavior/attr outputs | — | Backend/Derived |  | `[passthrough]` Copies test, device, fanclub, buy-now, charity, partner, and network-token fields from warehouse_loader.fact_transaction_base. |
| <a id="pub-005"></a>PUB_005 | [STG_020](#stg-020) | public.fact_transaction | 1 row / transaction_guid | is_plf | boolean | Amenity flags | false | Derived |  | `[passthrough]` Copies amenity flags from warehouse_loader.fact_transaction_base. |
| PUB_005 | [STG_020](#stg-020) | public.fact_transaction | 1 row / transaction_guid | is_imax | boolean | Amenity flags | false | Derived |  | `[passthrough]` Copies amenity flags from warehouse_loader.fact_transaction_base. |
| PUB_005 | [STG_020](#stg-020) | public.fact_transaction | 1 row / transaction_guid | is_3d | boolean | Amenity flags | false | Derived |  | `[passthrough]` Copies amenity flags from warehouse_loader.fact_transaction_base. |
| PUB_005 | [STG_020](#stg-020) | public.fact_transaction | 1 row / transaction_guid | is_higher_fee_amenity | boolean | Amenity flags | false | Derived |  | `[passthrough]` Copies amenity flags from warehouse_loader.fact_transaction_base. |
| PUB_006 | [STG_021](#stg-021) | public.fact_transaction | 1 row / transaction_guid | utm_source | varchar(255) | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` Copies UTM and ticket-tax fields from warehouse_loader.fact_transaction_base. |
| PUB_006 | [STG_021](#stg-021) | public.fact_transaction | 1 row / transaction_guid | utm_medium | varchar(255) | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` Copies UTM and ticket-tax fields from warehouse_loader.fact_transaction_base. |
| PUB_006 | [STG_021](#stg-021) | public.fact_transaction | 1 row / transaction_guid | utm_campaign | varchar(255) | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` Copies UTM and ticket-tax fields from warehouse_loader.fact_transaction_base. |
| PUB_006 | [STG_021](#stg-021) | public.fact_transaction | 1 row / transaction_guid | utm_content | varchar(255) | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` Copies UTM and ticket-tax fields from warehouse_loader.fact_transaction_base. |
| PUB_006 | [STG_021](#stg-021) | public.fact_transaction | 1 row / transaction_guid | utm_term | varchar(255) | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` Copies UTM and ticket-tax fields from warehouse_loader.fact_transaction_base. |
| <a id="pub-006"></a>PUB_006 | [STG_022](#stg-022) | public.fact_transaction | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` Copies UTM and ticket-tax fields from warehouse_loader.fact_transaction_base. |
| PUB_006 | [STG_023](#stg-023) | public.fact_transaction | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | PR tax + UTM fields | — | Backend/Derived |  | `[passthrough]` Copies UTM and ticket-tax fields from warehouse_loader.fact_transaction_base. |

</details>

<details open>
<summary><strong>Analytics (31 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_001 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction id | 9F2A... | Backend |  | `[passthrough]` Reads public.fact_transaction; public.fact_transaction.transaction_guid |
| AN_002 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | is_successful | boolean | Success flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; public.fact_transaction.is_successful |
| AN_003 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_start_datetime_utc | timestamp | Start UTC | 2026-10-01 16:10:00 | Backend |  | `[rename/cast]` Reads public.fact_transaction; public.fact_transaction.transaction_start_datetime |
| AN_005 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_commit_datetime_utc | timestamp | Commit UTC | 2026-10-01 16:11:05 | Backend |  | `[rename/cast]` Reads public.fact_transaction; public.fact_transaction.transaction_commit_datetime |
| AN_007 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_rollback_datetime_utc | timestamp | Rollback UTC | NULL | Backend |  | `[rename/cast]` Reads public.fact_transaction; public.fact_transaction.transaction_rollback_datetime |
| AN_009 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_utc | timestamp | End UTC | 2026-10-01 16:11:05 | Derived |  | `[rename/cast]` Reads public.fact_transaction; public.fact_transaction.transaction_end_datetime |
| AN_011 | [PUB_002](#pub-002) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_local | timestamp | End local | 2026-10-01 09:11:05 | Derived |  | `[passthrough]` Reads public.fact_transaction; public.fact_transaction.transaction_end_datetime_local |
| AN_015 | [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation code | CNF908812 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies point_of_sale_confirmation_code unchanged. |
| AN_016 | [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_reserved_seating unchanged. |
| AN_017 | [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | reserved_seats | varchar(100) | Reserved seats | A10,A11 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies reserved_seats unchanged. |
| AN_018 | [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code | LOY123 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies loyalty_code unchanged. |
| AN_019 | [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | performance_datetime_local | timestamp | Performance local | 2026-10-05 19:30:00 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies performance_datetime_local unchanged. |
| AN_089 | [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | pos_reference_cd | varchar(40) | POS reference code | POSREF1 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies pos_reference_cd unchanged. |
| AN_096 | [PUB_004](#pub-004) | analytics.transactions_data_source | 1 row / transaction_guid | is_test | boolean | Test flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_test unchanged. |
| AN_097 | [PUB_004](#pub-004) | analytics.transactions_data_source | 1 row / transaction_guid | devicetype | varchar | Device type | Mobile | Backend |  | `[passthrough]` Reads public.fact_transaction; copies devicetype unchanged. |
| AN_098 | [PUB_004](#pub-004) | analytics.transactions_data_source | 1 row / transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_fanclub unchanged. |
| AN_105 | [PUB_004](#pub-004) | analytics.transactions_data_source | 1 row / transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_buynow unchanged. |
| AN_106 | [PUB_004](#pub-004) | analytics.transactions_data_source | 1 row / transaction_guid | charity_name | varchar(256) | Charity name | Save The Arts | Backend |  | `[passthrough]` Reads public.fact_transaction; copies charity_name unchanged. |
| AN_107 | [PUB_004](#pub-004) | analytics.transactions_data_source | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner payload | campaign=fall | Third-party |  | `[passthrough]` Reads public.fact_transaction; copies partner_defined_data_value unchanged. |
| AN_111 | [PUB_004](#pub-004) | analytics.transactions_data_source | 1 row / transaction_guid | network_token_usage | boolean | Network token usage | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies network_token_usage unchanged. |
| AN_113 | [PUB_005](#pub-005) | analytics.transactions_data_source | 1 row / transaction_guid | is_plf | boolean | PLF flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_plf unchanged. |
| AN_114 | [PUB_005](#pub-005) | analytics.transactions_data_source | 1 row / transaction_guid | is_imax | boolean | IMAX flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_imax unchanged. |
| AN_115 | [PUB_005](#pub-005) | analytics.transactions_data_source | 1 row / transaction_guid | is_3d | boolean | 3D flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_3d unchanged. |
| AN_116 | [PUB_005](#pub-005) | analytics.transactions_data_source | 1 row / transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | false | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_higher_fee_amenity unchanged. |
| AN_117 | [PUB_006](#pub-006) | analytics.transactions_data_source | 1 row / transaction_guid | utm_source | varchar(255) | UTM source | google | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_source unchanged. |
| AN_118 | [PUB_006](#pub-006) | analytics.transactions_data_source | 1 row / transaction_guid | utm_medium | varchar(255) | UTM medium | cpc | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_medium unchanged. |
| AN_119 | [PUB_006](#pub-006) | analytics.transactions_data_source | 1 row / transaction_guid | utm_campaign | varchar(255) | UTM campaign | fall_promo | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_campaign unchanged. |
| AN_120 | [PUB_006](#pub-006) | analytics.transactions_data_source | 1 row / transaction_guid | utm_content | varchar(255) | UTM content | banner_a | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_content unchanged. |
| AN_121 | [PUB_006](#pub-006) | analytics.transactions_data_source | 1 row / transaction_guid | utm_term | varchar(255) | UTM term | movie+tickets | Backend |  | `[passthrough]` Reads public.fact_transaction; copies utm_term unchanged. |
| AN_122 | [PUB_006](#pub-006) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | PR ticket tax rate | 11.5000 | Backend |  | `[passthrough]` Reads public.fact_transaction; copies ticket_tax_rate unchanged. |
| AN_123 | [PUB_006](#pub-006) | analytics.transactions_data_source | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | PR tax txn flag | true | Derived |  | `[passthrough]` Reads public.fact_transaction; copies is_ticket_tax_transaction unchanged. |

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
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="raw-022"></a>RAW_022 | raw_import_cache.transactionPayment + raw_import_cache.paypaltrrdetail + raw_import_cache.paypalstldetail + raw_import_cache.processorbraintreeauthorizationrequest + raw_import_cache.processorbraintreeauthorizationresponse + raw_import_cache.processorbraintreerefundrequest + raw_import_cache.processorbraintreerefundresponse | payment row / transaction | Payment_tp | mixed | Payment amounts, types, and processor order IDs | VISA / ord_123 | Backend/Third-party |  | **RAW_022**: `factPayment.sql` reads transactionPayment and processor response/request feeds; payment aggregation reads the resulting fact_payment. |
| RAW_022 | raw_import_cache.transactionPayment + raw_import_cache.paypaltrrdetail + raw_import_cache.paypalstldetail + raw_import_cache.processorbraintreeauthorizationrequest + raw_import_cache.processorbraintreeauthorizationresponse + raw_import_cache.processorbraintreerefundrequest + raw_import_cache.processorbraintreerefundresponse | payment row / transaction | Account_cd | mixed | Payment amounts, types, and processor order IDs | VISA / ord_123 | Backend/Third-party |  | **RAW_022**: `factPayment.sql` reads transactionPayment and processor response/request feeds; payment aggregation reads the resulting fact_payment. |
| RAW_022 | raw_import_cache.transactionPayment + raw_import_cache.paypaltrrdetail + raw_import_cache.paypalstldetail + raw_import_cache.processorbraintreeauthorizationrequest + raw_import_cache.processorbraintreeauthorizationresponse + raw_import_cache.processorbraintreerefundrequest + raw_import_cache.processorbraintreerefundresponse | payment row / transaction | Transaction_at | mixed | Payment amounts, types, and processor order IDs | VISA / ord_123 | Backend/Third-party |  | **RAW_022**: `factPayment.sql` reads transactionPayment and processor response/request feeds; payment aggregation reads the resulting fact_payment. |
| RAW_022 | raw_import_cache.transactionPayment + raw_import_cache.paypaltrrdetail + raw_import_cache.paypalstldetail + raw_import_cache.processorbraintreeauthorizationrequest + raw_import_cache.processorbraintreeauthorizationresponse + raw_import_cache.processorbraintreerefundrequest + raw_import_cache.processorbraintreerefundresponse | payment row / transaction | Payment_GUID | mixed | Payment amounts, types, and processor order IDs | VISA / ord_123 | Backend/Third-party |  | **RAW_022**: `factPayment.sql` reads transactionPayment and processor response/request feeds; payment aggregation reads the resulting fact_payment. |
| RAW_022 | raw_import_cache.transactionPayment + raw_import_cache.paypaltrrdetail + raw_import_cache.paypalstldetail + raw_import_cache.processorbraintreeauthorizationrequest + raw_import_cache.processorbraintreeauthorizationresponse + raw_import_cache.processorbraintreerefundrequest + raw_import_cache.processorbraintreerefundresponse | payment row / transaction | processor transaction IDs | mixed | Payment amounts, types, and processor order IDs | VISA / ord_123 | Backend/Third-party |  | **RAW_022**: `factPayment.sql` reads transactionPayment and processor response/request feeds; payment aggregation reads the resulting fact_payment. |
| <a id="raw-023"></a>RAW_023 | raw_import_cache.transactionItem | 1 row / item | Item_qy | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | Item_at | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | SalesFee_at | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | WaivedSalesFee_at | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | SurchargeFee_at | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | ReservedFee_at | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | CardFee_at | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | Shipping_at | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | taxable_at | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | totaladmissions_qy | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| RAW_023 | raw_import_cache.transactionItem | 1 row / item | Item_tp | numeric/mixed | Item-level quantities, ticket sales, fees, and taxable-fee inputs | 2 / 35.00 | Backend |  | **RAW_023**: `factTransactionItem.sql` uses these fields to build public.fact_transaction_item; the aggregate item_count excludes subscription items. |
| <a id="raw-024"></a><a id="raw-035"></a>RAW_024 | raw_import_cache.ticketTypeMapping + raw_import_cache.ticketCategory | mapping/category row | pos_description | mixed | Item description lookups used by aggregate | — | Backend |  | **RAW_024**: `factTransactionItemAggregate.sql` uses these mappings to determine item_type labels before the transaction-level rollup. |
| RAW_024 | raw_import_cache.ticketTypeMapping + raw_import_cache.ticketCategory | mapping/category row | chain_id | mixed | Item description lookups used by aggregate | — | Backend |  | **RAW_024**: `factTransactionItemAggregate.sql` uses these mappings to determine item_type labels before the transaction-level rollup. |
| RAW_024 | raw_import_cache.ticketTypeMapping + raw_import_cache.ticketCategory | mapping/category row | category_id | mixed | Item description lookups used by aggregate | — | Backend |  | **RAW_024**: `factTransactionItemAggregate.sql` uses these mappings to determine item_type labels before the transaction-level rollup. |
| RAW_024 | raw_import_cache.ticketTypeMapping + raw_import_cache.ticketCategory | mapping/category row | category_description | mixed | Item description lookups used by aggregate | — | Backend |  | **RAW_024**: `factTransactionItemAggregate.sql` uses these mappings to determine item_type labels before the transaction-level rollup. |
| RAW_024 | raw_import_cache.ticketTypeMapping + raw_import_cache.ticketCategory | mapping/category row | display_description | mixed | Item description lookups used by aggregate | — | Backend |  | **RAW_024**: `factTransactionItemAggregate.sql` uses these mappings to determine item_type labels before the transaction-level rollup. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-024"></a>STG_024 | [RAW_022](#raw-022) | warehouse_loader.fact_payment_aggregate | 1 row / transaction_guid | payment_types | mixed | Payment aggregates at tx grain | — | Derived/Third-party |  | `[aggregate]` Transaction-level rollup of raw payment lineage. SQL reads public.fact_payment and joins public.dim_payment_instrument and public.dim_payment_type for payment type labels. |
| STG_024 | [RAW_022](#raw-022) | warehouse_loader.fact_payment_aggregate | 1 row / transaction_guid | processor_order_ids | mixed | Payment aggregates at tx grain | — | Derived/Third-party |  | `[aggregate]` Transaction-level rollup of raw payment lineage. SQL reads public.fact_payment and joins public.dim_payment_instrument and public.dim_payment_type for payment type labels. |
| STG_024 | [RAW_022](#raw-022) | warehouse_loader.fact_payment_aggregate | 1 row / transaction_guid | gross_transaction_amount | mixed | Payment aggregates at tx grain | — | Derived/Third-party |  | `[aggregate]` Transaction-level rollup of raw payment lineage. SQL reads public.fact_payment and joins public.dim_payment_instrument and public.dim_payment_type for payment type labels. |
| <a id="stg-025"></a>STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_count | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_sales_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_fees_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_surcharge_fee_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_waived_sales_fee_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | fanclub_waived_sales_fee_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | fee_waiver_item_count | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | fanclub_fee_waiver_item_count | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_taxable_sales_fee_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |
| STG_025 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | charity_amount | numeric | Item aggregates at tx grain | — | Derived |  | `[aggregate]` Transaction-level rollup of raw item measures. SQL reads public.fact_transaction_item; donation amounts feed charity separately, and subscription items are excluded from item_count. Item-label mappings are used for an intermediate item_types value, which is not listed in this row. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-007"></a>PUB_007 | [STG_024](#stg-024) | public.fact_transaction | 1 row / transaction_guid | payment_types | mixed | Payment labels and processor IDs | — | Derived/Third-party |  | `[passthrough]` Copies payment labels and processor IDs from warehouse_loader.fact_payment_aggregate. |
| PUB_007 | [STG_024](#stg-024) | public.fact_transaction | 1 row / transaction_guid | processor_order_ids | mixed | Payment labels and processor IDs | — | Derived/Third-party |  | `[passthrough]` Copies payment labels and processor IDs from warehouse_loader.fact_payment_aggregate. |
| <a id="pub-010"></a>PUB_008 | [STG_024](#stg-024) | public.fact_transaction | 1 row / transaction_guid | true_gross_transaction_amount | mixed | True gross transaction amount | — | Derived |  | `[passthrough]` Copies true_gross_transaction_amount from warehouse_loader.fact_payment_aggregate.gross_transaction_amount. |
| <a id="pub-008"></a>PUB_009 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | item_count | mixed | Item quantity | — | Derived |  | `[rename/cast]` Copies item_count from warehouse_loader.fact_transaction_item_aggregate. |
| <a id="pub-010-item"></a>PUB_008 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | sales_fee_amount | mixed | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate. |
| PUB_008 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | surcharge_fee_amount | mixed | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate. |
| PUB_008 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | waived_sales_fee_amount | mixed | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate. |
| PUB_008 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | waived_fanclub_fee_amount | mixed | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate. |
| PUB_008 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | waived_sales_fee_ticket_count | mixed | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate. |
| PUB_008 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | waived_fanclub_fee_ticket_count | mixed | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate. |
| PUB_008 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | ticket_sales_amount | mixed | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate. |
| PUB_008 | [STG_025](#stg-025), [STG_027](#stg-027) | public.fact_transaction | 1 row / transaction_guid | taxable_sales_fee_amount | mixed | Taxable sales fee amount | — | Derived |  | `[conditional]` Copies the purchase value from warehouse_loader.fact_transaction_item_aggregate and updates refund rows from #fact_return_lookup, built from public.fact_return. |
| PUB_008 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | charity_amount | mixed | Aggregated commercial metrics | — | Derived |  | `[passthrough]` Copies sales, fee, waiver, ticket-sales, taxable-fee, and charity measures from warehouse_loader.fact_transaction_item_aggregate. |
| <a id="pub-009"></a>PUB_010 | [STG_024](#stg-024), [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross minus charity | 42.50 | Derived |  | `[derived]` Sets gross amount to warehouse_loader.fact_payment_aggregate.gross_transaction_amount minus warehouse_loader.fact_transaction_item_aggregate.charity_amount. |
| <a id="pub-011"></a><a id="pub-020"></a>PUB_011 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_006](#stg-006), [STG_028](#stg-028) | public.fact_payment | payment grain | transaction_guid | mixed | Payment facts and type lookup used by payment aggregate | — | ETL |  | `[lookup]` Builds public.fact_payment by joining warehouse_loader.fact_transaction_base, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.paypaltrrdetail, raw_import_cache.paypalstldetail, raw_import_cache.processorbraintreeauthorizationrequest, raw_import_cache.processorbraintreeauthorizationresponse, raw_import_cache.processorbraintreerefundrequest, and raw_import_cache.processorbraintreerefundresponse; it also uses public.dim_payment_instrument, public.dim_payment_type, public.dim_processor, public.dim_processor_error_code, and public.dim_credit_card_type. The aggregate reads public.fact_payment and the payment instrument/type dimensions. |
| PUB_011 | [RAW_022](#raw-022) | public.fact_payment | payment grain | transaction_amount | mixed | Payment facts and type lookup used by payment aggregate | — | ETL |  | `[lookup]` Builds public.fact_payment by joining warehouse_loader.fact_transaction_base, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.paypaltrrdetail, raw_import_cache.paypalstldetail, raw_import_cache.processorbraintreeauthorizationrequest, raw_import_cache.processorbraintreeauthorizationresponse, raw_import_cache.processorbraintreerefundrequest, and raw_import_cache.processorbraintreerefundresponse; it also uses public.dim_payment_instrument, public.dim_payment_type, public.dim_processor, public.dim_processor_error_code, and public.dim_credit_card_type. The aggregate reads public.fact_payment and the payment instrument/type dimensions. |
| PUB_011 | [RAW_022](#raw-022) | public.fact_payment | payment grain | payment_instrument_key | mixed | Payment facts and type lookup used by payment aggregate | — | ETL |  | `[lookup]` Builds public.fact_payment by joining warehouse_loader.fact_transaction_base, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.paypaltrrdetail, raw_import_cache.paypalstldetail, raw_import_cache.processorbraintreeauthorizationrequest, raw_import_cache.processorbraintreeauthorizationresponse, raw_import_cache.processorbraintreerefundrequest, and raw_import_cache.processorbraintreerefundresponse; it also uses public.dim_payment_instrument, public.dim_payment_type, public.dim_processor, public.dim_processor_error_code, and public.dim_credit_card_type. The aggregate reads public.fact_payment and the payment instrument/type dimensions. |
| PUB_011 | [RAW_022](#raw-022) | public.fact_payment | payment grain | processor_order_id | mixed | Payment facts and type lookup used by payment aggregate | — | ETL |  | `[lookup]` Builds public.fact_payment by joining warehouse_loader.fact_transaction_base, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.paypaltrrdetail, raw_import_cache.paypalstldetail, raw_import_cache.processorbraintreeauthorizationrequest, raw_import_cache.processorbraintreeauthorizationresponse, raw_import_cache.processorbraintreerefundrequest, and raw_import_cache.processorbraintreerefundresponse; it also uses public.dim_payment_instrument, public.dim_payment_type, public.dim_processor, public.dim_processor_error_code, and public.dim_credit_card_type. The aggregate reads public.fact_payment and the payment instrument/type dimensions. |
| PUB_011 | [RAW_022](#raw-022) | public.fact_payment | payment grain | payment_type_id | mixed | Payment facts and type lookup used by payment aggregate | — | ETL |  | `[lookup]` Builds public.fact_payment by joining warehouse_loader.fact_transaction_base, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.paypaltrrdetail, raw_import_cache.paypalstldetail, raw_import_cache.processorbraintreeauthorizationrequest, raw_import_cache.processorbraintreeauthorizationresponse, raw_import_cache.processorbraintreerefundrequest, and raw_import_cache.processorbraintreerefundresponse; it also uses public.dim_payment_instrument, public.dim_payment_type, public.dim_processor, public.dim_processor_error_code, and public.dim_credit_card_type. The aggregate reads public.fact_payment and the payment instrument/type dimensions. |
| <a id="pub-012"></a><a id="pub-021"></a>PUB_012 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_006](#stg-006), [STG_028](#stg-028) | public.fact_transaction_item | item grain | transaction_guid | mixed | Item facts used by item aggregate | — | ETL |  | `[lookup]` Joins raw_import_cache.transactionItem to warehouse_loader.fact_transaction_base, public.dim_purchase_type, and public.dim_theater to build public.fact_transaction_item. Its aggregate also reads public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory. |
| PUB_012 | [RAW_023](#raw-023) | public.fact_transaction_item | item grain | item_count | mixed | Item facts used by item aggregate | — | ETL |  | `[lookup]` Joins raw_import_cache.transactionItem to warehouse_loader.fact_transaction_base, public.dim_purchase_type, and public.dim_theater to build public.fact_transaction_item. Its aggregate also reads public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory. |
| PUB_012 | [RAW_023](#raw-023) | public.fact_transaction_item | item grain | item_amount | mixed | Item facts used by item aggregate | — | ETL |  | `[lookup]` Joins raw_import_cache.transactionItem to warehouse_loader.fact_transaction_base, public.dim_purchase_type, and public.dim_theater to build public.fact_transaction_item. Its aggregate also reads public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory. |
| PUB_012 | [RAW_023](#raw-023) | public.fact_transaction_item | item grain | sales_fee_amount | mixed | Item facts used by item aggregate | — | ETL |  | `[lookup]` Joins raw_import_cache.transactionItem to warehouse_loader.fact_transaction_base, public.dim_purchase_type, and public.dim_theater to build public.fact_transaction_item. Its aggregate also reads public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory. |
| PUB_012 | [RAW_023](#raw-023) | public.fact_transaction_item | item grain | surcharge_fee_amount | mixed | Item facts used by item aggregate | — | ETL |  | `[lookup]` Joins raw_import_cache.transactionItem to warehouse_loader.fact_transaction_base, public.dim_purchase_type, and public.dim_theater to build public.fact_transaction_item. Its aggregate also reads public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory. |
| PUB_012 | [RAW_023](#raw-023) | public.fact_transaction_item | item grain | waived_sales_fee_amount | mixed | Item facts used by item aggregate | — | ETL |  | `[lookup]` Joins raw_import_cache.transactionItem to warehouse_loader.fact_transaction_base, public.dim_purchase_type, and public.dim_theater to build public.fact_transaction_item. Its aggregate also reads public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory. |
| PUB_012 | [RAW_023](#raw-023) | public.fact_transaction_item | item grain | taxable_sales_fee_amount | mixed | Item facts used by item aggregate | — | ETL |  | `[lookup]` Joins raw_import_cache.transactionItem to warehouse_loader.fact_transaction_base, public.dim_purchase_type, and public.dim_theater to build public.fact_transaction_item. Its aggregate also reads public.dim_chain, raw_import_cache.ticketTypeMapping, and raw_import_cache.ticketCategory. |

</details>

<details open>
<summary><strong>Analytics (13 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_020 | [PUB_007](#pub-007) | analytics.transactions_data_source | 1 row / transaction_guid | payment_types | varchar(200) | Payment types | VISA | Derived |  | `[passthrough]` Reads public.fact_transaction; copies payment_types from public.fact_transaction, assembled by warehouse_loader.fact_payment_aggregate. |
| AN_021 | [PUB_007](#pub-007) | analytics.transactions_data_source | 1 row / transaction_guid | processor_order_ids | varchar(124) | Processor order ids | ord_123 | Third-party |  | `[passthrough]` Reads public.fact_transaction; copies processor_order_ids from public.fact_transaction, assembled by warehouse_loader.fact_payment_aggregate. |
| AN_073 | [PUB_009](#pub-009) | analytics.transactions_data_source | 1 row / transaction_guid | item_qty | int | Item qty | 2 | Derived |  | `[rename/cast]` Reads public.fact_transaction; renames item_count to item_qty; public.fact_transaction gets the count from warehouse_loader.fact_transaction_item_aggregate. |
| AN_074 | [PUB_010](#pub-010) | analytics.transactions_data_source | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies gross_transaction_amount from public.fact_transaction; factTransaction.sql derives it from warehouse_loader.fact_payment_aggregate less aggregate charity. |
| AN_075 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee | 5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies sales_fee_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_076 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | waived_sales_fee_ticket_count | numeric(19,4) | Waived sales fee ticket count | 0 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies waived sales-fee ticket count from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_079 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies ticket_sales_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_080 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies waived_sales_fee_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_081 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee | 1.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies surcharge_fee_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_099 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies waived_fanclub_fee_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_100 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | waived_fanclub_fee_ticket_count | numeric(19,4) | Waived fanclub fee ticket count | 0 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies waived_fanclub_fee_ticket_count from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_101 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies charity_amount from public.fact_transaction, built from warehouse_loader.fact_transaction_item_aggregate. |
| AN_102 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | true_gross_transaction_amount | numeric(19,4) | Gross including charity | 43.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies true_gross_transaction_amount from public.fact_transaction, built from warehouse_loader.fact_payment_aggregate. |

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
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| RAW_025 | raw_import_cache.refund | 1 row / refund line | Price_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| RAW_025 | raw_import_cache.refund | 1 row / refund line | Fee_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| RAW_025 | raw_import_cache.refund | 1 row / refund line | Tax_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| RAW_025 | raw_import_cache.refund | 1 row / refund line | Surcharge_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| RAW_025 | raw_import_cache.refund | 1 row / refund line | ReservedFee_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| RAW_025 | raw_import_cache.refund | 1 row / refund line | CardFee_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| RAW_025 | raw_import_cache.refund | 1 row / refund line | taxable_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| RAW_025 | raw_import_cache.refund | 1 row / refund line | donation_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-026"></a>STG_026 | [RAW_025](#raw-025) | warehouse_loader.fact_return_aggregate | 1 row / purchase_transaction_guid | returned_ticket_amount | numeric | Purchase-side returned rollups | — | Derived |  | `[aggregate]` Transaction-level rollup of raw refund lineage. The ETL sums successful public.fact_return rows by purchase_transaction_guid. |
| STG_026 | [RAW_025](#raw-025) | warehouse_loader.fact_return_aggregate | 1 row / purchase_transaction_guid | returned_fee_amount | numeric | Purchase-side returned rollups | — | Derived |  | `[aggregate]` Transaction-level rollup of raw refund lineage. The ETL sums successful public.fact_return rows by purchase_transaction_guid. |
| STG_026 | [RAW_025](#raw-025) | warehouse_loader.fact_return_aggregate | 1 row / purchase_transaction_guid | returned_charity_amount | numeric | Purchase-side returned rollups | — | Derived |  | `[aggregate]` Transaction-level rollup of raw refund lineage. The ETL sums successful public.fact_return rows by purchase_transaction_guid. |
| <a id="stg-027"></a>STG_027 | [RAW_025](#raw-025) | #fact_return_lookup (temp) | 1 row / return_transaction_guid | return_* | mixed | Return lookup set in run window | — | ETL |  | `[lookup]` Stages raw refund lineage for refund-row updates by copying return details from public.fact_return into #fact_return_lookup. |
| STG_027 | [RAW_025](#raw-025) | #fact_return_lookup (temp) | 1 row / return_transaction_guid | purchase_transaction_guid | mixed | Return lookup set in run window | — | ETL |  | `[lookup]` Stages raw refund lineage for refund-row updates by copying return details from public.fact_return into #fact_return_lookup. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-013"></a>PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_transaction_guid | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | purchase_transaction_guid | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_fee_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_tax_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_surcharge_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_reservation_fee_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_card_fee_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| PUB_013 | [STG_001](#stg-001), [STG_002](#stg-002), [STG_003](#stg-003), [STG_004](#stg-004), [STG_006](#stg-006), [STG_007](#stg-007), [STG_008](#stg-008), [STG_024](#stg-024), [STG_025](#stg-025), [STG_028](#stg-028) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_charity_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-014"></a>PUB_014 | [STG_026](#stg-026) | public.fact_transaction | 1 row / purchase transaction_guid | returned_amount | numeric(19,4) | Returned values at purchase row | -42.50 | Derived |  | `[lookup]` Updates purchase rows from warehouse_loader.fact_return_aggregate, matched by purchase GUID. |
| PUB_014 | [STG_026](#stg-026) | public.fact_transaction | 1 row / purchase transaction_guid | returned_fee_amount | numeric(19,4) | Returned values at purchase row | -42.50 | Derived |  | `[lookup]` Updates purchase rows from warehouse_loader.fact_return_aggregate, matched by purchase GUID. |
| PUB_014 | [STG_026](#stg-026) | public.fact_transaction | 1 row / purchase transaction_guid | returned_charity_amount | numeric(19,4) | Returned values at purchase row | -42.50 | Derived |  | `[lookup]` Updates purchase rows from warehouse_loader.fact_return_aggregate, matched by purchase GUID. |
| <a id="pub-015"></a>PUB_015 | [STG_027](#stg-027) | public.fact_transaction | 1 row / refund transaction_guid | sales_fee_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| PUB_015 | [STG_027](#stg-027) | public.fact_transaction | 1 row / refund transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| PUB_015 | [STG_027](#stg-027) | public.fact_transaction | 1 row / refund transaction_guid | reservation_fee_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| PUB_015 | [STG_027](#stg-027) | public.fact_transaction | 1 row / refund transaction_guid | tax_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| PUB_015 | [STG_027](#stg-027) | public.fact_transaction | 1 row / refund transaction_guid | card_fee_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| PUB_015 | [STG_027](#stg-027) | public.fact_transaction | 1 row / refund transaction_guid | charity_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| <a id="pub-016"></a>PUB_016 | [STG_027](#stg-027), [STG_025](#stg-025) | public.fact_transaction | 1 row / refund transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Purchase fee amount tied to returned purchase | 5.00 | Derived |  | `[lookup]` Updates public.fact_transaction from warehouse_loader.fact_transaction_item_aggregate through #fact_return_lookup from public.fact_return. |

</details>

<details open>
<summary><strong>Analytics (11 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_077 | [PUB_014](#pub-014) | analytics.transactions_data_source | 1 row / transaction_guid | returned_amount | numeric(19,4) | Returned amount | -42.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies returned_amount from public.fact_transaction; factTransaction.sql updates that table from warehouse_loader.fact_return_aggregate, with delayed analytics sync from public.fact_transaction. |
| AN_078 | [PUB_014](#pub-014) | analytics.transactions_data_source | 1 row / transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | -5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies returned_fee_amount from public.fact_transaction; it is updated from warehouse_loader.fact_return_aggregate, with delayed analytics sync from public.fact_transaction. |
| AN_082 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_return_amount | numeric(19,4) | Sales fee return amount | -5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies sales_fee_return_amount from public.fact_transaction; factTransaction.sql negates the fee from public.fact_return via #fact_return_lookup. |
| AN_083 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Surcharge fee return amount | -1.50 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies surcharge_fee_return_amount from public.fact_transaction; factTransaction.sql negates the return surcharge from public.fact_return via #fact_return_lookup. |
| AN_084 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | reservation_fee_return_amount | numeric(19,4) | Reservation fee return amount | 0.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies reservation_fee_return_amount from public.fact_transaction; factTransaction.sql negates the return reservation fee from public.fact_return via #fact_return_lookup. |
| AN_085 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | tax_return_amount | numeric(19,4) | Tax return amount | -2.10 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies tax_return_amount from public.fact_transaction; factTransaction.sql negates the return tax from public.fact_return via #fact_return_lookup. |
| AN_086 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | card_fee_return_amount | numeric(19,4) | Card fee return amount | -0.40 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies card_fee_return_amount from public.fact_transaction; factTransaction.sql negates the return card fee from public.fact_return via #fact_return_lookup. |
| AN_087 | [PUB_016](#pub-016) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee amount for returned purchase | 5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies sales_fee_amount_for_returned_purchase from public.fact_transaction; factTransaction.sql gets it from warehouse_loader.fact_transaction_item_aggregate through #fact_return_lookup from public.fact_return. |
| AN_091 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies taxable_sales_fee_amount from public.fact_transaction; factTransaction.sql updates refund rows from public.fact_return via #fact_return_lookup. |
| AN_103 | [PUB_014](#pub-014) | analytics.transactions_data_source | 1 row / transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | -1.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies returned_charity_amount from public.fact_transaction; it is updated from warehouse_loader.fact_return_aggregate, with delayed analytics sync from public.fact_transaction. |
| AN_104 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | -1.00 | Derived |  | `[passthrough]` Reads public.fact_transaction; copies charity_return_amount from public.fact_transaction; factTransaction.sql negates the value from public.fact_return via #fact_return_lookup. |

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
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| RAW_026 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | names | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| RAW_026 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | codes | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| RAW_026 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | dates | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| RAW_026 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | locations | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| RAW_026 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | parent IDs | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| RAW_026 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | vendor | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| RAW_026 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | terminal attributes | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| RAW_026 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | and partner labels | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-027"></a><a id="raw-025"></a>RAW_027 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | Affiliate_id | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| RAW_027 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | Transaction_tp | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| RAW_027 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | UserAgent_nm | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| RAW_027 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | Property_nm | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| RAW_027 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | refund-to-purchase GUID | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| <a id="raw-028"></a><a id="raw-026"></a>RAW_028 | raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | movie_detail_id | int | Movie/theater ids | 10293/4451 | Backend |  | Inputs to movie/theater keys. |
| RAW_028 | raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | theater_pkey | int | Movie/theater ids | 10293/4451 | Backend |  | Inputs to movie/theater keys. |
| <a id="raw-029"></a>RAW_029 | raw_import_cache.transactionPurchaseFunnel | 1 row / tx | Funnel_cd | varchar | Funnel code | 14 | Backend |  | Input to purchase_funnel_key. |
| <a id="raw-030"></a>RAW_030 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (Channel) | varchar | Partner mapping attrs | TMO/WEBVIEW | Third-party |  | Inputs to partner dimension keys; refund rows can fall back to parent-purchase attributes. |
| RAW_030 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (PartnerPlatform) | varchar | Partner mapping attrs | TMO/WEBVIEW | Third-party |  | Inputs to partner dimension keys; refund rows can fall back to parent-purchase attributes. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-028"></a>STG_028 | [RAW_027](#raw-027), [RAW_028](#raw-028), [RAW_008](#raw-008), [RAW_029](#raw-029), [RAW_030](#raw-030) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | movie_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_027](#raw-027) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | affiliate_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_026](#raw-026) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | chain_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_027](#raw-027) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | channel_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_029](#raw-029) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | purchase_funnel_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_008](#raw-008) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | return_type_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_028](#raw-028) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | theater_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_027](#raw-027) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_type_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_027](#raw-027) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | user_agent_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_027](#raw-027) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | property_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_030](#raw-030) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | partner_channel_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| STG_028 | [RAW_030](#raw-030) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | partner_platform_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-017"></a>PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | movie_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies movie_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | affiliate_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies affiliate_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | chain_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies chain_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | channel_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies channel_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | purchase_funnel_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies purchase_funnel_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | return_type_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies return_type_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | theater_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies theater_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | transaction_type_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies transaction_type_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | user_agent_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies user_agent_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | property_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies property_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | partner_channel_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies partner_channel_key from warehouse_loader.fact_transaction_base. |
| PUB_017 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | partner_platform_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies partner_platform_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-018"></a>PUB_018 | [RAW_026](#raw-026) | public.dim_* tables | 1 row / dim key | descriptive attributes used by analytics | mixed | Lookup attributes | — | Backend/Third-party |  | `[lookup]` Analytics reads public.dim_affiliate, public.dim_movie, public.dim_channel, public.dim_chain, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_purchase_funnel, public.dim_return_type, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform. Their tier_1_dimensions jobs load the raw inputs grouped in RAW_026. |

</details>

<details open>
<summary><strong>Analytics (52 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_022 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_name | varchar(400) | Affiliate name | Fandango iOS App | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_affiliate; public.dim_affiliate.affiliate_name |
| AN_023 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_group_name | varchar(100) | Affiliate group name | Mobile | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_affiliate; public.dim_affiliate.affiliate_group_name |
| AN_024 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_category_name | varchar(480) | Affiliate category name | O&O | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_affiliate; public.dim_affiliate.affiliate_category_name |
| AN_025 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_id | int | Affiliate id | 11879 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_affiliate; public.dim_affiliate.affiliate_id |
| AN_026 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | channel | varchar(10) | Channel group | App | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_channel; public.dim_channel.channel_group |
| AN_027 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | channel_ungrouped | varchar(40) | Channel name | WAP | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_channel; public.dim_channel.channel_name |
| AN_028 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | filmid_child | int | Child movie id | 10293 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_id |
| AN_029 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | filmname_child | varchar(400) | Child movie title | Example Movie | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_title |
| AN_030 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | filmid_parent | varchar(32) | Parent movie id | 872 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_parent_id |
| AN_031 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | filmname_parent | varchar(400) | Parent movie title | Franchise X | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_parent_title |
| AN_032 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | releasedate | date | Release date | 2026-09-14 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_release_date |
| AN_033 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | movie_tms_id | int | Movie TMS id | 123456 | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_movie; public.dim_movie.movie_tms_id |
| AN_034 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | chainname | varchar(200) | Chain name | Example Cinemas | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_chain; public.dim_chain.chain_name |
| AN_035 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | chainid | varchar(10) | Chain id | CHN019 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_chain; public.dim_chain.chain_id |
| AN_036 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | pos_vendor | varchar(50) | POS vendor | Vista | Third-party |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_chain; public.dim_chain.point_of_sale_vendor |
| AN_037 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | awards_program_name | varchar(320) | Awards program | Rewards+ | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_chain; public.dim_chain.awards_program_name |
| AN_038 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | theater_id | int | Theater id | 4451 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.theater_id |
| AN_039 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | theater_code | varchar(10) | Theater code | TH4451 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.theater_code |
| AN_040 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | theater_name | varchar(200) | Theater name | Example Hollywood | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.theater_name |
| AN_041 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | pdi_id | varchar(6) | PDI id | AVSTA | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.pdi_id |
| AN_042 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | pos_theater_id | varchar(20) | POS theater id | POS-4451 | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.point_of_sale_theater_id |
| AN_043 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | latitude | real | Latitude | 34.10 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.latitude |
| AN_044 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | longitude | real | Longitude | -118.34 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.longitude |
| AN_045 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | point_of_sale_id | smallint | POS id | 1 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.point_of_sale_id |
| AN_046 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | address_1 | varchar(200) | Address 1 | 123 Main St | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.address_1 |
| AN_047 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | address_2 | varchar(200) | Address 2 | Suite 200 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.address_2 |
| AN_048 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | city_name | varchar(160) | City | Los Angeles | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.city_name |
| AN_049 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | state_code | varchar(2) | State code | CA | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.state_code |
| AN_050 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | zip_code | varchar(10) | ZIP code | 90028 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.zip_code |
| AN_051 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | country_code | varchar(3) | Country code | US | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.country_code |
| AN_052 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | tht_alternate_theater_phone_name | varchar(200) | Alternate phone/name | Box Office | Backend |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.alternate_theater_phone_name |
| AN_053 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | chain_specific_site_id | varchar(10) | Chain site id | SITE-22 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.chain_specific_site_id |
| AN_054 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | paymentech_terminal_id | varchar(24) | Paymentech terminal id | PT-100 | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.paymentech_terminal_id |
| AN_055 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | cybersource_terminal_id | varchar(24) | Cybersource terminal id | CS-200 | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.cybersource_terminal_id |
| AN_056 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | screen_count | smallint | Screen count | 12 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.screen_count |
| AN_057 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | seat_count | int | Seat count | 1800 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.seat_count |
| AN_058 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | theater_time_zone_name | varchar(50) | Theater timezone | America/Los_Angeles | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.theater_time_zone_name |
| AN_060 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_type_id | varchar(40) | Transaction type id | Purchase | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_transaction_type; public.dim_transaction_type.transaction_type_id |
| AN_061 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_type_name | varchar(40) | Transaction type name | Ticket Purchase | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_transaction_type; public.dim_transaction_type.transaction_type_name |
| AN_062 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | user_agent_id | varchar(480) | User agent id | UA-1 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_user_agent; public.dim_user_agent.user_agent_id |
| AN_063 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | user_agent_description | varchar(480) | User agent description | iOS Safari | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_user_agent; public.dim_user_agent.user_agent_description |
| AN_064 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | purchase_funnel_id | varchar(40) | Funnel id | 14 | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_purchase_funnel; public.dim_purchase_funnel.purchase_funnel_id |
| AN_065 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | purchase_funnel_name | varchar(100) | Funnel name | Guest Checkout | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_purchase_funnel; public.dim_purchase_funnel.purchase_funnel_name |
| AN_066 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | return_method | varchar(20) | Return method | Self-service | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_return_type; public.dim_return_type.return_method |
| AN_067 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | return_type_code | varchar(3) | Return type code | REF | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_return_type; public.dim_return_type.return_type_code |
| AN_068 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | return_type_description | varchar(20) | Return type description | Refund | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_return_type; public.dim_return_type.return_type_description |
| AN_088 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | property_id | varchar(60) | Property id | Fandango USA | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_property; public.dim_property.property_id |
| AN_090 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | braintree_merchant_id | varchar(60) | Merchant id | bt_merch_1 | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_theater; public.dim_theater.braintree_merchant_id |
| AN_108 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | partner_channel | varchar | Partner channel id | TMO | Third-party |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_partner_channel; public.dim_partner_channel.partner_channel_id |
| AN_109 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_partner_channel; public.dim_partner_channel.partner_display_name |
| AN_110 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | partner_platform | varchar | Partner platform id | WEBVIEW | Third-party |  | `[rename/cast][lookup]` Reads public.fact_transaction, public.dim_partner_platform; public.dim_partner_platform.partner_platform_id |
| AN_112 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | property_group | varchar | Property group | Fandango | Backend |  | `[lookup]` Reads public.fact_transaction, public.dim_property; public.dim_property.property_group |


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
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-019"></a>PUB_019 | [RAW_026](#raw-026) | public.dim_movie | dim grain | movie_release_date | mixed | Inputs to DATEDIFF and CASE logic | — | Backend |  | `[lookup]` Reads attributes from public.dim_movie, public.dim_affiliate, and public.dim_channel for analytics calculations. |
| PUB_019 | [RAW_026](#raw-026) | public.dim_affiliate | dim grain | affiliate_id | mixed | Inputs to DATEDIFF and CASE logic | — | Backend |  | `[lookup]` Reads attributes from public.dim_movie, public.dim_affiliate, and public.dim_channel for analytics calculations. |
| PUB_019 | [RAW_026](#raw-026) | public.dim_channel | dim grain | channel_name | mixed | Inputs to DATEDIFF and CASE logic | — | Backend |  | `[lookup]` Reads attributes from public.dim_movie, public.dim_affiliate, and public.dim_channel for analytics calculations. |

</details>

<details open>
<summary><strong>Analytics (11 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_004 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_start_datetime_los_angeles | timestamp | Pacific start datetime | 2026-10-01 09:10:00 | Derived |  | `[derived]` Reads public.fact_transaction; CONVERT_TIMEZONE('US/Pacific', start_utc) |
| AN_006 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_commit_datetime_los_angeles | timestamp | Pacific commit datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` Reads public.fact_transaction; timezone conversion |
| AN_008 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_rollback_datetime_los_angeles | timestamp | Pacific rollback datetime | NULL | Derived |  | `[derived]` Reads public.fact_transaction; timezone conversion |
| AN_010 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_los_angeles | timestamp | Pacific end datetime | 2026-10-01 09:11:05 | Derived |  | `[derived]` Reads public.fact_transaction; timezone conversion |
| AN_012 | [PUB_001](#pub-001), [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | deltareleasetopurchasedate | int | Days release->purchase day | 17 | Derived |  | `[derived]` Reads public.fact_transaction; calculates days from public.dim_movie.movie_release_date to the Pacific transaction date. |
| AN_013 | [PUB_003](#pub-003), [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | deltareleasetoshowtimedate | int | Days release->showtime day | 21 | Derived |  | `[derived]` Reads public.fact_transaction; calculates days from public.dim_movie.movie_release_date to public.fact_transaction.performance_datetime_local. |
| AN_014 | [PUB_001](#pub-001), [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | deltapurchasetoshowtimedate | int | Days purchase day->showtime day | 4 | Derived |  | `[derived]` Reads public.fact_transaction; calculates days between the Pacific transaction date and performance date in public.fact_transaction. |
| AN_069 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | "Affiliate Grouping" | varchar(256) | Detailed affiliate bucket | iPhone App | Derived |  | `[conditional]` Reads public.fact_transaction, public.dim_affiliate; uses public.dim_affiliate.affiliate_id in a CASE expression to assign the detailed affiliate bucket, including NULL handling. |
| AN_070 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | "Affiliate Grouping - HIGH LEVEL BINS" | varchar(256) | High-level bucket | Fandango O&O | Derived |  | `[conditional]` Reads public.fact_transaction, public.dim_affiliate; uses public.dim_affiliate.affiliate_id in a CASE expression to assign the high-level affiliate bucket. |
| AN_071 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | apps | varchar(256) | App bucket | Fandango App: iPhone | Derived |  | `[conditional]` Reads public.fact_transaction, public.dim_affiliate; uses public.dim_affiliate.affiliate_id in a CASE expression to assign the app bucket. |
| AN_072 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | platform | varchar(256) | Platform bucket | Mobile Apps | Derived |  | `[conditional]` Reads public.fact_transaction, public.dim_affiliate, public.dim_channel; uses public.dim_affiliate.affiliate_id and public.dim_channel.channel_name in a CASE expression; checks Platform first. |

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
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="raw-031"></a>RAW_031 | raw_import_cache.designatedMarketArea | 1 row / zip_code | zip_code | varchar | ZIP to DMA mapping | 90028 / Los Angeles | Third-party |  | External DMA source used in analytics join. |
| RAW_031 | raw_import_cache.designatedMarketArea | 1 row / zip_code | market_name | varchar | ZIP to DMA mapping | 90028 / Los Angeles | Third-party |  | External DMA source used in analytics join. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| PUB_020 | [STG_028](#stg-028) | public.dim_theater | 1 row / theater_key | zip_code | varchar(10) | Theater ZIP code | 90028 | Backend |  | `[lookup]` Reads ZIP from public.dim_theater; analytics joins it to raw_import_cache.designatedMarketArea by ZIP. |

</details>

<details open>
<summary><strong>Analytics (1 field)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_059 | [PUB_020](#pub-020) | analytics.transactions_data_source | 1 row / transaction_guid | dma_market_name | varchar(255) | DMA market name | Los Angeles | Third-party |  | `[lookup]` Reads public.fact_transaction, public.dim_theater, raw_import_cache.designatedMarketArea; joins public.dim_theater.zip_code to raw_import_cache.designatedMarketArea.zip_code and selects market_name. |

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
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="raw-032"></a>RAW_032 | data_lake.salestaxtransaction | line-item grain | document_number | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |
| RAW_032 | data_lake.salestaxtransaction | line-item grain | lineitem_number | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |
| RAW_032 | data_lake.salestaxtransaction | line-item grain | total_lineitem_tax | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |
| RAW_032 | data_lake.salestaxtransaction | line-item grain | tax area | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |
| RAW_032 | data_lake.salestaxtransaction | line-item grain | rate and status fields | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-029"></a>STG_029 | [RAW_032](#raw-032) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | document_number | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| STG_029 | [RAW_032](#raw-032) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | lineitem_number | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| STG_029 | [RAW_032](#raw-032) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | total_lineitem_tax | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| STG_029 | [RAW_032](#raw-032) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | tax area | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| STG_029 | [RAW_032](#raw-032) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | rate and status fields | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| <a id="stg-030"></a>STG_030 | [STG_029](#stg-029) | temp #transactionSalesTax | 1 row / transaction_guid | sales_fee_tax_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| STG_030 | [STG_029](#stg-029) | temp #transactionSalesTax | 1 row / transaction_guid | returned_sales_fee_tax_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| STG_030 | [STG_029](#stg-029) | temp #transactionSalesTax | 1 row / transaction_guid | sales_fee_tax_return_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| STG_030 | [STG_029](#stg-029) | temp #transactionSalesTax | 1 row / transaction_guid | sales_fee_tax_amount_for_returned_purchase | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| STG_030 | [STG_029](#stg-029) | temp #transactionSalesTax | 1 row / transaction_guid | pr_ticket_tax_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| STG_030 | [STG_029](#stg-029) | temp #transactionSalesTax | 1 row / transaction_guid | returned_ticket_tax_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| STG_030 | [STG_029](#stg-029) | temp #transactionSalesTax | 1 row / transaction_guid | ticket_tax_return_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| STG_030 | [STG_029](#stg-029) | temp #transactionSalesTax | 1 row / transaction_guid | ticket_tax_amount_for_returned_purchase | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| PUB_021 | [STG_029](#stg-029) | public.fact_sales_fee_tax | 1 row / transaction_guid | sales_fee_tax_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| PUB_021 | [STG_029](#stg-029) | public.fact_sales_fee_tax | 1 row / transaction_guid | returned_sales_fee_tax_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| PUB_021 | [STG_029](#stg-029) | public.fact_sales_fee_tax | 1 row / transaction_guid | sales_fee_tax_return_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| PUB_021 | [STG_029](#stg-029) | public.fact_sales_fee_tax | 1 row / transaction_guid | sales_fee_tax_amount_for_returned_purchase | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| PUB_021 | [STG_029](#stg-029) | public.fact_sales_fee_tax | 1 row / transaction_guid | pr_ticket_tax_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| PUB_021 | [STG_029](#stg-029) | public.fact_sales_fee_tax | 1 row / transaction_guid | returned_ticket_tax_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| PUB_021 | [STG_029](#stg-029) | public.fact_sales_fee_tax | 1 row / transaction_guid | ticket_tax_return_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| PUB_021 | [STG_029](#stg-029) | public.fact_sales_fee_tax | 1 row / transaction_guid | ticket_tax_amount_for_returned_purchase | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |

</details>

<details open>
<summary><strong>Analytics (9 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_092 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_amount | decimal(19,4) | Sales fee tax amount | 0.35 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies sales_fee_tax_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_093 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | returned_sales_fee_tax_amount | decimal(19,4) | Returned sales fee tax amount | -0.35 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies returned_sales_fee_tax_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_094 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_return_amount | decimal(19,4) | Sales fee tax return amount | -0.35 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies sales_fee_tax_return_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_095 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_amount_for_returned_purchase | decimal(19,4) | Sales fee tax amount for returned purchase | 0.35 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies sales_fee_tax_amount_for_returned_purchase from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_124 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_amount | decimal(19,4) | Vertex ticket tax amount | 3.50 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction, public.fact_sales_fee_tax; renames public.fact_sales_fee_tax.pr_ticket_tax_amount to ticket_tax_amount. |
| AN_125 | [PUB_021](#pub-021), [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | pre_tax_ticket_sales_amount | decimal(19,4) | Ticket sales amount before tax | 31.50 | Derived |  | `[derived]` Reads public.fact_transaction, public.fact_sales_fee_tax; calculates public.fact_transaction.ticket_sales_amount minus public.fact_sales_fee_tax.pr_ticket_tax_amount, using zero when tax is missing. |
| AN_126 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | returned_ticket_tax_amount | decimal(19,4) | Returned ticket tax amount | -3.50 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies returned_ticket_tax_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_127 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_return_amount | decimal(19,4) | Ticket tax return amount | -3.50 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies ticket_tax_return_amount from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |
| AN_128 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_amount_for_returned_purchase | decimal(19,4) | Ticket tax amount for returned purchase | 3.50 | Third-party/Derived |  | `[lookup]` Reads public.fact_transaction and public.fact_sales_fee_tax; copies ticket_tax_amount_for_returned_purchase from public.fact_sales_fee_tax. The salestax refresh also updates recent analytics rows from #transactionSalesTax. |


</details>
</details>

---

## Canonical Field ID Map (AN_001–AN_128)

| Field ID | Column Name |
| --- | --- |
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


