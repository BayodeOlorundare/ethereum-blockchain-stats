![This is an image](https://bayodeolorundare.com/wp-content/uploads/2023/02/tableau_hero.jpg)

# Ethereum Blockchain Stats

[*Dune dashboard can be accessed here*](https://dune.com/babylondon_204/ethereum-blockchain-stats)

**Ethereum Price Counter**

```sql
SELECT date_trunc('day', minute) AS time
, AVG(price) AS price
FROM prices.usd
WHERE symbol='ETH'
AND contract_address IS NULL
GROUP BY 1
```

**Ethereum Market Capitalisation**

```sql
WITH eth_supply AS (
  SELECT
    CAST(120529053 AS decimal) - SUM(CAST(eb.base_fee_per_gas AS decimal) * CAST(et.gas_used AS decimal)) / 1e18 + /* missing ETH2 rewards for now, awaiting beacon chain data, using estimated 1600 ETH staking issuance /day for now */ COUNT(eb.number) * 1600 / (
      24 * 60 * 60 / 12
    ) AS eth_supply
  FROM ethereum.transactions AS et
  INNER JOIN ethereum.blocks AS eb
    ON eb."number" = et.block_number
  WHERE
    et.block_time >= CAST('2022-09-29' AS TIMESTAMP)
), latest_eth_price AS (
  SELECT
    CAST(price AS decimal) AS eth_price
  FROM prices.usd
  WHERE
    symbol = 'ETH'
  ORDER BY
    minute DESC
  LIMIT 1
)
SELECT
  (
    eth_supply * eth_price
  ) / 1e9 AS eth_mcap
FROM eth_supply, latest_eth_price;
```

**Ethereum Price Line Chart**

```sql
SELECT date_trunc('day', minute) AS time
, AVG(price) AS price
FROM prices.usd
WHERE symbol='ETH'
AND contract_address IS NULL
GROUP BY 1
```

**Ethereum Addresses Counter**

```sql
SELECT COUNT(distinct to)/1e6 AS users
FROM ethereum.traces
WHERE success
```

**Ethereum Addresses Mixed Chart (Bar and Line Graph)**

```sql
SELECT 
  time,
  SUM(users) AS new_wallets,
  SUM(SUM(users)) OVER (ORDER BY time RANGE UNBOUNDED PRECEDING) AS cum_created_wallets
FROM (
  SELECT 
    date_trunc('week', creation_date) AS time,
    COUNT(users) AS users
  FROM (
    SELECT "to" AS users,
           MIN(block_time) AS creation_date
    FROM ethereum.traces
    WHERE tx_success
    GROUP BY "to"
  ) subquery
  GROUP BY date_trunc('week', creation_date)
) subquery2
GROUP BY time;
```

**Ethereum Users**

```sql
SELECT
  DATE_TRUNC('week', block_time) AS time,
  COUNT(DISTINCT "from") AS users,
  SUM(COUNT(*)) OVER (ORDER BY date_trunc('week', block_time) RANGE UNBOUNDED PRECEDING) AS cum_users
FROM ethereum.transactions
WHERE
  block_time < DATE_TRUNC('week', (
    SELECT
      MAX(block_time)
    FROM ethereum.transactions
  ))
GROUP BY
  1
```

**Ethereum Transactions Counter**

```sql
SELECT COUNT(*)/1e9 AS tx_count
FROM ethereum.transactions
```

**Ethereum Transactions Count Weekly Mixed Graph (Area and Line Graph)**

```sql
SELECT 
  date_trunc('week', block_time) AS time,
  COUNT(*) AS tx_count,
  SUM(COUNT(*)) OVER (ORDER BY date_trunc('week', block_time) RANGE UNBOUNDED PRECEDING) AS cum_tx_count
FROM ethereum.transactions
WHERE block_time < date_trunc('week', NOW())
GROUP BY date_trunc('week', block_time)
```

**Ethereum Blocks Counter**

```sql
SELECT MAX(number)/1e6 AS blocks
FROM ethereum.blocks
```
**Ethereum Total Number of Smart Contracts Deployed per Week and Number of Unique Addresses that Accessed a Smart Contract per Week**

```sql
SELECT  
  date_trunc('week', block_time) AS time,
  COUNT(distinct "to") AS smart_contracts_deployed,
  COUNT(distinct "from") AS unique_addresses_accessed_contract,
  SUM(COUNT(*)) OVER (ORDER BY date_trunc('week', block_time) RANGE UNBOUNDED PRECEDING) AS cum_smart_contracts_deployed
FROM ethereum.transactions
WHERE block_time < date_trunc('week', NOW()) and
    "to" IS NOT NULL
GROUP BY date_trunc('week', block_time)
```

**Created Smart Contracts Counter**

```sql
SELECT COUNT(*)/1e6 AS contracts
FROM ethereum.traces
WHERE type = 'create'
```
