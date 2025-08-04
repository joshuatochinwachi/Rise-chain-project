# Rise Chain Observatory: Network Performance & Usage Metrics
[Full Dashboard](https://joshuatochinwachi.github.io/Rise-chain-project/top-events/index.html) | [Data story telling on 𝕏](https://x.com/defi__josh/status/1914724744252850586)

This dashboard and entire analysis was built and developed using [Flipside Crypto](https://flipsidecrypto.xyz) by me. Unfortunately, Flipside’s querying and dashboarding tool has gone dark because of their collaboration with [Snowflake](https://www.snowflake.com/en/).

Flipside also used Snowflake’s SQL dialect as their SQL flavor when they were active, so the SQL code you’ll see in this project—for both querying and dashboarding—is very similar to Snowflake’s syntax.

Using [Python scripts](https://github.com/joshuatochinwachi/Flipside_dashboard_porter), I scraped my Flipside dashboard and visualizations from the Flipside website.

## Queries/Metrics used

### 1. First activity on Rise chain

```
select 
    min(block_timestamp) as "date & time of first activity on Rise chain"
from rise.testnet.fact_transactions
```

### 2. Network Activity Overview

```
WITH daily_metrics AS (
  SELECT 
    DATE_TRUNC('day', block_timestamp) as date,
    COUNT(DISTINCT tx_hash) as total_transactions,
    SUM(COUNT(DISTINCT tx_hash)) OVER(ORDER BY DATE_TRUNC('day', block_timestamp)) as cumulative_tx_count,
    COUNT(DISTINCT from_address) as unique_users,
    SUM(value) as total_native_value_transferred,
    AVG(tx_fee) as avg_transaction_fee,
    SUM(gas_used) as total_gas_used
  FROM rise.testnet.fact_transactions
  WHERE block_timestamp >= '2025-03-04'
  GROUP BY 1
  ORDER BY 1
)
SELECT 
  date as "day",
  total_transactions as "no. of transactions",
  cumulative_tx_count as "cumulative transactions",
  unique_users as "no. of unique users",
  total_native_value_transferred as "total native value transferred",
  avg_transaction_fee as "avg transaction fee",
  total_gas_used as "total gas used"
FROM daily_metrics
order by 1 desc;
```
