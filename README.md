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

### 3. Block Production Metrics

```
SELECT 
  DATE_TRUNC('hour', block_timestamp) as "hour",
  COUNT(*) as "blocks produced",
  AVG(tx_count) as "avg txs per block",
  AVG(gas_used) as "avg gas used",
  AVG(size) as "avg block size"
FROM rise.testnet.fact_blocks
WHERE block_timestamp >= '2025-03-04'
GROUP BY 1
ORDER BY 1;
```

### 4. Top Contract Activity Metrics

```
SELECT 
  c.name as "contract name",
  c.symbol as "symbol",
  COUNT(DISTINCT t.tx_hash) as "total interactions",
  COUNT(DISTINCT t.from_address) as "unique users",
  COUNT(DISTINCT DATE_TRUNC('day', t.block_timestamp)) as "active days"
FROM rise.testnet.fact_transactions t
JOIN rise.testnet.dim_contracts c ON t.to_address = c.address
WHERE t.block_timestamp >= '2025-03-04'
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 20;
```

### 5. Top Active Users

```
with top_20_addresses as 
(SELECT 
  from_address as "user",
  COUNT(DISTINCT tx_hash) as "total transactions",
  COUNT(DISTINCT DATE_TRUNC('day', block_timestamp)) as "no. of active days",
  SUM(value) as "total value transferred",
  SUM(tx_fee) as "total fees paid"
FROM rise.testnet.fact_transactions
WHERE block_timestamp >= '2025-03-04'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 20)

select 
    dense_rank() over(order by "total transactions" desc) as "user ranking based on total transactions",
    *
from top_20_addresses
order by "total transactions" desc;
```

### 6. Top Events

```
with top_events as

(SELECT 
  contract_address as "contract",
  topic_0 as "event type",
  COUNT(*) as "total events",
  COUNT(DISTINCT tx_hash) as "total transactions",
  COUNT(DISTINCT DATE_TRUNC('day', block_timestamp)) as "days active"
FROM rise.testnet.fact_event_logs
WHERE block_timestamp >= '2025-03-04'
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 20)


select 
    dense_rank() over(order by "total events" desc) as "event ranking based on total events",
    *
from top_events
order by "total events" desc;
```
