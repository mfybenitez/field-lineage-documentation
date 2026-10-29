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
- **Example values** are illustrative field formats, not samples from production data.
- **Data types** use declared types where bundled DDL/copy definitions are available; otherwise they show the type family inferred from the source expression, and exact target precision is unconfirmed.
- **Consistent field IDs**: `AN_001` … `AN_128` identify analytics fields; `RAW_###`, `STG_###`, and `PUB_###` identify Raw, Staging, and Public lineage fields. Every data row has one unique Field ID, numbered in document order within its layer.
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
| <a id="raw-001"></a>RAW_001 |  | raw_import_cache.transaction | 1 row / transaction_guid | Transaction_GUID | varchar(36) | Transaction identifier | 9F2A... | Backend |  | [passthrough] Reads `Transaction_GUID` from `raw_import_cache.transaction`. |
| <a id="raw-002"></a>RAW_002 |  | raw_import_cache.transaction | 1 row / transaction_guid | Create_dm | timestamp | Transaction start time | 2026-10-01 16:10:00 | Backend |  | [passthrough] Reads `Create_dm` from `raw_import_cache.transaction`. |
| <a id="raw-003"></a>RAW_003 |  | raw_import_cache.transaction | 1 row / transaction_guid | test_bt | boolean | Source test flag | false | Backend |  | [passthrough] Reads `test_bt` from `raw_import_cache.transaction`. |
| <a id="raw-004"></a>RAW_004 |  | raw_import_cache.transaction | 1 row / transaction_guid | DeviceType_cd | varchar | Device type code | Mobile | Backend |  | [passthrough] Reads `DeviceType_cd` from `raw_import_cache.transaction`. |
| <a id="raw-005"></a>RAW_005 |  | raw_import_cache.transaction | 1 row / transaction_guid | ticketTaxRate_rt | decimal(6,4) | Ticket tax rate | 0.0825 | Backend |  | [passthrough] Reads `ticketTaxRate_rt` from `raw_import_cache.transaction`. |
| <a id="raw-006"></a>RAW_006 |  | raw_import_cache.transaction | 1 row / transaction_guid | ticketTaxTransaction_bt | boolean | Ticket-tax transaction flag | true | Backend |  | [passthrough] Reads `ticketTaxTransaction_bt` from `raw_import_cache.transaction`. |
| <a id="raw-007"></a>RAW_007 |  | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Create_dm | timestamp | Commit time | 2026-10-01 16:11:05 | Backend |  | [passthrough] Reads `Create_dm` from `raw_import_cache.transactionCommitted`. |
| <a id="raw-008"></a>RAW_008 |  | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Confirmation_cd | varchar | POS confirmation code | CNF908812 | Backend |  | [passthrough] Reads `Confirmation_cd` from `raw_import_cache.transactionCommitted`. |
| <a id="raw-009"></a>RAW_009 |  | raw_import_cache.transactionCommitted | 1 row / transaction_guid | ReservedSeats_tx | varchar | Reserved-seat payload | A10,A11 | Backend |  | [passthrough] Reads `ReservedSeats_tx` from `raw_import_cache.transactionCommitted`. |
| <a id="raw-010"></a>RAW_010 |  | raw_import_cache.transactionCommitted | 1 row / transaction_guid | Loyalty_cd | varchar | Loyalty code | LOY123 | Backend |  | [passthrough] Reads `Loyalty_cd` from `raw_import_cache.transactionCommitted`. |
| <a id="raw-011"></a>RAW_011 |  | raw_import_cache.transactionCommitted | 1 row / transaction_guid | POS_reference_cd | varchar | POS reference code | POSREF1 | Backend |  | [passthrough] Reads `POS_reference_cd` from `raw_import_cache.transactionCommitted`. |
| <a id="raw-012"></a>RAW_012 |  | raw_import_cache.transactionRolledBack | 1 row / transaction_guid | Create_dm | timestamp | Rollback time | — | Backend |  | [passthrough] Reads `Create_dm` from `raw_import_cache.transactionRolledBack`. |
| <a id="raw-013"></a>RAW_013 |  | raw_import_cache.transactionPerformanceAttribute | 1 row / transaction and performance | Performance_dm | timestamp | Local performance time | 2026-10-05 19:30:00 | Backend |  | [passthrough] Reads `Performance_dm` from `raw_import_cache.transactionPerformanceAttribute`. |
| <a id="raw-014"></a>RAW_014 |  | raw_import_cache.transactionCustomer | 1 row / transaction and customer | Email_nm | varchar | Customer email used in test classification | — | Backend |  | [passthrough] Reads `Email_nm` from `raw_import_cache.transactionCustomer`. |
| <a id="raw-015"></a>RAW_015 |  | raw_import_cache.testEmailAddress | 1 row / email | email_address | varchar | Address checked against the test-email allowlist | tester@example.com | Backend |  | [passthrough] Reads `email_address` from `raw_import_cache.testEmailAddress`. |
| <a id="raw-016"></a>RAW_016 |  | raw_import_cache.testEmailAddress | 1 row / email | activation_datetime | timestamp | Allowlist activation time | 2026-01-01 00:00:00 | Backend |  | [passthrough] Reads `activation_datetime` from `raw_import_cache.testEmailAddress`. |
| <a id="raw-017"></a>RAW_017 |  | raw_import_cache.testEmailAddress | 1 row / email | expiration_datetime | timestamp | Allowlist expiration time | 2027-01-01 00:00:00 | Backend |  | [passthrough] Reads `expiration_datetime` from `raw_import_cache.testEmailAddress`. |
| <a id="raw-018"></a>RAW_018 |  | raw_import_cache.testTransactionEmailList | 1 row / email rule | email_tx | varchar | Test email address or pattern | loadtester_%@fandango.com | Backend |  | [passthrough] Reads `email_tx` from `raw_import_cache.testTransactionEmailList`. |
| <a id="raw-019"></a>RAW_019 |  | raw_import_cache.testTransactionEmailList | 1 row / email rule | active_bt | boolean | Whether the test-email rule is enabled | true | Backend |  | [passthrough] Reads `active_bt` from `raw_import_cache.testTransactionEmailList`. |
| <a id="raw-020"></a>RAW_020 |  | raw_import_cache.testTransactionEmailList | 1 row / email rule | pattern_bt | boolean | Whether the email rule is a pattern | true | Backend |  | [passthrough] Reads `pattern_bt` from `raw_import_cache.testTransactionEmailList`. |
| <a id="raw-021"></a>RAW_021 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (FanClubStatus) | varchar | Fanclub status value | Active | Backend |  | [passthrough] Reads `value_tx (FanClubStatus)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-022"></a>RAW_022 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (IsBuyNowOrder) | varchar | Buy-now order value | true | Backend |  | [passthrough] Reads `value_tx (IsBuyNowOrder)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-023"></a>RAW_023 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (CharityName) | varchar | Charity name attribute | Save The Arts | Backend |  | [passthrough] Reads `value_tx (CharityName)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-024"></a>RAW_024 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (PartnerDefinedData) | varchar | Partner-defined payload | campaign=fall | Third-party |  | [passthrough] Reads `value_tx (PartnerDefinedData)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-025"></a>RAW_025 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (networktoken) | varchar | Network-token usage value | true | Backend |  | [passthrough] Reads `value_tx (networktoken)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-026"></a>RAW_026 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (UtmSource) | varchar | UTM source value | google | Third-party |  | [passthrough] Reads `value_tx (UtmSource)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-027"></a>RAW_027 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (UtmMedium) | varchar | UTM medium value | cpc | Third-party |  | [passthrough] Reads `value_tx (UtmMedium)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-028"></a>RAW_028 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (UtmCampaign) | varchar | UTM campaign value | fall_promo | Third-party |  | [passthrough] Reads `value_tx (UtmCampaign)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-029"></a>RAW_029 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (UtmContent) | varchar | UTM content value | banner_a | Third-party |  | [passthrough] Reads `value_tx (UtmContent)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-030"></a>RAW_030 |  | raw_import_cache.transactionAttribute | 1 row / transaction and attribute | value_tx (UtmTerm) | varchar | UTM term value | movie+tickets | Third-party |  | [passthrough] Reads `value_tx (UtmTerm)` from `raw_import_cache.transactionAttribute`. |
| <a id="raw-031"></a>RAW_031 |  | raw_import_cache.transactionPerformanceAttribute | 1 row / transaction and performance | amenity_ids | varchar | Comma-delimited amenity IDs | 3,10 | Backend |  | [passthrough] Reads `amenity_ids` from `raw_import_cache.transactionPerformanceAttribute`. |
| <a id="raw-032"></a>RAW_032 |  | analytics_manual.plf_amenity_mapping | 1 row / amenity_id | amenity_id | varchar | Amenity ID lookup key | 10 | Internal mapping |  | [lookup] Reads `amenity_id` from `analytics_manual.plf_amenity_mapping`. |
| <a id="raw-033"></a>RAW_033 |  | analytics_manual.plf_amenity_mapping | 1 row / amenity_id | is_imax | varchar | IMAX amenity indicator | YES | Internal mapping |  | [lookup] Reads `is_imax` from `analytics_manual.plf_amenity_mapping`. |
| <a id="raw-034"></a>RAW_034 |  | analytics_manual.plf_amenity_mapping | 1 row / amenity_id | is_3d | varchar | 3D amenity indicator | NO | Internal mapping |  | [lookup] Reads `is_3d` from `analytics_manual.plf_amenity_mapping`. |
| <a id="raw-035"></a>RAW_035 |  | analytics_manual.plf_amenity_mapping | 1 row / amenity_id | is_higher_fee_amenity | varchar | Higher-fee amenity indicator | NO | Internal mapping |  | [lookup] Reads `is_higher_fee_amenity` from `analytics_manual.plf_amenity_mapping`. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-001"></a>STG_001 | [RAW_001](#raw-001) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction key | 9F2A... | Backend |  | [passthrough] Copies `Transaction_GUID` from `raw_import_cache.transaction`. |
| <a id="stg-002"></a>STG_002 | [RAW_007](#raw-007), [RAW_012](#raw-012) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_successful | boolean | Successful transaction flag | true | Derived |  | [conditional] True when `raw_import_cache.transactionCommitted` has a row and `raw_import_cache.transactionRolledBack` does not. |
| <a id="stg-003"></a>STG_003 | [RAW_002](#raw-002) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_start_datetime | timestamp | Transaction start time (UTC) | 2026-10-01 16:10:00 | Backend |  | [rename/cast] Casts `raw_import_cache.transaction.Create_dm` to timestamp. |
| <a id="stg-004"></a>STG_004 | [RAW_007](#raw-007) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_commit_datetime | timestamp | Transaction commit time (UTC) | 2026-10-01 16:11:05 | Backend |  | [passthrough] Copies `Create_dm` from `raw_import_cache.transactionCommitted`. |
| <a id="stg-005"></a>STG_005 | [RAW_012](#raw-012) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_rollback_datetime | timestamp | Transaction rollback time (UTC) | — | Backend |  | [passthrough] Copies `Create_dm` from `raw_import_cache.transactionRolledBack`. |
| <a id="stg-006"></a>STG_006 | [RAW_002](#raw-002), [RAW_007](#raw-007), [RAW_012](#raw-012) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_end_datetime | timestamp | Transaction end time (UTC) | 2026-10-01 16:11:05 | Derived |  | [conditional] Uses the later commit or rollback time; falls back to `raw_import_cache.transaction.Create_dm`. |
| <a id="stg-007"></a>STG_007 | [RAW_013](#raw-013) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | performance_datetime_local | timestamp | Local performance time | 2026-10-05 19:30:00 | Backend |  | [passthrough] Copies `Performance_dm` from `raw_import_cache.transactionPerformanceAttribute`. |
| <a id="stg-008"></a>STG_008 | [RAW_008](#raw-008) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation code | CNF908812 | Backend |  | [passthrough] Copies `Confirmation_cd` from `raw_import_cache.transactionCommitted`. |
| <a id="stg-009"></a>STG_009 | [RAW_009](#raw-009) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_reserved_seating | boolean | Reserved-seating flag | true | Derived |  | [conditional] True when `ReservedSeats_tx` in `raw_import_cache.transactionCommitted` is nonblank. |
| <a id="stg-010"></a>STG_010 | [RAW_009](#raw-009) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | reserved_seats | varchar(100) | Reserved seats | A10,A11 | Backend |  | [conditional] Trims `ReservedSeats_tx` from `raw_import_cache.transactionCommitted`; blank becomes NULL. |
| <a id="stg-011"></a>STG_011 | [RAW_010](#raw-010) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code | LOY123 | Backend |  | [conditional] Trims `Loyalty_cd` from `raw_import_cache.transactionCommitted`; blank becomes NULL. |
| <a id="stg-012"></a>STG_012 | [RAW_011](#raw-011) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | POS_reference_cd | varchar(40) | POS reference code | POSREF1 | Backend |  | [passthrough] Copies `POS_reference_cd` from `raw_import_cache.transactionCommitted`. |
| <a id="stg-013"></a>STG_013 | [RAW_003](#raw-003), [RAW_014](#raw-014), [RAW_015](#raw-015), [RAW_016](#raw-016), [RAW_017](#raw-017), [RAW_018](#raw-018), [RAW_019](#raw-019), [RAW_020](#raw-020), [RAW_078](#raw-078), [RAW_083](#raw-083) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_test | boolean | Test transaction flag | false | Derived |  | [conditional] Checks `raw_import_cache.transaction`, `raw_import_cache.transactionCustomer`, `raw_import_cache.testEmailAddress`, `raw_import_cache.testTransactionEmailList`, and `public.dim_theater`/`public.dim_transaction_type`. |
| <a id="stg-014"></a>STG_014 | [RAW_004](#raw-004) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | deviceType | varchar | Device type | Mobile | Backend |  | [passthrough] Copies `DeviceType_cd` from `raw_import_cache.transaction`. |
| <a id="stg-015"></a>STG_015 | [RAW_021](#raw-021) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_fanclub | boolean | Fanclub transaction flag | true | Derived |  | [conditional] Sets true for Active, Trial, OverdueTrial, or Overdue in `raw_import_cache.transactionAttribute`. |
| <a id="stg-016"></a>STG_016 | [RAW_022](#raw-022) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_buynow | boolean | Buy-now transaction flag | true | Derived |  | [conditional] Checks `IsBuyNowOrder` in `raw_import_cache.transactionAttribute`. |
| <a id="stg-017"></a>STG_017 | [RAW_023](#raw-023) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | charity_name | varchar(256) | Charity name | Save The Arts | Backend |  | [passthrough] Reads `CharityName` from `raw_import_cache.transactionAttribute`. |
| <a id="stg-018"></a>STG_018 | [RAW_024](#raw-024) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner-defined data | campaign=fall | Third-party |  | [passthrough] Reads `PartnerDefinedData` from `raw_import_cache.transactionAttribute`. |
| <a id="stg-019"></a>STG_019 | [RAW_025](#raw-025), [RAW_081](#raw-081) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | network_token_usage | boolean | Network-token usage flag | true | Derived |  | [conditional] Uses `NetworkToken` from `raw_import_cache.transactionAttribute`; refunds fall back through `raw_import_cache.refund` to the parent attribute. |
| <a id="stg-020"></a>STG_020 | [RAW_031](#raw-031), [RAW_032](#raw-032) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_plf | boolean | PLF amenity flag | false | Derived |  | [aggregate] Matches `raw_import_cache.transactionPerformanceAttribute.amenity_ids` to `analytics_manual.plf_amenity_mapping` and groups by transaction. |
| <a id="stg-021"></a>STG_021 | [RAW_031](#raw-031), [RAW_032](#raw-032), [RAW_033](#raw-033) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_imax | boolean | IMAX amenity flag | false | Derived |  | [aggregate] Matches `raw_import_cache.transactionPerformanceAttribute.amenity_ids` to `analytics_manual.plf_amenity_mapping` and groups by transaction. |
| <a id="stg-022"></a>STG_022 | [RAW_031](#raw-031), [RAW_032](#raw-032), [RAW_034](#raw-034) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_3d | boolean | 3D amenity flag | false | Derived |  | [aggregate] Matches `raw_import_cache.transactionPerformanceAttribute.amenity_ids` to `analytics_manual.plf_amenity_mapping` and groups by transaction. |
| <a id="stg-023"></a>STG_023 | [RAW_031](#raw-031), [RAW_032](#raw-032), [RAW_035](#raw-035) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | false | Derived |  | [aggregate] Matches `raw_import_cache.transactionPerformanceAttribute.amenity_ids` to `analytics_manual.plf_amenity_mapping` and groups by transaction. |
| <a id="stg-024"></a>STG_024 | [RAW_026](#raw-026) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_source | varchar(255) | UTM source | google | Third-party |  | [lookup] Reads `UtmSource` from `raw_import_cache.transactionAttribute`. |
| <a id="stg-025"></a>STG_025 | [RAW_027](#raw-027) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_medium | varchar(255) | UTM medium | cpc | Third-party |  | [lookup] Reads `UtmMedium` from `raw_import_cache.transactionAttribute`. |
| <a id="stg-026"></a>STG_026 | [RAW_028](#raw-028) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_campaign | varchar(255) | UTM campaign | fall_promo | Third-party |  | [lookup] Reads `UtmCampaign` from `raw_import_cache.transactionAttribute`. |
| <a id="stg-027"></a>STG_027 | [RAW_029](#raw-029) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_content | varchar(255) | UTM content | banner_a | Third-party |  | [lookup] Reads `UtmContent` from `raw_import_cache.transactionAttribute`. |
| <a id="stg-028"></a>STG_028 | [RAW_030](#raw-030) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | utm_term | varchar(255) | UTM term | movie+tickets | Third-party |  | [lookup] Reads `UtmTerm` from `raw_import_cache.transactionAttribute`. |
| <a id="stg-029"></a>STG_029 | [RAW_005](#raw-005) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | Ticket tax rate | 0.0825 | Backend |  | [passthrough] Copies `ticketTaxRate_rt` from `raw_import_cache.transaction`. |
| <a id="stg-030"></a>STG_030 | [RAW_006](#raw-006) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | Ticket-tax transaction flag | true | Derived |  | [conditional] Uses `ticketTaxTransaction_bt` from `raw_import_cache.transaction`; NULL defaults to false. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-001"></a>PUB_001 | [STG_001](#stg-001) | public.fact_transaction | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction identifier | 9F2A... | Backend |  | [passthrough] Copies `transaction_guid` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-002"></a>PUB_002 | [STG_002](#stg-002) | public.fact_transaction | 1 row / transaction_guid | is_successful | boolean | Whether the transaction completed successfully | true | Derived |  | [passthrough] Copies `is_successful` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-003"></a>PUB_003 | [STG_003](#stg-003) | public.fact_transaction | 1 row / transaction_guid | transaction_start_datetime | timestamp | Transaction start time in UTC | 2026-10-01 16:10:00 | Backend |  | [passthrough] Copies `transaction_start_datetime` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-004"></a>PUB_004 | [STG_004](#stg-004) | public.fact_transaction | 1 row / transaction_guid | transaction_commit_datetime | timestamp | Transaction commit time in UTC | 2026-10-01 16:11:05 | Backend |  | [passthrough] Copies `transaction_commit_datetime` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-005"></a>PUB_005 | [STG_005](#stg-005) | public.fact_transaction | 1 row / transaction_guid | transaction_rollback_datetime | timestamp | Transaction rollback time in UTC | NULL | Backend |  | [passthrough] Copies `transaction_rollback_datetime` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-006"></a>PUB_006 | [STG_006](#stg-006) | public.fact_transaction | 1 row / transaction_guid | transaction_end_datetime | timestamp | Transaction end time in UTC | 2026-10-01 16:11:05 | Derived |  | [passthrough] Copies `transaction_end_datetime` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-007"></a>PUB_007 | [STG_006](#stg-006), [STG_055](#stg-055) | public.fact_transaction | 1 row / transaction_guid | transaction_end_datetime_local | timestamp | Transaction end time in the theater time zone | 2026-10-01 09:11:05 | Derived |  | [derived] Converts `transaction_end_datetime` using `public.dim_theater.theater_time_zone_name`. |
| <a id="pub-008"></a>PUB_008 | [STG_007](#stg-007) | public.fact_transaction | 1 row / transaction_guid | performance_datetime_local | timestamp | Performance time associated with the transaction | 2026-10-05 19:30:00 | Backend |  | [passthrough] Copies `performance_datetime_local` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-009"></a>PUB_009 | [STG_008](#stg-008) | public.fact_transaction | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation code | CNF908812 | Backend |  | [passthrough] Copies `point_of_sale_confirmation_code` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-010"></a>PUB_010 | [STG_009](#stg-009) | public.fact_transaction | 1 row / transaction_guid | is_reserved_seating | boolean | Whether reserved seating was selected | true | Derived |  | [passthrough] Copies `is_reserved_seating` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-011"></a>PUB_011 | [STG_010](#stg-010) | public.fact_transaction | 1 row / transaction_guid | reserved_seats | varchar(100) | Reserved seat identifiers | A10,A11 | Backend |  | [passthrough] Copies `reserved_seats` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-012"></a>PUB_012 | [STG_011](#stg-011) | public.fact_transaction | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code on the transaction | LOY123 | Backend |  | [passthrough] Copies `loyalty_code` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-013"></a>PUB_013 | [STG_012](#stg-012) | public.fact_transaction | 1 row / transaction_guid | POS_reference_cd | varchar(40) | POS reference code | POSREF1 | Backend |  | [passthrough] Copies `POS_reference_cd` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-014"></a>PUB_014 | [STG_013](#stg-013) | public.fact_transaction | 1 row / transaction_guid | is_test | boolean | Whether the transaction is classified as a test | false | Derived |  | [passthrough] Copies `is_test` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-015"></a>PUB_015 | [STG_014](#stg-014) | public.fact_transaction | 1 row / transaction_guid | deviceType | varchar | Device type recorded for the transaction | Mobile | Backend |  | [passthrough] Copies `deviceType` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-016"></a>PUB_016 | [STG_015](#stg-015) | public.fact_transaction | 1 row / transaction_guid | is_fanclub | boolean | Whether the transaction is associated with fanclub | true | Derived |  | [passthrough] Copies `is_fanclub` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-017"></a>PUB_017 | [STG_016](#stg-016) | public.fact_transaction | 1 row / transaction_guid | is_buynow | boolean | Whether the transaction used Buy Now | true | Derived |  | [passthrough] Copies `is_buynow` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-018"></a>PUB_018 | [STG_017](#stg-017) | public.fact_transaction | 1 row / transaction_guid | charity_name | varchar(256) | Charity name associated with the transaction | Save The Arts | Backend |  | [passthrough] Copies `charity_name` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-019"></a>PUB_019 | [STG_018](#stg-018) | public.fact_transaction | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner-defined transaction data | campaign=fall | Third-party |  | [passthrough] Copies `partner_defined_data_value` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-020"></a>PUB_020 | [STG_019](#stg-019) | public.fact_transaction | 1 row / transaction_guid | network_token_usage | boolean | Whether network-token payment was used | true | Derived |  | [passthrough] Copies `network_token_usage` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-021"></a>PUB_021 | [STG_020](#stg-020) | public.fact_transaction | 1 row / transaction_guid | is_plf | boolean | Whether the performance has a PLF amenity | false | Derived |  | [passthrough] Copies `is_plf` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-022"></a>PUB_022 | [STG_021](#stg-021) | public.fact_transaction | 1 row / transaction_guid | is_imax | boolean | Whether the performance has an IMAX amenity | false | Derived |  | [passthrough] Copies `is_imax` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-023"></a>PUB_023 | [STG_022](#stg-022) | public.fact_transaction | 1 row / transaction_guid | is_3d | boolean | Whether the performance has a 3D amenity | false | Derived |  | [passthrough] Copies `is_3d` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-024"></a>PUB_024 | [STG_023](#stg-023) | public.fact_transaction | 1 row / transaction_guid | is_higher_fee_amenity | boolean | Whether the performance has a higher-fee amenity | false | Derived |  | [passthrough] Copies `is_higher_fee_amenity` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-025"></a>PUB_025 | [STG_024](#stg-024) | public.fact_transaction | 1 row / transaction_guid | utm_source | varchar(255) | UTM source associated with the transaction | google | Third-party |  | [passthrough] Copies `utm_source` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-026"></a>PUB_026 | [STG_025](#stg-025) | public.fact_transaction | 1 row / transaction_guid | utm_medium | varchar(255) | UTM medium associated with the transaction | cpc | Third-party |  | [passthrough] Copies `utm_medium` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-027"></a>PUB_027 | [STG_026](#stg-026) | public.fact_transaction | 1 row / transaction_guid | utm_campaign | varchar(255) | UTM campaign associated with the transaction | fall_promo | Third-party |  | [passthrough] Copies `utm_campaign` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-028"></a>PUB_028 | [STG_027](#stg-027) | public.fact_transaction | 1 row / transaction_guid | utm_content | varchar(255) | UTM content associated with the transaction | banner_a | Third-party |  | [passthrough] Copies `utm_content` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-029"></a>PUB_029 | [STG_028](#stg-028) | public.fact_transaction | 1 row / transaction_guid | utm_term | varchar(255) | UTM term associated with the transaction | movie+tickets | Third-party |  | [passthrough] Copies `utm_term` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-030"></a>PUB_030 | [STG_029](#stg-029) | public.fact_transaction | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | Ticket tax rate associated with the transaction | 0.0825 | Backend |  | [passthrough] Copies `ticket_tax_rate` from `warehouse_loader.fact_transaction_base`. |
| <a id="pub-031"></a>PUB_031 | [STG_030](#stg-030) | public.fact_transaction | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | Whether ticket tax applies to the transaction | true | Derived |  | [passthrough] Copies `is_ticket_tax_transaction` from `warehouse_loader.fact_transaction_base`. |

</details>

<details open>
<summary><strong>Analytics (31 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_001 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_guid | varchar(36) | Transaction id | 9F2A... | Backend |  | [passthrough] Reads `transaction_guid` from `public.fact_transaction`. |
| AN_002 | [PUB_002](#pub-002) | analytics.transactions_data_source | 1 row / transaction_guid | is_successful | boolean | Success flag | true | Derived |  | [passthrough] Reads `is_successful` from `public.fact_transaction`. |
| AN_003 | [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_start_datetime_utc | timestamp | Start UTC | 2026-10-01 16:10:00 | Backend |  | [rename/cast] Maps `transaction_start_datetime` to `transaction_start_datetime_utc` from `public.fact_transaction`. |
| AN_005 | [PUB_004](#pub-004) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_commit_datetime_utc | timestamp | Commit UTC | 2026-10-01 16:11:05 | Backend |  | [rename/cast] Maps `transaction_commit_datetime` to `transaction_commit_datetime_utc` from `public.fact_transaction`. |
| AN_007 | [PUB_005](#pub-005) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_rollback_datetime_utc | timestamp | Rollback UTC | NULL | Backend |  | [rename/cast] Maps `transaction_rollback_datetime` to `transaction_rollback_datetime_utc` from `public.fact_transaction`. |
| AN_009 | [PUB_006](#pub-006) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_utc | timestamp | End UTC | 2026-10-01 16:11:05 | Derived |  | [rename/cast] Maps `transaction_end_datetime` to `transaction_end_datetime_utc` from `public.fact_transaction`. |
| AN_011 | [PUB_007](#pub-007) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_local | timestamp | End local | 2026-10-01 09:11:05 | Derived |  | [passthrough] Reads `transaction_end_datetime_local` from `public.fact_transaction`. |
| AN_015 | [PUB_009](#pub-009) | analytics.transactions_data_source | 1 row / transaction_guid | point_of_sale_confirmation_code | varchar(40) | POS confirmation code | CNF908812 | Backend |  | [passthrough] Reads `point_of_sale_confirmation_code` from `public.fact_transaction`. |
| AN_016 | [PUB_010](#pub-010) | analytics.transactions_data_source | 1 row / transaction_guid | is_reserved_seating | boolean | Reserved seating flag | true | Derived |  | [passthrough] Reads `is_reserved_seating` from `public.fact_transaction`. |
| AN_017 | [PUB_011](#pub-011) | analytics.transactions_data_source | 1 row / transaction_guid | reserved_seats | varchar(100) | Reserved seats | A10,A11 | Backend |  | [passthrough] Reads `reserved_seats` from `public.fact_transaction`. |
| AN_018 | [PUB_012](#pub-012) | analytics.transactions_data_source | 1 row / transaction_guid | loyalty_code | varchar(40) | Loyalty code | LOY123 | Backend |  | [passthrough] Reads `loyalty_code` from `public.fact_transaction`. |
| AN_019 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | performance_datetime_local | timestamp | Performance local | 2026-10-05 19:30:00 | Backend |  | [passthrough] Reads `performance_datetime_local` from `public.fact_transaction`. |
| AN_089 | [PUB_013](#pub-013) | analytics.transactions_data_source | 1 row / transaction_guid | pos_reference_cd | varchar(40) | POS reference code | POSREF1 | Backend |  | [passthrough] Reads `POS_reference_cd` from `public.fact_transaction`. |
| AN_096 | [PUB_014](#pub-014) | analytics.transactions_data_source | 1 row / transaction_guid | is_test | boolean | Test flag | false | Derived |  | [passthrough] Reads `is_test` from `public.fact_transaction`. |
| AN_097 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | devicetype | varchar | Device type | Mobile | Backend |  | [passthrough] Reads `deviceType` from `public.fact_transaction`. |
| AN_098 | [PUB_016](#pub-016) | analytics.transactions_data_source | 1 row / transaction_guid | is_fanclub | boolean | Fanclub flag | true | Derived |  | [passthrough] Reads `is_fanclub` from `public.fact_transaction`. |
| AN_105 | [PUB_017](#pub-017) | analytics.transactions_data_source | 1 row / transaction_guid | is_buynow | boolean | Buy-now flag | true | Derived |  | [passthrough] Reads `is_buynow` from `public.fact_transaction`. |
| AN_106 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | charity_name | varchar(256) | Charity name | Save The Arts | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_107 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | partner_defined_data_value | varchar | Partner payload | campaign=fall | Third-party |  | [passthrough] Reads `partner_defined_data_value` from `public.fact_transaction`. |
| AN_111 | [PUB_020](#pub-020) | analytics.transactions_data_source | 1 row / transaction_guid | network_token_usage | boolean | Network token usage | true | Derived |  | [passthrough] Reads `network_token_usage` from `public.fact_transaction`. |
| AN_113 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | is_plf | boolean | PLF flag | false | Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_114 | [PUB_022](#pub-022) | analytics.transactions_data_source | 1 row / transaction_guid | is_imax | boolean | IMAX flag | false | Derived |  | [passthrough] Reads `is_imax` from `public.fact_transaction`. |
| AN_115 | [PUB_023](#pub-023) | analytics.transactions_data_source | 1 row / transaction_guid | is_3d | boolean | 3D flag | false | Derived |  | [passthrough] Reads `is_3d` from `public.fact_transaction`. |
| AN_116 | [PUB_024](#pub-024) | analytics.transactions_data_source | 1 row / transaction_guid | is_higher_fee_amenity | boolean | Higher-fee amenity flag | false | Derived |  | [passthrough] Reads `is_higher_fee_amenity` from `public.fact_transaction`. |
| AN_117 | [PUB_025](#pub-025) | analytics.transactions_data_source | 1 row / transaction_guid | utm_source | varchar(255) | UTM source | google | Backend |  | [passthrough] Reads `utm_source` from `public.fact_transaction`. |
| AN_118 | [PUB_026](#pub-026) | analytics.transactions_data_source | 1 row / transaction_guid | utm_medium | varchar(255) | UTM medium | cpc | Backend |  | [passthrough] Reads `utm_medium` from `public.fact_transaction`. |
| AN_119 | [PUB_027](#pub-027) | analytics.transactions_data_source | 1 row / transaction_guid | utm_campaign | varchar(255) | UTM campaign | fall_promo | Backend |  | [passthrough] Reads `utm_campaign` from `public.fact_transaction`. |
| AN_120 | [PUB_028](#pub-028) | analytics.transactions_data_source | 1 row / transaction_guid | utm_content | varchar(255) | UTM content | banner_a | Backend |  | [passthrough] Reads `utm_content` from `public.fact_transaction`. |
| AN_121 | [PUB_029](#pub-029) | analytics.transactions_data_source | 1 row / transaction_guid | utm_term | varchar(255) | UTM term | movie+tickets | Backend |  | [passthrough] Reads `utm_term` from `public.fact_transaction`. |
| AN_122 | [PUB_030](#pub-030) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_rate | decimal(6,4) | PR ticket tax rate | 0.0825 | Backend |  | [passthrough] Reads `ticket_tax_rate` from `public.fact_transaction`. |
| AN_123 | [PUB_031](#pub-031) | analytics.transactions_data_source | 1 row / transaction_guid | is_ticket_tax_transaction | boolean | PR tax txn flag | true | Derived |  | [passthrough] Reads `is_ticket_tax_transaction` from `public.fact_transaction`. |

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
| <a id="raw-036"></a>RAW_036 |  | raw_import_cache.transactionPayment | 1 row / payment_guid | Payment_tp | varchar(40) | Payment type code | VISA | Backend |  | [passthrough] Reads `Payment_tp` from `raw_import_cache.transactionPayment`. |
| <a id="raw-037"></a>RAW_037 |  | raw_import_cache.transactionPayment | 1 row / payment_guid | Account_cd | varchar(600) | Payment account or instrument code | — | Backend |  | [passthrough] Reads `Account_cd` from `raw_import_cache.transactionPayment`. |
| <a id="raw-038"></a>RAW_038 |  | raw_import_cache.transactionPayment | 1 row / payment_guid | Transaction_at | numeric(10,4) | Payment amount | 42.50 | Backend |  | [passthrough] Reads `Transaction_at` from `raw_import_cache.transactionPayment`. |
| <a id="raw-039"></a>RAW_039 |  | raw_import_cache.transactionPayment | 1 row / payment_guid | Payment_GUID | varchar(36) | Payment identifier | — | Backend |  | [passthrough] Reads `Payment_GUID` from `raw_import_cache.transactionPayment`. |
| <a id="raw-040"></a>RAW_040 |  | raw_import_cache.processorbraintreeauthorizationresponse | 1 row / payment response | BraintreeTransaction_cd | varchar | Braintree authorization ID | — | Third-party |  | [passthrough] Reads `BraintreeTransaction_cd` from `raw_import_cache.processorbraintreeauthorizationresponse`. |
| <a id="raw-041"></a>RAW_041 |  | raw_import_cache.processorbraintreerefundresponse | 1 row / payment response | BraintreeTransaction_cd | varchar | Braintree refund ID | — | Third-party |  | [passthrough] Reads `BraintreeTransaction_cd` from `raw_import_cache.processorbraintreerefundresponse`. |
| <a id="raw-042"></a>RAW_042 |  | raw_import_cache.paypaltrrdetail | 1 row / processor transaction | transaction_id | varchar | PayPal TRR transaction ID | — | Third-party |  | [passthrough] Reads `transaction_id` from `raw_import_cache.paypaltrrdetail`. |
| <a id="raw-043"></a>RAW_043 |  | raw_import_cache.paypalstldetail | 1 row / processor transaction | transaction_id | varchar | PayPal STL transaction ID | — | Third-party |  | [passthrough] Reads `transaction_id` from `raw_import_cache.paypalstldetail`. |
| <a id="raw-044"></a>RAW_044 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | Item_qy | integer | Item quantity | 2 | Backend |  | [passthrough] Reads `Item_qy` from `raw_import_cache.transactionItem`. |
| <a id="raw-045"></a>RAW_045 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | Item_at | numeric | Item amount | 35.00 | Backend |  | [passthrough] Reads `Item_at` from `raw_import_cache.transactionItem`. |
| <a id="raw-046"></a>RAW_046 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | SalesFee_at | numeric | Sales fee per admission | 5.00 | Backend |  | [passthrough] Reads `SalesFee_at` from `raw_import_cache.transactionItem`. |
| <a id="raw-047"></a>RAW_047 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | WaivedSalesFee_at | numeric | Waived sales fee per admission | 0.00 | Backend |  | [passthrough] Reads `WaivedSalesFee_at` from `raw_import_cache.transactionItem`. |
| <a id="raw-048"></a>RAW_048 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | SurchargeFee_at | numeric | Surcharge fee per admission | 1.50 | Backend |  | [passthrough] Reads `SurchargeFee_at` from `raw_import_cache.transactionItem`. |
| <a id="raw-049"></a>RAW_049 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | ReservedFee_at | numeric | Reserved fee per admission | 0.00 | Backend |  | [passthrough] Reads `ReservedFee_at` from `raw_import_cache.transactionItem`. |
| <a id="raw-050"></a>RAW_050 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | CardFee_at | numeric | Card fee per admission | 0.00 | Backend |  | [passthrough] Reads `CardFee_at` from `raw_import_cache.transactionItem`. |
| <a id="raw-051"></a>RAW_051 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | Shipping_at | numeric | Shipping fee per admission | 0.00 | Backend |  | [passthrough] Reads `Shipping_at` from `raw_import_cache.transactionItem`. |
| <a id="raw-052"></a>RAW_052 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | taxable_at | numeric | Taxable sales-fee amount | 0.00 | Backend |  | [passthrough] Reads `taxable_at` from `raw_import_cache.transactionItem`. |
| <a id="raw-053"></a>RAW_053 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | totaladmissions_qy | integer | Admissions per item | 1 | Backend |  | [passthrough] Reads `totaladmissions_qy` from `raw_import_cache.transactionItem`. |
| <a id="raw-054"></a>RAW_054 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | Item_tp | varchar | Item type code | Donation | Backend |  | [passthrough] Reads `Item_tp` from `raw_import_cache.transactionItem`. |
| <a id="raw-055"></a>RAW_055 |  | raw_import_cache.transactionItem | 1 row / transaction_guid + item_guid | POS_TicketType_nm | varchar | Ticket type description | Adult Ticket | Backend |  | [passthrough] Reads `POS_TicketType_nm` from `raw_import_cache.transactionItem`. |
| <a id="raw-056"></a>RAW_056 |  | raw_import_cache.ticketTypeMapping | 1 row / chain_id + POS description | pos_description | varchar | POS description used in item labels | Adult Ticket | Backend |  | [lookup] Reads `pos_description` from `raw_import_cache.ticketTypeMapping`. |
| <a id="raw-057"></a>RAW_057 |  | raw_import_cache.ticketTypeMapping | 1 row / chain_id + POS description | chain_id | varchar | Chain ID for ticket mapping | 1 | Backend |  | [lookup] Reads `chain_id` from `raw_import_cache.ticketTypeMapping`. |
| <a id="raw-058"></a>RAW_058 |  | raw_import_cache.ticketTypeMapping | 1 row / chain_id + POS description | category_id | integer | Mapped ticket category ID | 10 | Backend |  | [lookup] Reads `category_id` from `raw_import_cache.ticketTypeMapping`. |
| <a id="raw-059"></a>RAW_059 |  | raw_import_cache.ticketCategory | 1 row / category_id | category_description | varchar | Ticket category description | Adult | Backend |  | [lookup] Reads `category_description` from `raw_import_cache.ticketCategory`. |
| <a id="raw-060"></a>RAW_060 |  | raw_import_cache.ticketTypeMapping | 1 row / chain_id + POS description | display_description | varchar | Display description used in item labels | Adult Ticket | Backend |  | [lookup] Reads `display_description` from `raw_import_cache.ticketTypeMapping`. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-031"></a>STG_031 | [RAW_036](#raw-036), [RAW_037](#raw-037) | warehouse_loader.fact_payment_aggregate | 1 row / transaction_guid | payment_types | varchar(200) | Payment types | VISA | Backend |  | [aggregate] Groups `public.fact_payment` by transaction; resolves labels through `public.dim_payment_instrument` and `public.dim_payment_type`. |
| <a id="stg-032"></a>STG_032 | [RAW_040](#raw-040), [RAW_041](#raw-041), [RAW_042](#raw-042), [RAW_043](#raw-043) | warehouse_loader.fact_payment_aggregate | 1 row / transaction_guid | processor_order_ids | varchar(124) | Processor order IDs | — | Third-party |  | [aggregate] Lists distinct `processor_order_id` values from `public.fact_payment` by transaction. |
| <a id="stg-033"></a>STG_033 | [RAW_038](#raw-038) | warehouse_loader.fact_payment_aggregate | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross payment amount | 42.50 | Backend |  | [aggregate] Sums `transaction_amount` from `public.fact_payment` by transaction. |
| <a id="stg-034"></a>STG_034 | [RAW_044](#raw-044), [RAW_055](#raw-055) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_count | integer | Non-subscription item count | 2 | Backend |  | [aggregate] Sums non-donation item quantities from `public.fact_transaction_item`, excluding subscription descriptions. |
| <a id="stg-035"></a>STG_035 | [RAW_044](#raw-044), [RAW_045](#raw-045) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_sales_amount | numeric(19,4) | Ticket sales amount | 70.00 | Backend |  | [aggregate] Sums item quantity × amount from non-donation rows in `public.fact_transaction_item`. |
| <a id="stg-036"></a>STG_036 | [RAW_044](#raw-044), [RAW_046](#raw-046), [RAW_049](#raw-049), [RAW_050](#raw-050), [RAW_051](#raw-051), [RAW_053](#raw-053) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_fees_amount | numeric(19,4) | Sales and item fees | 6.50 | Backend |  | [aggregate] Sums quantity × admissions × sales, reserved, card, and shipping fees from non-donation rows in `public.fact_transaction_item`. |
| <a id="stg-037"></a>STG_037 | [RAW_044](#raw-044), [RAW_048](#raw-048), [RAW_053](#raw-053) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_surcharge_fee_amount | numeric(19,4) | Surcharge fee amount | 1.50 | Backend |  | [aggregate] Sums quantity × admissions × surcharge fee from non-donation rows in `public.fact_transaction_item`. |
| <a id="stg-038"></a>STG_038 | [RAW_044](#raw-044), [RAW_047](#raw-047), [RAW_053](#raw-053) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Backend |  | [aggregate] Sums quantity × admissions × waived sales fee from non-donation rows in `public.fact_transaction_item`. |
| <a id="stg-039"></a>STG_039 | [RAW_044](#raw-044), [RAW_049](#raw-049), [RAW_053](#raw-053) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | fanclub_waived_sales_fee_amount | numeric(19,4) | Fanclub waived fee amount | 0.00 | Backend |  | [aggregate] Sums quantity × admissions × reserved fee from non-donation rows in `public.fact_transaction_item`. |
| <a id="stg-040"></a>STG_040 | [RAW_044](#raw-044), [RAW_047](#raw-047), [RAW_053](#raw-053) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | fee_waiver_item_count | integer | Waived sales-fee item count | 0 | Backend |  | [aggregate] Counts quantity × admissions where waived sales fee is positive in `public.fact_transaction_item`. |
| <a id="stg-041"></a>STG_041 | [RAW_044](#raw-044), [RAW_049](#raw-049), [RAW_053](#raw-053) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | fanclub_fee_waiver_item_count | integer | Fanclub waived-fee item count | 0 | Backend |  | [aggregate] Counts quantity × admissions where reserved fee is positive in `public.fact_transaction_item`. |
| <a id="stg-042"></a>STG_042 | [RAW_052](#raw-052) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | item_taxable_sales_fee_amount | numeric(19,4) | Taxable sales-fee amount | 0.00 | Backend |  | [aggregate] Sums taxable sales fees from non-donation rows in `public.fact_transaction_item`. |
| <a id="stg-043"></a>STG_043 | [RAW_045](#raw-045), [RAW_054](#raw-054) | warehouse_loader.fact_transaction_item_aggregate | 1 row / transaction_guid | charity_amount | numeric(19,4) | Donation amount | 1.00 | Backend |  | [aggregate] Sums item amounts for Donation rows in `public.fact_transaction_item`. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-032"></a>PUB_032 | [STG_031](#stg-031) | public.fact_transaction | 1 row / transaction_guid | payment_types | varchar(200) | Payment-type labels | VISA | Backend |  | [passthrough] Copies `payment_types` from `warehouse_loader.fact_payment_aggregate`. |
| <a id="pub-033"></a>PUB_033 | [STG_032](#stg-032) | public.fact_transaction | 1 row / transaction_guid | processor_order_ids | varchar(124) | Processor order IDs | — | Third-party |  | [passthrough] Copies `processor_order_ids` from `warehouse_loader.fact_payment_aggregate`. |
| <a id="pub-034"></a>PUB_034 | [STG_033](#stg-033) | public.fact_transaction | 1 row / transaction_guid | true_gross_transaction_amount | numeric(19,4) | Gross payment total before charity | 43.50 | Backend |  | [passthrough] Copies the gross payment total from `warehouse_loader.fact_payment_aggregate`. |
| <a id="pub-035"></a>PUB_035 | [STG_034](#stg-034) | public.fact_transaction | 1 row / transaction_guid | item_count | integer | Non-subscription item count | 2 | Backend |  | [passthrough] Copies `item_count` from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-036"></a>PUB_036 | [STG_036](#stg-036) | public.fact_transaction | 1 row / transaction_guid | sales_fee_amount | numeric(19,4) | Sales and item fees | 6.50 | Backend |  | [passthrough] Copies `item_fees_amount` from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-037"></a>PUB_037 | [STG_037](#stg-037) | public.fact_transaction | 1 row / transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee amount | 1.50 | Backend |  | [passthrough] Copies the surcharge aggregate from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-038"></a>PUB_038 | [STG_038](#stg-038) | public.fact_transaction | 1 row / transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Backend |  | [passthrough] Copies the waived-fee aggregate from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-039"></a>PUB_039 | [STG_039](#stg-039) | public.fact_transaction | 1 row / transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Fanclub waived fee amount | 0.00 | Backend |  | [passthrough] Copies the reserved-fee aggregate from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-040"></a>PUB_040 | [STG_040](#stg-040) | public.fact_transaction | 1 row / transaction_guid | waived_sales_fee_ticket_count | integer | Waived sales-fee item count | 0 | Backend |  | [passthrough] Copies the waiver count from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-041"></a>PUB_041 | [STG_041](#stg-041) | public.fact_transaction | 1 row / transaction_guid | waived_fanclub_fee_ticket_count | integer | Fanclub waived-fee item count | 0 | Backend |  | [passthrough] Copies the fanclub count from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-042"></a>PUB_042 | [STG_035](#stg-035) | public.fact_transaction | 1 row / transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 70.00 | Backend |  | [passthrough] Copies `item_sales_amount` from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-043"></a>PUB_043 | [STG_042](#stg-042), [STG_047](#stg-047) | public.fact_transaction | 1 row / transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales-fee amount | 0.00 | Backend |  | [conditional] Uses the item aggregate for purchases and `#fact_return_lookup` for refunds. |
| <a id="pub-044"></a>PUB_044 | [STG_043](#stg-043) | public.fact_transaction | 1 row / transaction_guid | charity_amount | numeric(19,4) | Charity donation amount | 1.00 | Backend |  | [passthrough] Copies `charity_amount` from `warehouse_loader.fact_transaction_item_aggregate`. |
| <a id="pub-045"></a>PUB_045 | [STG_033](#stg-033), [STG_043](#stg-043) | public.fact_transaction | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount excluding charity | 42.50 | Backend |  | [derived] Subtracts charity from gross payment total using the two aggregate tables. |
| <a id="pub-046"></a>PUB_046 | [STG_001](#stg-001) | public.fact_payment | 1 row / payment_guid | transaction_guid | varchar(36) | Transaction key on payment row | 9F2A... | Backend |  |  |
| <a id="pub-047"></a>PUB_047 | [RAW_038](#raw-038) | public.fact_payment | 1 row / payment_guid | transaction_amount | numeric | Payment amount | 42.50 | Backend |  |  |
| <a id="pub-048"></a>PUB_048 | [RAW_036](#raw-036), [RAW_037](#raw-037) | public.fact_payment | 1 row / payment_guid | payment_instrument_key | integer | Payment instrument key | — | Backend |  |  |
| <a id="pub-049"></a>PUB_049 | [RAW_040](#raw-040), [RAW_041](#raw-041), [RAW_042](#raw-042), [RAW_043](#raw-043) | public.fact_payment | 1 row / payment_guid | processor_order_id | varchar | Processor order ID | — | Third-party |  |  |
| <a id="pub-050"></a>PUB_050 | [STG_001](#stg-001) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | transaction_guid | varchar(36) | Transaction key on item row | 9F2A... | Backend |  | [passthrough] Uses `warehouse_loader.fact_transaction_base.transaction_guid` for each item. |
| <a id="pub-051"></a>PUB_051 | [RAW_055](#raw-055) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | ticket_type_description | varchar | Ticket type description | Adult Ticket | Backend |  | [passthrough] Copies `POS_TicketType_nm` from `raw_import_cache.transactionItem`. |
| <a id="pub-052"></a>PUB_052 | [RAW_044](#raw-044) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | item_count | integer | Item quantity | 2 | Backend |  | [passthrough] Copies `Item_qy` from `raw_import_cache.transactionItem`. |
| <a id="pub-053"></a>PUB_053 | [RAW_045](#raw-045) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | item_amount | numeric(19,4) | Item amount | 35.00 | Backend |  | [passthrough] Copies `Item_at` from `raw_import_cache.transactionItem`. |
| <a id="pub-054"></a>PUB_054 | [RAW_046](#raw-046) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | sales_fee_amount | numeric(19,4) | Sales fee per item | 5.00 | Backend |  | [passthrough] Copies `SalesFee_at` from `raw_import_cache.transactionItem`. |
| <a id="pub-055"></a>PUB_055 | [RAW_047](#raw-047) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee per item | 0.00 | Backend |  | [passthrough] Copies `WaivedSalesFee_at` from `raw_import_cache.transactionItem`. |
| <a id="pub-056"></a>PUB_056 | [RAW_048](#raw-048) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee per item | 1.50 | Backend |  | [passthrough] Copies `SurchargeFee_at` from `raw_import_cache.transactionItem`. |
| <a id="pub-057"></a>PUB_057 | [RAW_049](#raw-049) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | reserved_fee_amount | numeric(19,4) | Reserved fee per item | 0.00 | Backend |  | [passthrough] Copies `ReservedFee_at` from `raw_import_cache.transactionItem`. |
| <a id="pub-058"></a>PUB_058 | [RAW_050](#raw-050) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | card_fee_amount | numeric(19,4) | Card fee per item | 0.00 | Backend |  | [passthrough] Copies `CardFee_at` from `raw_import_cache.transactionItem`. |
| <a id="pub-059"></a>PUB_059 | [RAW_051](#raw-051) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | shipping_fee_amount | numeric(19,4) | Shipping fee per item | 0.00 | Backend |  | [passthrough] Copies `Shipping_at` from `raw_import_cache.transactionItem`. |
| <a id="pub-060"></a>PUB_060 | [RAW_052](#raw-052) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales-fee amount | 0.00 | Backend |  | [passthrough] Copies `taxable_at` from `raw_import_cache.transactionItem`. |
| <a id="pub-061"></a>PUB_061 | [RAW_053](#raw-053) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | admissions_count | integer | Admissions per item | 1 | Backend |  | [passthrough] Copies `totaladmissions_qy` from `raw_import_cache.transactionItem`. |
| <a id="pub-062"></a>PUB_062 | [RAW_054](#raw-054) | public.fact_transaction_item | 1 row / transaction_guid + item_guid | source_item_type | varchar | Source item type | Donation | Backend |  | [passthrough] Copies `Item_tp` from `raw_import_cache.transactionItem`. |

</details>

<details open>
<summary><strong>Analytics (13 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_020 | [PUB_032](#pub-032) | analytics.transactions_data_source | 1 row / transaction_guid | payment_types | varchar(200) | Payment types | VISA | Derived |  | [passthrough] Reads payment types from `public.fact_transaction`, populated by `warehouse_loader.fact_payment_aggregate`. |
| AN_021 | [PUB_033](#pub-033) | analytics.transactions_data_source | 1 row / transaction_guid | processor_order_ids | varchar(124) | Processor order ids | ord_123 | Third-party |  | [passthrough] Reads processor order IDs from `public.fact_transaction`, populated by `warehouse_loader.fact_payment_aggregate`. |
| AN_073 | [PUB_035](#pub-035) | analytics.transactions_data_source | 1 row / transaction_guid | item_qty | int | Item qty | 2 | Derived |  | [rename/cast] Renames `public.fact_transaction.item_count` to `item_qty`. |
| AN_074 | [PUB_045](#pub-045) | analytics.transactions_data_source | 1 row / transaction_guid | gross_transaction_amount | numeric(19,4) | Gross amount | 42.50 | Derived |  | [passthrough] Reads gross amount from `public.fact_transaction`; this is payment total less charity. |
| AN_075 | [PUB_036](#pub-036) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_amount | numeric(19,4) | Sales fee | 5.00 | Derived |  | [passthrough] Reads sales fees from `public.fact_transaction`, built from `warehouse_loader.fact_transaction_item_aggregate`. |
| AN_076 | [PUB_040](#pub-040) | analytics.transactions_data_source | 1 row / transaction_guid | waived_sales_fee_ticket_count | numeric(19,4) | Waived sales fee ticket count | 0 | Derived |  | [passthrough] Reads waived-fee count from `public.fact_transaction`, built from the item aggregate. |
| AN_079 | [PUB_042](#pub-042) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_sales_amount | numeric(19,4) | Ticket sales amount | 35.00 | Derived |  | [passthrough] Reads ticket sales from `public.fact_transaction`, built from the item aggregate. |
| AN_080 | [PUB_038](#pub-038) | analytics.transactions_data_source | 1 row / transaction_guid | waived_sales_fee_amount | numeric(19,4) | Waived sales fee amount | 0.00 | Derived |  | [passthrough] Reads waived sales fees from `public.fact_transaction`, built from the item aggregate. |
| AN_081 | [PUB_037](#pub-037) | analytics.transactions_data_source | 1 row / transaction_guid | surcharge_fee_amount | numeric(19,4) | Surcharge fee | 1.50 | Derived |  | [passthrough] Reads surcharge fees from `public.fact_transaction`, built from the item aggregate. |
| AN_099 | [PUB_039](#pub-039) | analytics.transactions_data_source | 1 row / transaction_guid | waived_fanclub_fee_amount | numeric(19,4) | Waived fanclub fee amount | 0.00 | Derived |  | [passthrough] Reads fanclub waived fees from `public.fact_transaction`, built from the item aggregate. |
| AN_100 | [PUB_041](#pub-041) | analytics.transactions_data_source | 1 row / transaction_guid | waived_fanclub_fee_ticket_count | numeric(19,4) | Waived fanclub fee ticket count | 0 | Derived |  | [passthrough] Reads fanclub waiver count from `public.fact_transaction`, built from the item aggregate. |
| AN_101 | [PUB_044](#pub-044) | analytics.transactions_data_source | 1 row / transaction_guid | charity_amount | numeric(19,4) | Charity amount | 1.00 | Derived |  | [passthrough] Reads charity amount from `public.fact_transaction`, built from the item aggregate. |
| AN_102 | [PUB_034](#pub-034) | analytics.transactions_data_source | 1 row / transaction_guid | true_gross_transaction_amount | numeric(19,4) | Gross including charity | 43.50 | Derived |  | [passthrough] Reads gross payment total from `public.fact_transaction`, populated by `warehouse_loader.fact_payment_aggregate`. |

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
| <a id="raw-061"></a>RAW_061 | raw_import_cache.refund | 1 row / refund line | Price_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| <a id="raw-062"></a>RAW_062 | raw_import_cache.refund | 1 row / refund line | Fee_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| <a id="raw-063"></a>RAW_063 | raw_import_cache.refund | 1 row / refund line | Tax_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| <a id="raw-064"></a>RAW_064 | raw_import_cache.refund | 1 row / refund line | Surcharge_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| <a id="raw-065"></a>RAW_065 | raw_import_cache.refund | 1 row / refund line | ReservedFee_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| <a id="raw-066"></a>RAW_066 | raw_import_cache.refund | 1 row / refund line | CardFee_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| <a id="raw-067"></a>RAW_067 | raw_import_cache.refund | 1 row / refund line | taxable_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |
| <a id="raw-068"></a>RAW_068 | raw_import_cache.refund | 1 row / refund line | donation_at | numeric | Return-side financial values | — | Backend |  | `factReturn.sql` groups refund lines by refund GUID, purchase GUID, return create datetime, and reason before inserting public.fact_return; downstream aggregates and refund-row updates use those values. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-044"></a>STG_044 | [RAW_061](#raw-061) | warehouse_loader.fact_return_aggregate | 1 row / purchase_transaction_guid | returned_ticket_amount | numeric | Purchase-side returned rollups | — | Derived |  | `[aggregate]` Transaction-level rollup of raw refund lineage. The ETL sums successful public.fact_return rows by purchase_transaction_guid. |
| <a id="stg-045"></a>STG_045 | [RAW_062](#raw-062), [RAW_065](#raw-065), [RAW_066](#raw-066) | warehouse_loader.fact_return_aggregate | 1 row / purchase_transaction_guid | returned_fee_amount | numeric | Purchase-side returned rollups | — | Derived |  | `[aggregate]` Transaction-level rollup of raw refund lineage. The ETL sums successful public.fact_return rows by purchase_transaction_guid. |
| <a id="stg-046"></a>STG_046 | [RAW_068](#raw-068) | warehouse_loader.fact_return_aggregate | 1 row / purchase_transaction_guid | returned_charity_amount | numeric | Purchase-side returned rollups | — | Derived |  | `[aggregate]` Transaction-level rollup of raw refund lineage. The ETL sums successful public.fact_return rows by purchase_transaction_guid. |
| <a id="stg-047"></a>STG_047 | [RAW_061](#raw-061), [RAW_062](#raw-062), [RAW_063](#raw-063), [RAW_064](#raw-064), [RAW_065](#raw-065), [RAW_066](#raw-066), [RAW_067](#raw-067), [RAW_068](#raw-068) | #fact_return_lookup (temp) | 1 row / return_transaction_guid | return_* | mixed | Return lookup set in run window | — | ETL |  | `[lookup]` Stages raw refund lineage for refund-row updates by copying return details from public.fact_return into #fact_return_lookup. |
| <a id="stg-048"></a>STG_048 | [RAW_081](#raw-081) | #fact_return_lookup (temp) | 1 row / return_transaction_guid | purchase_transaction_guid | mixed | Return lookup set in run window | — | ETL |  | `[lookup]` Stages raw refund lineage for refund-row updates by copying return details from public.fact_return into #fact_return_lookup. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-063"></a>PUB_063 | [STG_001](#stg-001) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_transaction_guid | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-064"></a>PUB_064 | [RAW_081](#raw-081) | public.fact_return | refund/purchase GUID, return datetime, and reason group | purchase_transaction_guid | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-065"></a>PUB_065 | [RAW_061](#raw-061) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-066"></a>PUB_066 | [RAW_062](#raw-062) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_fee_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-067"></a>PUB_067 | [RAW_063](#raw-063) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_tax_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-068"></a>PUB_068 | [RAW_064](#raw-064) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_surcharge_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-069"></a>PUB_069 | [RAW_065](#raw-065) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_reservation_fee_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-070"></a>PUB_070 | [RAW_066](#raw-066) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_card_fee_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-071"></a>PUB_071 | [RAW_068](#raw-068) | public.fact_return | refund/purchase GUID, return datetime, and reason group | return_charity_amount | mixed | Return facts consumed by return aggregation and refund updates | — | ETL |  | `[aggregate/lookup]` Groups raw_import_cache.refund by refund GUID, purchase GUID, create time, and reason. Joins warehouse_loader.fact_transaction_base, warehouse_loader.fact_payment_aggregate, warehouse_loader.fact_transaction_item_aggregate, public.dim_return_reason, public.dim_chain, public.dim_theater, public.dim_property, and public.dim_data_source to build public.fact_return. |
| <a id="pub-072"></a>PUB_072 | [STG_044](#stg-044), [STG_045](#stg-045), [STG_046](#stg-046) | public.fact_transaction | 1 row / purchase transaction_guid | returned_amount | numeric(19,4) | Returned values at purchase row | -42.50 | Derived |  | `[lookup]` Updates purchase rows from warehouse_loader.fact_return_aggregate, matched by purchase GUID. |
| <a id="pub-073"></a>PUB_073 | [STG_045](#stg-045) | public.fact_transaction | 1 row / purchase transaction_guid | returned_fee_amount | numeric(19,4) | Returned values at purchase row | -42.50 | Derived |  | `[lookup]` Updates purchase rows from warehouse_loader.fact_return_aggregate, matched by purchase GUID. |
| <a id="pub-074"></a>PUB_074 | [STG_046](#stg-046) | public.fact_transaction | 1 row / purchase transaction_guid | returned_charity_amount | numeric(19,4) | Returned values at purchase row | -42.50 | Derived |  | `[lookup]` Updates purchase rows from warehouse_loader.fact_return_aggregate, matched by purchase GUID. |
| <a id="pub-075"></a>PUB_075 | [STG_047](#stg-047) | public.fact_transaction | 1 row / refund transaction_guid | sales_fee_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| <a id="pub-076"></a>PUB_076 | [STG_047](#stg-047) | public.fact_transaction | 1 row / refund transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| <a id="pub-077"></a>PUB_077 | [STG_047](#stg-047) | public.fact_transaction | 1 row / refund transaction_guid | reservation_fee_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| <a id="pub-078"></a>PUB_078 | [STG_047](#stg-047) | public.fact_transaction | 1 row / refund transaction_guid | tax_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| <a id="pub-079"></a>PUB_079 | [STG_047](#stg-047) | public.fact_transaction | 1 row / refund transaction_guid | card_fee_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| <a id="pub-080"></a>PUB_080 | [STG_047](#stg-047) | public.fact_transaction | 1 row / refund transaction_guid | charity_return_amount | numeric(19,4) | Return amounts on refund row | -5.00 etc | Derived |  | `[derived]` Updates refund rows in public.fact_transaction from #fact_return_lookup, built from public.fact_return; writes return amounts as negatives. |
| <a id="pub-081"></a>PUB_081 | [STG_047](#stg-047), [STG_036](#stg-036) | public.fact_transaction | 1 row / refund transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Purchase fee amount tied to returned purchase | 5.00 | Derived |  | `[lookup]` Updates public.fact_transaction from warehouse_loader.fact_transaction_item_aggregate through #fact_return_lookup from public.fact_return. |

</details>

<details open>
<summary><strong>Analytics (11 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_077 | [PUB_014](#pub-014) | analytics.transactions_data_source | 1 row / transaction_guid | returned_amount | numeric(19,4) | Returned amount | -42.50 | Derived |  | [passthrough] Reads `is_test` from `public.fact_transaction`. |
| AN_078 | [PUB_014](#pub-014) | analytics.transactions_data_source | 1 row / transaction_guid | returned_fee_amount | numeric(19,4) | Returned fee amount | -5.00 | Derived |  | [passthrough] Reads `is_test` from `public.fact_transaction`. |
| AN_082 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_return_amount | numeric(19,4) | Sales fee return amount | -5.00 | Derived |  | [passthrough] Reads `deviceType` from `public.fact_transaction`. |
| AN_083 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | surcharge_fee_return_amount | numeric(19,4) | Surcharge fee return amount | -1.50 | Derived |  | [passthrough] Reads `deviceType` from `public.fact_transaction`. |
| AN_084 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | reservation_fee_return_amount | numeric(19,4) | Reservation fee return amount | 0.00 | Derived |  | [passthrough] Reads `deviceType` from `public.fact_transaction`. |
| AN_085 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | tax_return_amount | numeric(19,4) | Tax return amount | -2.10 | Derived |  | [passthrough] Reads `deviceType` from `public.fact_transaction`. |
| AN_086 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | card_fee_return_amount | numeric(19,4) | Card fee return amount | -0.40 | Derived |  | [passthrough] Reads `deviceType` from `public.fact_transaction`. |
| AN_087 | [PUB_016](#pub-016) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_amount_for_returned_purchase | numeric(19,4) | Sales fee amount for returned purchase | 5.00 | Derived |  | [passthrough] Reads `is_fanclub` from `public.fact_transaction`. |
| AN_091 | [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | taxable_sales_fee_amount | numeric(19,4) | Taxable sales fee amount | 5.00 | Derived |  | [passthrough] Reads `performance_datetime_local` from `public.fact_transaction`. |
| AN_103 | [PUB_014](#pub-014) | analytics.transactions_data_source | 1 row / transaction_guid | returned_charity_amount | numeric(19,4) | Returned charity amount | -1.00 | Derived |  | [passthrough] Reads `is_test` from `public.fact_transaction`. |
| AN_104 | [PUB_015](#pub-015) | analytics.transactions_data_source | 1 row / transaction_guid | charity_return_amount | numeric(19,4) | Charity return amount | -1.00 | Derived |  | [passthrough] Reads `deviceType` from `public.fact_transaction`. |

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
| <a id="raw-069"></a>RAW_069 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | names | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-070"></a>RAW_070 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | codes | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-071"></a>RAW_071 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | dates | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-072"></a>RAW_072 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | locations | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-073"></a>RAW_073 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | parent IDs | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-074"></a>RAW_074 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | vendor | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-075"></a>RAW_075 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | terminal attributes | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-076"></a>RAW_076 | raw_import_cache.affiliate, raw_import_cache.affiliateGroupMapping, raw_import_cache.affiliateGroup, raw_import_cache.affiliateCategory, raw_import_cache.movieDetail, raw_import_cache.movieDetailParent, raw_import_cache.movieDetailDeleted, raw_import_cache.filmRelatedFormatType, raw_import_cache.showtimeTMS, raw_import_cache.movieParentOverride, raw_import_cache.magicMovieIdMapping, raw_import_cache.universalBoxOfficeSales, raw_import_cache.movieProduction, raw_import_cache.chain, raw_import_cache.theater, raw_import_cache.theaterpos, raw_import_cache.postalCodeTimeZone, raw_import_cache.purchaseChannelStatus, raw_import_cache.returnType, raw_import_cache.transaction, raw_import_cache.transactionPayment, raw_import_cache.transactionItem, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute | dimension source grain | and partner labels | mixed | Raw sources for public dimensions used in projection | — | Backend/Third-party |  | The tier_1_dimensions SQL loads the relevant public.dim_* attributes from these raw tables; specific dimension source tables are named in each dimension loader. |
| <a id="raw-077"></a>RAW_077 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | Affiliate_id | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| <a id="raw-078"></a>RAW_078 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | Transaction_tp | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| <a id="raw-079"></a>RAW_079 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | UserAgent_nm | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| <a id="raw-080"></a>RAW_080 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | Property_nm | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| <a id="raw-081"></a>RAW_081 | raw_import_cache.transaction + raw_import_cache.refund | 1 row / transaction_guid | refund-to-purchase GUID | mixed | Natural keys/codes for dim resolution | 11879/WAP/... | Backend |  | Inputs to surrogate key mapping; `factTransactionBase.sql` follows refund-to-purchase GUIDs to fall back to the parent purchase property. |
| <a id="raw-082"></a>RAW_082 | raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | movie_detail_id | int | Movie/theater ids | 10293/4451 | Backend |  | Inputs to movie/theater keys. |
| <a id="raw-083"></a>RAW_083 | raw_import_cache.transactionPerformanceAttribute | 1 row / tx/perf | theater_pkey | int | Movie/theater ids | 10293/4451 | Backend |  | Inputs to movie/theater keys. |
| <a id="raw-084"></a>RAW_084 | raw_import_cache.transactionPurchaseFunnel | 1 row / tx | Funnel_cd | varchar | Funnel code | 14 | Backend |  | Input to purchase_funnel_key. |
| <a id="raw-085"></a>RAW_085 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (Channel) | varchar | Partner mapping attrs | TMO/WEBVIEW | Third-party |  | Inputs to partner dimension keys; refund rows can fall back to parent-purchase attributes. |
| <a id="raw-086"></a>RAW_086 | raw_import_cache.transactionAttribute | 1 row / tx + attr | value_tx (PartnerPlatform) | varchar | Partner mapping attrs | TMO/WEBVIEW | Third-party |  | Inputs to partner dimension keys; refund rows can fall back to parent-purchase attributes. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-049"></a>STG_049 | [RAW_082](#raw-082) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | movie_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-050"></a>STG_050 | [RAW_077](#raw-077) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | affiliate_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-051"></a>STG_051 | [RAW_070](#raw-070) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | chain_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-052"></a>STG_052 | [RAW_078](#raw-078) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | channel_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-053"></a>STG_053 | [RAW_084](#raw-084) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | purchase_funnel_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-054"></a>STG_054 | [RAW_008](#raw-008) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | return_type_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-055"></a>STG_055 | [RAW_083](#raw-083) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | theater_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-056"></a>STG_056 | [RAW_078](#raw-078) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | transaction_type_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-057"></a>STG_057 | [RAW_079](#raw-079) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | user_agent_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-058"></a>STG_058 | [RAW_080](#raw-080), [RAW_081](#raw-081) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | property_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-059"></a>STG_059 | [RAW_085](#raw-085) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | partner_channel_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |
| <a id="stg-060"></a>STG_060 | [RAW_086](#raw-086) | warehouse_loader.fact_transaction_base | 1 row / transaction_guid | partner_platform_key | bigint | Conformed surrogate keys | 8123 | ETL |  | `[lookup]` Resolves raw keys from raw_import_cache.transaction, raw_import_cache.transactionPerformanceAttribute, raw_import_cache.transactionCommitted, raw_import_cache.transactionPurchaseFunnel, raw_import_cache.transactionAttribute, raw_import_cache.transactionCustomer, and raw_import_cache.refund against public.dim_channel, public.dim_customer, public.dim_affiliate, public.dim_movie, public.dim_purchase_funnel, public.dim_return_type, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_email_address, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform; unmatched keys use unknown rows. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-082"></a>PUB_082 | [STG_049](#stg-049) | public.fact_transaction | 1 row / transaction_guid | movie_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies movie_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-083"></a>PUB_083 | [STG_050](#stg-050) | public.fact_transaction | 1 row / transaction_guid | affiliate_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies affiliate_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-084"></a>PUB_084 | [STG_051](#stg-051) | public.fact_transaction | 1 row / transaction_guid | chain_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies chain_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-085"></a>PUB_085 | [STG_052](#stg-052) | public.fact_transaction | 1 row / transaction_guid | channel_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies channel_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-086"></a>PUB_086 | [STG_053](#stg-053) | public.fact_transaction | 1 row / transaction_guid | purchase_funnel_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies purchase_funnel_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-087"></a>PUB_087 | [STG_054](#stg-054) | public.fact_transaction | 1 row / transaction_guid | return_type_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies return_type_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-088"></a>PUB_088 | [STG_055](#stg-055) | public.fact_transaction | 1 row / transaction_guid | theater_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies theater_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-089"></a>PUB_089 | [STG_056](#stg-056) | public.fact_transaction | 1 row / transaction_guid | transaction_type_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies transaction_type_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-090"></a>PUB_090 | [STG_057](#stg-057) | public.fact_transaction | 1 row / transaction_guid | user_agent_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies user_agent_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-091"></a>PUB_091 | [STG_058](#stg-058) | public.fact_transaction | 1 row / transaction_guid | property_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies property_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-092"></a>PUB_092 | [STG_059](#stg-059) | public.fact_transaction | 1 row / transaction_guid | partner_channel_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies partner_channel_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-093"></a>PUB_093 | [STG_060](#stg-060) | public.fact_transaction | 1 row / transaction_guid | partner_platform_key | bigint | Join key for dimension projection | — | ETL |  | `[passthrough]` Copies partner_platform_key from warehouse_loader.fact_transaction_base. |
| <a id="pub-094"></a>PUB_094 | [RAW_069](#raw-069), [RAW_070](#raw-070), [RAW_071](#raw-071) | public.dim_* tables | 1 row / dim key | descriptive attributes used by analytics | mixed | Lookup attributes | — | Backend/Third-party |  | `[lookup]` Analytics reads public.dim_affiliate, public.dim_movie, public.dim_channel, public.dim_chain, public.dim_theater, public.dim_transaction_type, public.dim_user_agent, public.dim_purchase_funnel, public.dim_return_type, public.dim_property, public.dim_partner_channel, and public.dim_partner_platform. Their tier_1_dimensions jobs load the raw inputs grouped in RAW_026. |

</details>

<details open>
<summary><strong>Analytics (52 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_022 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_name | varchar(400) | Affiliate name | Fandango iOS App | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_023 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_group_name | varchar(100) | Affiliate group name | Mobile | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_024 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_category_name | varchar(480) | Affiliate category name | O&O | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_025 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | affiliate_id | int | Affiliate id | 11879 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_026 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | channel | varchar(10) | Channel group | App | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_027 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | channel_ungrouped | varchar(40) | Channel name | WAP | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_028 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | filmid_child | int | Child movie id | 10293 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_029 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | filmname_child | varchar(400) | Child movie title | Example Movie | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_030 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | filmid_parent | varchar(32) | Parent movie id | 872 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_031 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | filmname_parent | varchar(400) | Parent movie title | Franchise X | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_032 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | releasedate | date | Release date | 2026-09-14 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_033 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | movie_tms_id | int | Movie TMS id | 123456 | Third-party |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_034 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | chainname | varchar(200) | Chain name | Example Cinemas | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_035 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | chainid | varchar(10) | Chain id | CHN019 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_036 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | pos_vendor | varchar(50) | POS vendor | Vista | Third-party |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_037 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | awards_program_name | varchar(320) | Awards program | Rewards+ | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_038 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | theater_id | int | Theater id | 4451 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_039 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | theater_code | varchar(10) | Theater code | TH4451 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_040 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | theater_name | varchar(200) | Theater name | Example Hollywood | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_041 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | pdi_id | varchar(6) | PDI id | AVSTA | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_042 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | pos_theater_id | varchar(20) | POS theater id | POS-4451 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_043 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | latitude | real | Latitude | 34.10 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_044 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | longitude | real | Longitude | -118.34 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_045 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | point_of_sale_id | smallint | POS id | 1 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_046 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | address_1 | varchar(200) | Address 1 | 123 Main St | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_047 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | address_2 | varchar(200) | Address 2 | Suite 200 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_048 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | city_name | varchar(160) | City | Los Angeles | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_049 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | state_code | varchar(2) | State code | CA | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_050 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | zip_code | varchar(10) | ZIP code | 90028 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_051 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | country_code | varchar(3) | Country code | US | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_052 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | tht_alternate_theater_phone_name | varchar(200) | Alternate phone/name | Box Office | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_053 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | chain_specific_site_id | varchar(10) | Chain site id | SITE-22 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_054 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | paymentech_terminal_id | varchar(24) | Paymentech terminal id | PT-100 | Third-party |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_055 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | cybersource_terminal_id | varchar(24) | Cybersource terminal id | CS-200 | Third-party |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_056 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | screen_count | smallint | Screen count | 12 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_057 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | seat_count | int | Seat count | 1800 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_058 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | theater_time_zone_name | varchar(50) | Theater timezone | America/Los_Angeles | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_060 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_type_id | varchar(40) | Transaction type id | Purchase | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_061 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_type_name | varchar(40) | Transaction type name | Ticket Purchase | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_062 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | user_agent_id | varchar(480) | User agent id | UA-1 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_063 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | user_agent_description | varchar(480) | User agent description | iOS Safari | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_064 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | purchase_funnel_id | varchar(40) | Funnel id | 14 | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_065 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | purchase_funnel_name | varchar(100) | Funnel name | Guest Checkout | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_066 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | return_method | varchar(20) | Return method | Self-service | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_067 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | return_type_code | varchar(3) | Return type code | REF | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_068 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | return_type_description | varchar(20) | Return type description | Refund | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_088 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | property_id | varchar(60) | Property id | Fandango USA | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_090 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | braintree_merchant_id | varchar(60) | Merchant id | bt_merch_1 | Third-party |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_108 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | partner_channel | varchar | Partner channel id | TMO | Third-party |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_109 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | partner_display_name | varchar | Partner display name | T-Mobile Tuesdays | Third-party |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_110 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | partner_platform | varchar | Partner platform id | WEBVIEW | Third-party |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |
| AN_112 | [PUB_018](#pub-018) | analytics.transactions_data_source | 1 row / transaction_guid | property_group | varchar | Property group | Fandango | Backend |  | [passthrough] Reads `charity_name` from `public.fact_transaction`. |


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
| <a id="pub-095"></a>PUB_095 | [RAW_071](#raw-071) | public.dim_movie | dim grain | movie_release_date | mixed | Inputs to DATEDIFF and CASE logic | — | Backend |  | `[lookup]` Reads attributes from public.dim_movie, public.dim_affiliate, and public.dim_channel for analytics calculations. |
| <a id="pub-096"></a>PUB_096 | [RAW_070](#raw-070) | public.dim_affiliate | dim grain | affiliate_id | mixed | Inputs to DATEDIFF and CASE logic | — | Backend |  | `[lookup]` Reads attributes from public.dim_movie, public.dim_affiliate, and public.dim_channel for analytics calculations. |
| <a id="pub-097"></a>PUB_097 | [RAW_069](#raw-069) | public.dim_channel | dim grain | channel_name | mixed | Inputs to DATEDIFF and CASE logic | — | Backend |  | `[lookup]` Reads attributes from public.dim_movie, public.dim_affiliate, and public.dim_channel for analytics calculations. |

</details>

<details open>
<summary><strong>Analytics (11 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_004 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_start_datetime_los_angeles | timestamp | Pacific start datetime | 2026-10-01 09:10:00 | Derived |  | [passthrough] Reads `transaction_guid` from `public.fact_transaction`. |
| AN_006 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_commit_datetime_los_angeles | timestamp | Pacific commit datetime | 2026-10-01 09:11:05 | Derived |  | [passthrough] Reads `transaction_guid` from `public.fact_transaction`. |
| AN_008 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_rollback_datetime_los_angeles | timestamp | Pacific rollback datetime | NULL | Derived |  | [passthrough] Reads `transaction_guid` from `public.fact_transaction`. |
| AN_010 | [PUB_001](#pub-001) | analytics.transactions_data_source | 1 row / transaction_guid | transaction_end_datetime_los_angeles | timestamp | Pacific end datetime | 2026-10-01 09:11:05 | Derived |  | [passthrough] Reads `transaction_guid` from `public.fact_transaction`. |
| AN_012 | [PUB_001](#pub-001), [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | deltareleasetopurchasedate | int | Days release->purchase day | 17 | Derived |  | [passthrough] Reads `transaction_guid` from `public.fact_transaction`. |
| AN_013 | [PUB_003](#pub-003), [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | deltareleasetoshowtimedate | int | Days release->showtime day | 21 | Derived |  | [passthrough] Reads `transaction_start_datetime` from `public.fact_transaction`. |
| AN_014 | [PUB_001](#pub-001), [PUB_003](#pub-003) | analytics.transactions_data_source | 1 row / transaction_guid | deltapurchasetoshowtimedate | int | Days purchase day->showtime day | 4 | Derived |  | [passthrough] Reads `transaction_guid` from `public.fact_transaction`. |
| AN_069 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | "Affiliate Grouping" | varchar(256) | Detailed affiliate bucket | iPhone App | Derived |  | [passthrough] Reads `partner_defined_data_value` from `public.fact_transaction`. |
| AN_070 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | "Affiliate Grouping - HIGH LEVEL BINS" | varchar(256) | High-level bucket | Fandango O&O | Derived |  | [passthrough] Reads `partner_defined_data_value` from `public.fact_transaction`. |
| AN_071 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | apps | varchar(256) | App bucket | Fandango App: iPhone | Derived |  | [passthrough] Reads `partner_defined_data_value` from `public.fact_transaction`. |
| AN_072 | [PUB_019](#pub-019) | analytics.transactions_data_source | 1 row / transaction_guid | platform | varchar(256) | Platform bucket | Mobile Apps | Derived |  | [passthrough] Reads `partner_defined_data_value` from `public.fact_transaction`. |

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
| <a id="raw-087"></a>RAW_087 | raw_import_cache.designatedMarketArea | 1 row / zip_code | zip_code | varchar | ZIP to DMA mapping | 90028 / Los Angeles | Third-party |  | External DMA source used in analytics join. |
| <a id="raw-088"></a>RAW_088 | raw_import_cache.designatedMarketArea | 1 row / zip_code | market_name | varchar | ZIP to DMA mapping | 90028 / Los Angeles | Third-party |  | External DMA source used in analytics join. |

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
| <a id="pub-098"></a>PUB_098 | [STG_055](#stg-055) | public.dim_theater | 1 row / theater_key | zip_code | varchar(10) | Theater ZIP code | 90028 | Backend |  | `[lookup]` Reads ZIP from public.dim_theater; analytics joins it to raw_import_cache.designatedMarketArea by ZIP. |

</details>

<details open>
<summary><strong>Analytics (1 field)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_059 | [PUB_020](#pub-020) | analytics.transactions_data_source | 1 row / transaction_guid | dma_market_name | varchar(255) | DMA market name | Los Angeles | Third-party |  | [passthrough] Reads `network_token_usage` from `public.fact_transaction`. |

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
| <a id="raw-084"></a>RAW_084 | data_lake.salestaxtransaction | line-item grain | document_number | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |
| <a id="raw-085"></a>RAW_085 | data_lake.salestaxtransaction | line-item grain | lineitem_number | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |
| <a id="raw-086"></a>RAW_086 | data_lake.salestaxtransaction | line-item grain | total_lineitem_tax | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |
| <a id="raw-087"></a>RAW_087 | data_lake.salestaxtransaction | line-item grain | tax area | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |
| <a id="raw-088"></a>RAW_088 | data_lake.salestaxtransaction | line-item grain | rate and status fields | mixed | Sales-tax feed used by the ticketing sales-tax load | — | Third-party |  | `salestax/sql/transactions_load.sql` loads this feed into raw_import_cache.stgsalestaxtransaction and builds public.fact_sales_fee_tax. |

</details>

<details>
<summary><strong>Staging</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="stg-061"></a>STG_061 | [RAW_084](#raw-084) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | document_number | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| <a id="stg-062"></a>STG_062 | [RAW_085](#raw-085) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | lineitem_number | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| <a id="stg-063"></a>STG_063 | [RAW_086](#raw-086) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | total_lineitem_tax | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| <a id="stg-064"></a>STG_064 | [RAW_087](#raw-087) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | tax area | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| <a id="stg-065"></a>STG_065 | [RAW_088](#raw-088) | raw_import_cache.stgsalestaxtransaction | line-item staging grain | rate and status fields | mixed | Raw sales-tax rows staged before fact construction | — | ETL |  | `[passthrough]` Loads data_lake.salestaxtransaction rows into raw_import_cache.stgsalestaxtransaction. |
| <a id="stg-066"></a>STG_066 | [STG_063](#stg-063) | temp #transactionSalesTax | 1 row / transaction_guid | sales_fee_tax_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| <a id="stg-067"></a>STG_067 | [STG_063](#stg-063) | temp #transactionSalesTax | 1 row / transaction_guid | returned_sales_fee_tax_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| <a id="stg-068"></a>STG_068 | [STG_063](#stg-063) | temp #transactionSalesTax | 1 row / transaction_guid | sales_fee_tax_return_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| <a id="stg-069"></a>STG_069 | [STG_063](#stg-063) | temp #transactionSalesTax | 1 row / transaction_guid | sales_fee_tax_amount_for_returned_purchase | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| <a id="stg-070"></a>STG_070 | [STG_063](#stg-063) | temp #transactionSalesTax | 1 row / transaction_guid | pr_ticket_tax_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| <a id="stg-071"></a>STG_071 | [STG_063](#stg-063) | temp #transactionSalesTax | 1 row / transaction_guid | returned_ticket_tax_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| <a id="stg-072"></a>STG_072 | [STG_063](#stg-063) | temp #transactionSalesTax | 1 row / transaction_guid | ticket_tax_return_amount | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |
| <a id="stg-073"></a>STG_073 | [STG_063](#stg-063) | temp #transactionSalesTax | 1 row / transaction_guid | ticket_tax_amount_for_returned_purchase | numeric(19,4) | Tax values staged for update | — | ETL |  | `[lookup]` Stages transaction-level tax lineage for delayed analytics refresh. The SQL snapshot is copied from recent public.fact_sales_fee_tax rows. |

</details>

<details>
<summary><strong>Public</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <a id="pub-099"></a>PUB_099 | [STG_063](#stg-063) | public.fact_sales_fee_tax | 1 row / transaction_guid | sales_fee_tax_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| <a id="pub-100"></a>PUB_100 | [STG_063](#stg-063) | public.fact_sales_fee_tax | 1 row / transaction_guid | returned_sales_fee_tax_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| <a id="pub-101"></a>PUB_101 | [STG_063](#stg-063) | public.fact_sales_fee_tax | 1 row / transaction_guid | sales_fee_tax_return_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| <a id="pub-102"></a>PUB_102 | [STG_063](#stg-063) | public.fact_sales_fee_tax | 1 row / transaction_guid | sales_fee_tax_amount_for_returned_purchase | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| <a id="pub-103"></a>PUB_103 | [STG_063](#stg-063) | public.fact_sales_fee_tax | 1 row / transaction_guid | pr_ticket_tax_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| <a id="pub-104"></a>PUB_104 | [STG_063](#stg-063) | public.fact_sales_fee_tax | 1 row / transaction_guid | returned_ticket_tax_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| <a id="pub-105"></a>PUB_105 | [STG_063](#stg-063) | public.fact_sales_fee_tax | 1 row / transaction_guid | ticket_tax_return_amount | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |
| <a id="pub-106"></a>PUB_106 | [STG_063](#stg-063) | public.fact_sales_fee_tax | 1 row / transaction_guid | ticket_tax_amount_for_returned_purchase | numeric(19,4) | Tax measures assembled by the salestax ETL for analytics consumption | — | Third-party/Derived |  | `[derived]` Builds public.fact_sales_fee_tax from raw_import_cache.stgsalestaxtransaction and raw_import_cache.stgsalestaxtransaction_unmatched, using public.fact_transaction, public.fact_return, public.dim_transaction_type, public.dim_theater, and public.dim_theater_sales_fee_tax_rate. |

</details>

<details open>
<summary><strong>Analytics (9 fields)</strong></summary>

| Field ID | Previous Stage ID(s) | Redshift Table | Table Grain | Column Name | Data Type | Description | Example Value | Source | GitHub Link | Logic Explanation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AN_092 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_amount | decimal(19,4) | Sales fee tax amount | 0.35 | Third-party/Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_093 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | returned_sales_fee_tax_amount | decimal(19,4) | Returned sales fee tax amount | -0.35 | Third-party/Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_094 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_return_amount | decimal(19,4) | Sales fee tax return amount | -0.35 | Third-party/Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_095 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | sales_fee_tax_amount_for_returned_purchase | decimal(19,4) | Sales fee tax amount for returned purchase | 0.35 | Third-party/Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_124 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_amount | decimal(19,4) | Vertex ticket tax amount | 3.50 | Third-party/Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_125 | [PUB_021](#pub-021), [PUB_008](#pub-008) | analytics.transactions_data_source | 1 row / transaction_guid | pre_tax_ticket_sales_amount | decimal(19,4) | Ticket sales amount before tax | 31.50 | Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_126 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | returned_ticket_tax_amount | decimal(19,4) | Returned ticket tax amount | -3.50 | Third-party/Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_127 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_return_amount | decimal(19,4) | Ticket tax return amount | -3.50 | Third-party/Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |
| AN_128 | [PUB_021](#pub-021) | analytics.transactions_data_source | 1 row / transaction_guid | ticket_tax_amount_for_returned_purchase | decimal(19,4) | Ticket tax amount for returned purchase | 3.50 | Third-party/Derived |  | [passthrough] Reads `is_plf` from `public.fact_transaction`. |


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


