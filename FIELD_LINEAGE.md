# analytics.transactions_data_source — Detailed Field Lineage

## Overview

This document traces the transformation of fields in `analytics.transactions_data_source` from the source system through each ETL layer:

MSSQL source
→ raw_import_cache
→ warehouse_loader.fact_transaction_base
→ public.fact_transaction
→ analytics.transactions_data_source

The final table is built by:
- the warehouse load jobs (extract from MSSQL and load raw tables),
- the base transaction fact build (warehouse_loader),
- the public fact load (`public.fact_transaction`),
- the analytics denormalization step (`analyticsTransactionsDataSource.sql`).

---

## Quick Navigation

- [Identity / Key Fields](#identity--key-fields)
- [Date / Time Fields](#date--time-fields)
- [Amount Fields](#amount-fields)
- [Dimension / Descriptive Fields](#dimension--descriptive-fields)
- [Flag Fields](#flag-fields)
- [Derived / Calculated / Business Logic Fields](#derived--calculated--business-logic-fields)

---

# Identity / Key Fields

## 1. transaction_guid

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_guid` |
| Data Type | `varchar` |
| Purpose | Unique transaction identifier, used to match across layers |
| Purchase Example | `ABC123` |
| Refund Example | Same `transaction_guid` as original purchase if refund is associated to it |

Lineage Flow:
```text
MSSQL.transaction.Transaction_GUID
    ↓
raw_import_cache.transaction.Transaction_GUID
    ↓
warehouse_loader.fact_transaction_base.transaction_guid
    ↓
public.fact_transaction.transaction_guid
    ↓
analytics.transactions_data_source.transaction_guid
```

Transformation Logic:
- Direct pass-through through all layers
- Used as the join key across multiple tables

SQL Pattern:
```sql
SELECT t.Transaction_GUID AS transaction_guid
FROM raw_import_cache.transaction t;
```

Source:
- `raw_import_cache.transaction`

---

## 2. transaction_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_id` |
| Data Type | `bigint` / `int` |
| Purpose | Transaction ID from the transaction system |
| Example | `354`, `355`, `358` |

Lineage Flow:
```text
MSSQL.transaction.transaction_id
    ↓
raw_import_cache.transaction.transaction_id
    ↓
warehouse_loader.fact_transaction_base.transaction_id
    ↓
public.fact_transaction.transaction_id
    ↓
analytics.transactions_data_source.transaction_id
```

Transformation Logic:
- Direct pass-through from raw transaction source to final analytics table

---

# Date / Time Fields

## 3. transaction_start_datetime_utc

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_start_datetime_utc` |
| Data Type | `timestamp` |
| Purpose | UTC start time of the transaction |

Lineage Flow:
```text
MSSQL.transaction.Create_dm
    ↓
raw_import_cache.transaction.Create_dm
    ↓
warehouse_loader.fact_transaction_base.transaction_start_datetime
    ↓
public.fact_transaction.transaction_start_datetime
    ↓
analytics.transactions_data_source.transaction_start_datetime_utc
```

Transformation Logic:
- Source transaction start time is preserved through the layers
- Used alongside end/commit/rollback timestamps for reporting

---

## 4. transaction_start_datetime_los_angeles

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_start_datetime_los_angeles` |
| Data Type | `timestamp` |
| Purpose | Start time converted to Pacific time |

Lineage Flow:
```text
public.fact_transaction.transaction_start_datetime
    ↓
CONVERT_TIMEZONE('US/Pacific', ...)
    ↓
analytics.transactions_data_source.transaction_start_datetime_los_angeles
```

Transformation Logic:
```sql
CONVERT_TIMEZONE('US/Pacific', ft.transaction_start_datetime)
```

Use case:
- Local scheduling and theater reporting in Pacific time

---

## 5. transaction_commit_datetime_utc

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_commit_datetime_utc` |
| Data Type | `timestamp` |
| Purpose | UTC timestamp when the transaction committed |

Lineage Flow:
```text
MSSQL.transactionCommitted.create_dm
    ↓
raw_import_cache.transactionCommitted.create_dm
    ↓
warehouse_loader.fact_transaction_base.transaction_commit_datetime
    ↓
public.fact_transaction.transaction_commit_datetime
    ↓
analytics.transactions_data_source.transaction_commit_datetime_utc
```

Transformation Logic:
- Usually populated when payment was successfully finalized
- Some rows remain NULL for cancelled or incomplete transactions

---

## 6. transaction_commit_datetime_los_angeles

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_commit_datetime_los_angeles` |
| Data Type | `timestamp` |
| Purpose | Commit time converted to Pacific time |

Transformation Logic:
```sql
CONVERT_TIMEZONE('US/Pacific', ft.transaction_commit_datetime)
```

---

## 7. transaction_rollback_datetime_utc

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_rollback_datetime_utc` |
| Data Type | `timestamp` |
| Purpose | UTC timestamp when the transaction was rolled back |

Lineage Flow:
```text
MSSQL.transactionRolledBack.create_dm
    ↓
raw_import_cache.transactionRolledBack.create_dm
    ↓
warehouse_loader.fact_transaction_base.transaction_rollback_datetime
    ↓
public.fact_transaction.transaction_rollback_datetime
    ↓
analytics.transactions_data_source.transaction_rollback_datetime_utc
```

Transformation Logic:
- This field is often NULL
- If present, it indicates the purchase was cancelled/reversed

---

## 8. transaction_rollback_datetime_los_angeles

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_rollback_datetime_los_angeles` |
| Data Type | `timestamp` |
| Purpose | Rollback time in Pacific time |

---

## 9. transaction_end_datetime_utc

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_end_datetime_utc` |
| Data Type | `timestamp` |
| Purpose | Final event time for the transaction in UTC |

Lineage Flow:
```text
MSSQL.transaction.Create_dm
+ MSSQL.transactionCommitted.create_dm
+ MSSQL.transactionRolledBack.create_dm
    ↓
raw_import_cache.transaction / committed / rolledback
    ↓
warehouse_loader.fact_transaction_base.transaction_end_datetime
    ↓
public.fact_transaction.transaction_end_datetime
    ↓
analytics.transactions_data_source.transaction_end_datetime_utc
```

Transformation Logic:
```sql
CASE
    WHEN tc.create_dm IS NOT NULL AND tc.create_dm >= COALESCE(tr.create_dm, tc.create_dm, t.Create_dm)
    THEN tc.create_dm
    WHEN tr.create_dm IS NOT NULL AND tr.create_dm >= COALESCE(tc.create_dm, tr.create_dm, t.Create_dm)
    THEN tr.create_dm
    ELSE t.Create_dm
END AS transaction_end_datetime
```

Business Meaning:
- If transaction was committed, use commit time
- If it was rolled back later, use rollback time
- Else use original transaction time

---

## 10. transaction_end_datetime_los_angeles

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_end_datetime_los_angeles` |
| Data Type | `timestamp` |
| Purpose | Final event time in Los Angeles / Pacific timezone |

Transformation Logic:
```sql
CONVERT_TIMEZONE('US/Pacific', ft.transaction_end_datetime)
```

Example:
- UTC: `2026-09-29 15:00:05`
- Los Angeles: `2026-09-29 08:00:05`

---

## 11. transaction_end_datetime_local

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_end_datetime_local` |
| Data Type | `timestamp` |
| Purpose | Local business timezone value when available |

Source:
- `public.fact_transaction.transaction_end_datetime_local`

---

## 12. performance_datetime_local

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `performance_datetime_local` |
| Data Type | `timestamp` |
| Purpose | Showtime/performance local timestamp |

Source:
- `public.fact_transaction.performance_datetime_local`

---

# Amount Fields

## 13. gross_transaction_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `gross_transaction_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Gross transaction amount before returns and refund adjustments |

Lineage Flow:
```text
raw_import_cache.payment tables / transaction aggregates
    ↓
warehouse_loader.fact_transaction_base.gross_transaction_amount
    ↓
public.fact_transaction.gross_transaction_amount
    ↓
analytics.transactions_data_source.gross_transaction_amount
```

Transformation Logic:
- Aggregated from line item and payment fact calculations
- May reflect transaction value before taxes/returns depending on business logic

Example:
- Purchase: `42.50`
- Refund: `-42.50`

---

## 14. sales_fee_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `sales_fee_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Sales fee amount collected on the purchase |

Source:
- `public.fact_transaction.sales_fee_amount`

---

## 15. waived_sales_fee_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `waived_sales_fee_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Amount of sales fee waived |

Source:
- `public.fact_transaction.waived_sales_fee_amount`

---

## 16. waived_sales_fee_ticket_count

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `waived_sales_fee_ticket_count` |
| Data Type | `int` |
| Purpose | Number of tickets on which sales fees were waived |

Source:
- `public.fact_transaction.waived_sales_fee_ticket_count`

---

## 17. returned_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `returned_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Total returned/refunded amount |

Lineage Flow:
```text
public.fact_return
    ↓
fact_transaction return update logic
    ↓
analytics.transactions_data_source.returned_amount
```

Transformation Logic:
- Populated from return data, often after initial insert
- A late-update pattern is used in `analyticsTransactionsDataSource.sql`

---

## 18. returned_fee_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `returned_fee_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Fee portion returned to the customer |

Source:
- `public.fact_return`
- then updated in the analytics layer

---

## 19. ticket_sales_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `ticket_sales_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Net ticket sales amount |

Source:
- `public.fact_transaction.ticket_sales_amount`

---

## 20. surcharge_fee_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `surcharge_fee_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Surcharge amount applied to transaction |

Source:
- `public.fact_transaction.surcharge_fee_amount`

---

## 21. sales_fee_return_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `sales_fee_return_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Returned sales fee amount |

Source:
- `public.fact_transaction.sales_fee_return_amount`

---

## 22. surcharge_fee_return_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `surcharge_fee_return_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Returned surcharge fee amount |

Source:
- `public.fact_transaction.surcharge_fee_return_amount`

---

## 23. reservation_fee_return_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `reservation_fee_return_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Returned reservation fee amount |

Source:
- `public.fact_transaction.reservation_fee_return_amount`

---

## 24. tax_return_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `tax_return_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Tax portion returned due to refund |

Source:
- `public.fact_transaction.tax_return_amount`

---

## 25. card_fee_return_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `card_fee_return_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Card fee portion returned |

Source:
- `public.fact_transaction.card_fee_return_amount`

---

## 26. sales_fee_amount_for_returned_purchase

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `sales_fee_amount_for_returned_purchase` |
| Data Type | `numeric(19,4)` |
| Purpose | Sales fee associated with a purchase that was later returned |

Source:
- `public.fact_transaction.sales_fee_amount_for_returned_purchase`

---

## 27. taxable_sales_fee_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `taxable_sales_fee_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Taxable sales fee amount |

Source:
- `public.fact_transaction.taxable_sales_fee_amount`

---

## 28. charity_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `charity_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Charity amount included on the transaction |

Source:
- `public.fact_transaction.charity_amount`

---

## 29. true_gross_transaction_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `true_gross_transaction_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Gross transaction amount before charity-related deductions |

Source:
- `public.fact_transaction.true_gross_transaction_amount`

---

## 30. returned_charity_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `returned_charity_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Charity amount associated with a returned transaction |

Source:
- `public.fact_transaction.returned_charity_amount`

---

## 31. charity_return_amount

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `charity_return_amount` |
| Data Type | `numeric(19,4)` |
| Purpose | Charity return value for refund transactions |

Source:
- `public.fact_transaction.charity_return_amount`

---

# Dimension / Descriptive Fields

## 32. affiliate_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `affiliate_name` |
| Data Type | `varchar` |
| Purpose | Affiliate/channel name associated with traffic source |

Lineage Flow:
```text
raw_import_cache.transaction.Affiliate_id
    ↓
public.dim_affiliate
    ↓
public.fact_transaction.affiliate_key
    ↓
analytics.transactions_data_source.affiliate_name
```

Transformation Logic:
- Join on `affiliate_key`
- Use `dim_affiliate.affiliate_name`

---

## 33. affiliate_group_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `affiliate_group_name` |
| Data Type | `varchar` |
| Purpose | Business grouping of affiliate source |

Source:
- `public.dim_affiliate.affiliate_group_name`

---

## 34. affiliate_category_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `affiliate_category_name` |
| Data Type | `varchar` |
| Purpose | Category of affiliate source |

Source:
- `public.dim_affiliate.affiliate_category_name`

---

## 35. affiliate_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `affiliate_id` |
| Data Type | `int` |
| Purpose | Affiliate ID |

Source:
- `public.dim_affiliate.affiliate_id`

---

## 36. channel

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `channel` |
| Data Type | `varchar` |
| Purpose | Channel grouping |

Source:
- `public.dim_channel.channel_group`

---

## 37. channel_ungrouped

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `channel_ungrouped` |
| Data Type | `varchar` |
| Purpose | Ungrouped channel label |

Source:
- `public.dim_channel.channel_name`

---

## 38. filmid_child

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `filmid_child` |
| Data Type | `int` |
| Purpose | Movie ID for child title |

Source:
- `public.dim_movie.movie_id`

---

## 39. filmname_child

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `filmname_child` |
| Data Type | `varchar` |
| Purpose | Movie title child film |

Source:
- `public.dim_movie.movie_title`

---

## 40. filmid_parent

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `filmid_parent` |
| Data Type | `int` |
| Purpose | Parent movie ID |

Source:
- `public.dim_movie.movie_parent_id`

---

## 41. filmname_parent

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `filmname_parent` |
| Data Type | `varchar` |
| Purpose | Parent movie title |

Source:
- `public.dim_movie.movie_parent_title`

---

## 42. releasedate

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `releasedate` |
| Data Type | `date` |
| Purpose | Release date of the movie |

Source:
- `public.dim_movie.movie_release_date`

---

## 43. movie_tms_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `movie_tms_id` |
| Data Type | `varchar` |
| Purpose | TMS movie identifier |

Source:
- `public.dim_movie.movie_tms_id`

---

## 44. chainname

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `chainname` |
| Data Type | `varchar` |
| Purpose | Marketing chain name |

Source:
- `public.dim_chain.chain_name`

---

## 45. chainid

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `chainid` |
| Data Type | `varchar` |
| Purpose | Chain ID |

Source:
- `public.dim_chain.chain_id`

---

## 46. pos_vendor

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `pos_vendor` |
| Data Type | `varchar` |
| Purpose | POS vendor name |

Source:
- `public.dim_chain.point_of_sale_vendor`

---

## 47. awards_program_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `awards_program_name` |
| Data Type | `varchar` |
| Purpose | Awards program name |

Source:
- `public.dim_chain.awards_program_name`

---

## 48. theater_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `theater_id` |
| Data Type | `int` |
| Purpose | Theater ID |

Source:
- `public.dim_theater.theater_id`

---

## 49. theater_code

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `theater_code` |
| Data Type | `varchar` |
| Purpose | Theater code |

Source:
- `public.dim_theater.theater_code`

---

## 50. theater_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `theater_name` |
| Data Type | `varchar` |
| Purpose | Theater name |

Source:
- `public.dim_theater.theater_name`

---

## 51. pdi_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `pdi_id` |
| Data Type | `varchar` |
| Purpose | Theater / location PDI ID |

Source:
- `public.dim_theater.pdi_id`

---

## 52. pos_theater_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `pos_theater_id` |
| Data Type | `varchar` |
| Purpose | POS theater ID |

Source:
- `public.dim_theater.point_of_sale_theater_id`

---

## 53. latitude

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `latitude` |
| Data Type | `numeric` |
| Purpose | Theater latitude |

Source:
- `public.dim_theater.latitude`

---

## 54. longitude

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `longitude` |
| Data Type | `numeric` |
| Purpose | Theater longitude |

Source:
- `public.dim_theater.longitude`

---

## 55. point_of_sale_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `point_of_sale_id` |
| Data Type | `varchar` |
| Purpose | POS device / terminal identifier |

Source:
- `public.dim_theater.point_of_sale_id`

---

## 56. address_1

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `address_1` |
| Data Type | `varchar` |
| Purpose | Theater physical address |

Source:
- `public.dim_theater.address_1`

---

## 57. address_2

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `address_2` |
| Data Type | `varchar` |
| Purpose | Theater address line 2 |

Source:
- `public.dim_theater.address_2`

---

## 58. city_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `city_name` |
| Data Type | `varchar` |
| Purpose | Theater city |

Source:
- `public.dim_theater.city_name`

---

## 59. state_code

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `state_code` |
| Data Type | `varchar` |
| Purpose | Theater state |

Source:
- `public.dim_theater.state_code`

---

## 60. zip_code

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `zip_code` |
| Data Type | `varchar` |
| Purpose | Theater ZIP |

Source:
- `public.dim_theater.zip_code`

Then joined to:
```sql
LEFT JOIN raw_import_cache.designatedMarketArea dma
    ON dma.zip_code = tht.zip_code
```

to populate:
- `dma_market_name`

---

## 61. country_code

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `country_code` |
| Data Type | `varchar` |
| Purpose | Theater country |

Source:
- `public.dim_theater.country_code`

---

## 62. dma_market_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `dma_market_name` |
| Data Type | `varchar` |
| Purpose | Designated market area name |

Lineage Flow:
```text
raw_import_cache.designatedMarketArea.zip_code
    ↓
public.dim_theater.zip_code
    ↓
analytics.transactions_data_source.dma_market_name
```

---

## 63. transaction_type_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_type_id` |
| Data Type | `varchar` |
| Purpose | Transaction type code/id |

Source:
- `public.dim_transaction_type.transaction_type_id`

---

## 64. transaction_type_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `transaction_type_name` |
| Data Type | `varchar` |
| Purpose | Human-readable transaction type |

Source:
- `public.dim_transaction_type.transaction_type_name`

---

## 65. user_agent_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `user_agent_id` |
| Data Type | `varchar` |
| Purpose | User agent ID |

Source:
- `public.dim_user_agent.user_agent_id`

---

## 66. user_agent_description

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `user_agent_description` |
| Data Type | `varchar` |
| Purpose | Description of user agent |

Source:
- `public.dim_user_agent.user_agent_description`

---

## 67. purchase_funnel_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `purchase_funnel_id` |
| Data Type | `varchar` |
| Purpose | Purchase funnel code |

Source:
- `public.dim_purchase_funnel.purchase_funnel_id`

---

## 68. purchase_funnel_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `purchase_funnel_name` |
| Data Type | `varchar` |
| Purpose | Purchase funnel label |

Source:
- `public.dim_purchase_funnel.purchase_funnel_name`

---

## 69. return_method

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `return_method` |
| Data Type | `varchar` |
| Purpose | How the return was processed |

Source:
- `public.dim_return_type.return_method`

---

## 70. return_type_code

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `return_type_code` |
| Data Type | `varchar` |
| Purpose | Return type code |

Source:
- `public.dim_return_type.return_type_code`

---

## 71. return_type_description

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `return_type_description` |
| Data Type | `varchar` |
| Purpose | Human readable return type |

Source:
- `public.dim_return_type.return_type_description`

---

## 72. property_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `property_id` |
| Data Type | `varchar` |
| Purpose | Property / marketplace identifier |

Source:
- `public.dim_property.property_id`

---

## 73. property_group

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `property_group` |
| Data Type | `varchar` |
| Purpose | Property group |

Source:
- `public.dim_property.property_group`

---

## 74. POS_reference_cd

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `POS_reference_cd` |
| Data Type | `varchar` |
| Purpose | POS reference code |

Source:
- `public.fact_transaction.POS_reference_cd`

---

## 75. braintree_merchant_id

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `braintree_merchant_id` |
| Data Type | `varchar` |
| Purpose | Braintree merchant identifier |

Source:
- `public.dim_theater.braintree_merchant_id`

---

# Flag Fields

## 76. is_successful

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_successful` |
| Data Type | `boolean` |
| Purpose | Whether the transaction succeeded |

Transformation Logic:
```sql
CASE WHEN tc.Transaction_GUID IS NOT NULL AND tr.Transaction_GUID IS NULL THEN TRUE ELSE FALSE END
```

Example:
- Purchase: `true`
- Rolled back: `false`

---

## 77. is_test

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_test` |
| Data Type | `boolean` |
| Purpose | Marks QA or test transactions |

Source:
- `public.fact_transaction.is_test`

---

## 78. is_fanclub

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_fanclub` |
| Data Type | `boolean` |
| Purpose | Whether transaction is from FanClub |

Source:
- `public.fact_transaction.is_fanclub`

---

## 79. is_buynow

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_buynow` |
| Data Type | `boolean` |
| Purpose | Whether the transaction was a Buy Now flow |

Source:
- `public.fact_transaction.is_buynow`

---

## 80. network_token_usage

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `network_token_usage` |
| Data Type | `varchar` / `boolean` |
| Purpose | Whether network token was used |

Source:
- `public.fact_transaction.network_token_usage`

---

## 81. is_plf

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_plf` |
| Data Type | `boolean` |
| Purpose | PLF amenity flag |

Source:
- `public.fact_transaction.is_plf`

---

## 82. is_imax

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_imax` |
| Data Type | `boolean` |
| Purpose | IMAX flag |

Source:
- `public.fact_transaction.is_imax`

---

## 83. is_3d

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_3d` |
| Data Type | `boolean` |
| Purpose | 3D flag |

Source:
- `public.fact_transaction.is_3d`

---

## 84. is_higher_fee_amenity

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_higher_fee_amenity` |
| Data Type | `boolean` |
| Purpose | Whether the transaction included a higher-fee amenity |

Source:
- `public.fact_transaction.is_higher_fee_amenity`

---

# Derived / Calculated / Business Logic Fields

## 85. deltareleasetopurchasedate

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `deltareleasetopurchasedate` |
| Data Type | `int` |
| Purpose | Days from movie release date to purchase date |

Transformation Logic:
```sql
DATEDIFF(day, film.movie_release_date, DATE_TRUNC('day', CONVERT_TIMEZONE('US/Pacific', ft.transaction_end_datetime)))
```

---

## 86. deltareleasetoshowtimedate

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `deltareleasetoshowtimedate` |
| Data Type | `int` |
| Purpose | Days from movie release to performance date |

---

## 87. deltapurchasetoshowtimedate

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `deltapurchasetoshowtimedate` |
| Data Type | `int` |
| Purpose | Days from purchase to showtime |

---

## 88. point_of_sale_confirmation_code

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `point_of_sale_confirmation_code` |
| Data Type | `varchar` |
| Purpose | Confirmation code from POS |

Source:
- `public.fact_transaction.point_of_sale_confirmation_code`

---

## 89. is_reserved_seating

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `is_reserved_seating` |
| Data Type | `boolean` |
| Purpose | Whether reservation seating was used |

Source:
- `public.fact_transaction.is_reserved_seating`

---

## 90. reserved_seats

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `reserved_seats` |
| Data Type | `varchar` / `int` |
| Purpose | Reserved seat counts |

Source:
- `public.fact_transaction.reserved_seats`

---

## 91. loyalty_code

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `loyalty_code` |
| Data Type | `varchar` |
| Purpose | Loyalty code / loyalty identifier |

Source:
- `public.fact_transaction.loyalty_code`

---

## 92. payment_types

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `payment_types` |
| Data Type | `varchar` |
| Purpose | Payment method description |

Source:
- `public.fact_transaction.payment_types`

---

## 93. processor_order_ids

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `processor_order_ids` |
| Data Type | `varchar` |
| Purpose | Processor order IDs |

Source:
- `public.fact_transaction.processor_order_ids`

---

## 94. item_qty

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `item_qty` |
| Data Type | `int` |
| Purpose | Item quantity on the transaction |

Source:
- `public.fact_transaction.item_count`

---

## 95. charity_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `charity_name` |
| Data Type | `varchar` |
| Purpose | Charity name associated with the transaction |

Source:
- `public.fact_transaction.charity_name`

---

## 96. partner_defined_data_value

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `partner_defined_data_value` |
| Data Type | `varchar` |
| Purpose | Partner-specific metadata |

Source:
- `public.fact_transaction.partner_defined_data_value`

---

## 97. partner_channel

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `partner_channel` |
| Data Type | `varchar` |
| Purpose | Partner channel name |

Source:
- `public.dim_partner_channel.partner_channel_id` and `partner_display_name`

---

## 98. partner_display_name

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `partner_display_name` |
| Data Type | `varchar` |
| Purpose | Human-readable partner display name |

Source:
- `public.dim_partner_channel.partner_display_name`

---

## 99. partner_platform

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `partner_platform` |
| Data Type | `varchar` |
| Purpose | Partner platform label |

Source:
- `public.dim_partner_platform.partner_platform_id`

---

## 100. "Affiliate Grouping"

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `"Affiliate Grouping"` |
| Data Type | `varchar` |
| Purpose | High-level classification of affiliate source |

Transformation Logic:
- Complex CASE statement using affiliate IDs
- Categories include:
  - iPhone App
  - Android App
  - Fandango O&O
  - Portal: Google OneBox
  - Portal: IMDb
  - White Label: AMC
  - Other Referrers

---

## 101. "Affiliate Grouping - HIGH LEVEL BINS"

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `"Affiliate Grouping - HIGH LEVEL BINS"` |
| Data Type | `varchar` |
| Purpose | Coarser high-level affiliate bin |

Example:
- `Fandango O&O`
- `Other Non-O&O`

---

## 102. apps

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `apps` |
| Data Type | `varchar` |
| Purpose | App classification |

Example:
- `Non-App`
- `Fandango App: iPhone`
- `Mobile Apps`

---

## 103. platform

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `platform` |
| Data Type | `varchar` |
| Purpose | Browser/platform categorization |

Example:
- `Web`
- `Mobile Web`
- `Other`

---

## 104. deviceType

| Aspect | Value |
|---|---|
| Final Table | `analytics.transactions_data_source` |
| Column | `deviceType` |
| Data Type | `varchar` |
| Purpose | Device classification |

Source:
- `public.fact_transaction.deviceType`

---

# How to Use This Document

## 1. Search by field name
Open the file in GitHub or VS Code and press Ctrl+F.
Search values like:
- `transaction_end_datetime_los_angeles`
- `gross_transaction_amount`
- `is_successful`
- `affiliate_name`
- `filmname_child`

---

## 2. Follow the flow
Every field has:
- original source
- raw_import_cache table
- warehouse_loader table
- public table
- analytics table

This is the main rule:
```text
source → raw_import_cache → warehouse_loader → public → analytics
```

---

## 3. Look for transformation type
Each field falls into one of the following:
- direct pass-through
- business CASE logic
- dimension lookup
- aggregate addition
- timezone conversion
- late return update

That helps answer:
- “Who created this value?”
- “Where was it transformed?”
- “Is it a dimension, fact, or derived field?”

---

## 4. Use this for audits
This is very useful when you need to answer:
- Where does this field come from?
- Did it change at the public layer or analytics layer?
- Is this a raw source field or a derived field?
- Which fact table or dimension is responsible?

---

## 5. Extend it by adding links
If you store this in a repo or wiki, add links to:
- the exact SQL file
- the DAG file
- the raw source table
- the dimension table (if relevant)

Example:
```markdown
[analyticsTransactionsDataSource.sql](...)  
[factTransactionBase.sql](...)  
[factTransaction.sql](...)
```

---

# Summary

The underlying pattern is consistent:

```text
MSSQL tables
    ↓
raw_import_cache
    ↓
warehouse_loader.fact_transaction_base
    ↓
public.fact_transaction
    ↓
analytics.transactions_data_source
```

The final analytics table is a denormalized, business-friendly version of the fact data:
- flattened dimensions
- timezone converted datetimes
- business-only classifications
- return adjustments
- aggregated amounts

If you want, the next step I can do is:
1. create a version organized into a searchable table with all 96 fields in a single CSV/Markdown table
2. turn this into an HTML prototype with a search box
3. build a field-by-field map for specifically:
   - `transaction_end_datetime_los_angeles`
   - `is_successful`
   - `gross_transaction_amount`
   - `affiliate_name`
   - `filmname_child`
   - `returned_amount`

If you want, I can do the next version as a compact table-only format instead of long-form narrative.
