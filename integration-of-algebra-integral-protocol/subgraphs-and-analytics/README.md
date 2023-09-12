---
ID: '0'
title: Subgraphs and analytics
---

# Subgraphs and analytics

## Introduction

This section is devoted to obtaining data about an instance of the Algebra Integral protocol from the corresponding indexer (The Graph procotol subgraphs).

This page describes the entity schemas in a subgraph.&#x20;

Examples of requests are given on the subpage: [Examples of queries](examples-of-queries.md)

## Subgraph Schema

### Factory entity

```graphql
{
  # factory address
  id: ID!
  # amount of pools created
  poolCount: BigInt!
  # amoutn of transactions all time
  txCount: BigInt!
  # total volume all time in derived USD
  totalVolumeUSD: BigDecimal!
  # total volume all time in derived Matic
  totalVolumeMatic: BigDecimal!
  # total swap fees all time in USD
  totalFeesUSD: BigDecimal!
  # total swap fees all time in USD
  totalFeesMatic: BigDecimal!
  # all volume even through less reliable USD values
  untrackedVolumeUSD: BigDecimal!
  # TVL derived in USD
  totalValueLockedUSD: BigDecimal!
  # value that will be set in the pool when the pool is created 
  defaultCommunityFee: BigInt!
  # TVL derived in Matic
  totalValueLockedMatic: BigDecimal!
  # TVL derived in USD untracked
  totalValueLockedUSDUntracked: BigDecimal!
  # TVL derived in Matic untracked
  totalValueLockedMaticUntracked: BigDecimal!
  # current owner of the factory
  owner: ID!
}
```

### Bundle entity

```graphql
{
  id: ID!
  # price of Matic in usdgr
  maticPriceUSD: BigDecimal!
}
```

### Token entity

```graphql
{
  # token address
  id: ID!
  # token symbol
  symbol: String!
  # token name
  name: String!
  # token decimals
  decimals: BigInt!
  # token total supply
  totalSupply: BigInt!
  # volume in token units
  volume: BigDecimal!
  # volume in derived USD
  volumeUSD: BigDecimal!
  # volume in USD even on pools with less reliable USD values
  untrackedVolumeUSD: BigDecimal!
  # fees in USD
  feesUSD: BigDecimal!
  # transactions across all pools that include this token
  txCount: BigInt!
  # number of pools containing this token
  poolCount: BigInt!
  # liquidity across all pools in token units
  totalValueLocked: BigDecimal!
  # liquidity across all pools in derived USD
  totalValueLockedUSD: BigDecimal!
  # TVL derived in USD untracked
  totalValueLockedUSDUntracked: BigDecimal!
  # derived price in Matic
  derivedMatic: BigDecimal!
  # pools token is in that are white listed for USD pricing
  whitelistPools: [Pool!]!
  # derived fields
  tokenDayData: [TokenDayData!]! @derivedFrom(field: "token")
}
```

### Pool entity

```graphql
{
  # pool address
  id: ID!
  # creation
  createdAtTimestamp: BigInt!
  # block pool was created at
  createdAtBlockNumber: BigInt!
  # token0
  token0: Token!
  # token1
  token1: Token!
  # fee amount
  fee: BigInt!
  communityFee: BigInt!
  # in range liquidity
  liquidity: BigInt!
  # current price tracker
  sqrtPrice: BigInt!
  # tracker for global fee growth
  feeGrowthGlobal0X128: BigInt!
  # tracker for global fee growth
  feeGrowthGlobal1X128: BigInt!
  # token0 per token1
  token0Price: BigDecimal!
  # token1 per token0
  token1Price: BigDecimal!
  # current tick
  tick: BigInt!
  # current observation index
  observationIndex: BigInt!
  # all time token0 swapped
  volumeToken0: BigDecimal!
  # all time token1 swapped
  volumeToken1: BigDecimal!
  # all time USD swapped
  volumeUSD: BigDecimal!
  # all time USD swapped, unfiltered for unreliable USD pools
  untrackedVolumeUSD: BigDecimal!
  # fees in USD
  feesUSD: BigDecimal!
  # tick spacing
  tickSpacing: BigInt!
  # untracked fees in usd
  untrackedFeesUSD: BigDecimal!
  # all time number of transactions
  txCount: BigInt!
  # all time fees collected token0
  collectedFeesToken0: BigDecimal!
  # all time fees collected token1
  collectedFeesToken1: BigDecimal!
  # all time fees collected derived USD
  collectedFeesUSD: BigDecimal!
  # total token 0 across all ticks
  totalValueLockedToken0: BigDecimal!
  # total token 1 across all ticks
  totalValueLockedToken1: BigDecimal!
  feesToken0: BigDecimal!
  feesToken1: BigDecimal!
  # tvl derived Matic
  totalValueLockedMatic: BigDecimal!
  # tvl USD
  totalValueLockedUSD: BigDecimal!
  # TVL derived in USD untracked
  totalValueLockedUSDUntracked: BigDecimal!
  # Fields used to help derived relationship
  liquidityProviderCount: BigInt! # used to detect new exchanges
  # hourly snapshots of pool data
  poolHourData: [PoolHourData!]! @derivedFrom(field: "pool")
  # daily snapshots of pool data
  poolDayData: [PoolDayData!]! @derivedFrom(field: "pool")
  # derived fields
  mints: [Mint!]! @derivedFrom(field: "pool")
  burns: [Burn!]! @derivedFrom(field: "pool")
  swaps: [Swap!]! @derivedFrom(field: "pool")
  collects: [Collect!]! @derivedFrom(field: "pool")
  ticks: [Tick!]! @derivedFrom(field: "pool")
}
```

### PoolPosition entity

```graphql
type PoolPosition @entity {
  id: ID!
  pool: Pool!
  lowerTick: Tick!
  upperTick: Tick!
  owner: Bytes!
  liquidity: BigInt!
}
```

### Tick entity

```graphql
{
  "format: <pool address>#<tick index>”
  id: ID!
  "pool address”
  poolAddress: String
  "tick index”
  tickIdx: BigInt!
  "pointer to pool”
  pool: Pool!
  "total liquidity pool has as tick lower or upper”
  liquidityGross: BigInt!
  "how much liquidity changes when tick crossed”
  liquidityNet: BigInt!
  "calculated price of token0 of tick within this pool - constant”
  price0: BigDecimal!
  "calculated price of token1 of tick within this pool - constant”
  price1: BigDecimal!
  "lifetime volume of token0 with this tick in range”
  volumeToken0: BigDecimal!
  "lifetime volume of token1 with this tick in range”
  volumeToken1: BigDecimal!
  "lifetime volume in derived USD with this tick in range”
  volumeUSD: BigDecimal!
  "lifetime volume in untracked USD with this tick in range”
  untrackedVolumeUSD: BigDecimal!
  "fees in USD”
  feesUSD: BigDecimal!
  "all time collected fees in token0”
  collectedFeesToken0: BigDecimal!
  "all time collected fees in token1”
  collectedFeesToken1: BigDecimal!
  "all time collected fees in USD”
  collectedFeesUSD: BigDecimal!
  "created time”
  createdAtTimestamp: BigInt!
  "created block”
  createdAtBlockNumber: BigInt!
  "Fields used to help derived relationship”
  liquidityProviderCount: BigInt! "used to detect new exchanges
  "derived fields”
  swaps: [Swap!]! @derivedFrom(field: "tick")
  "vars needed for fee computation”
  feeGrowthOutside0X128: BigInt!
  feeGrowthOutside1X128: BigInt!
}
```

### Position entity

```graphql
{
  # Positions created through NonfungiblePositionManager
  # NFT token id
  id: ID!
  # owner of the NFT
  owner: Bytes!
  # pool position is within
  pool: Pool!
  # allow indexing by tokens
  token0: Token!
  # allow indexing by tokens
  token1: Token!
  # lower tick of the position
  tickLower: Tick!
  # upper tick of the position
  tickUpper: Tick!
  # total position liquidity
  liquidity: BigInt!
  # amount of token 0 ever deposited to position
  depositedToken0: BigDecimal!
  # amount of token 1 ever deposited to position
  depositedToken1: BigDecimal!
  # amount of token 0 ever withdrawn from position (without fees)
  withdrawnToken0: BigDecimal!
  # amount of token 1 ever withdrawn from position (without fees)
  withdrawnToken1: BigDecimal!
  # all time collected token0 (withdrawnToken0 + collectedFeesToken0)
  collectedToken0: BigDecimal!
  # all time collected token1 (withdrawnToken1 + collectedFeesToken1)
  collectedToken1: BigDecimal!
  # all time collected fees in token0
  collectedFeesToken0: BigDecimal!
  # all time collected fees in token1
  collectedFeesToken1: BigDecimal!
  # tx in which the position was initialized
  transaction: Transaction!
  # vars needed for fee computation
  feeGrowthInside0LastX128: BigInt!
  feeGrowthInside1LastX128: BigInt!

  token0Tvl : BigDecimal
  token1Tvl : BigDecimal 
}
```

### PositionSnapshot entity

```graphql
{
  "<NFT token id>#<block number>”
  id: ID!
  "owner of the NFT”
  owner: Bytes!
  "pool the position is within”
  pool: Pool!
  "position of which the snap was taken of”
  position: Position!
  "block in which the snap was created”
  blockNumber: BigInt!
  "timestamp of block in which the snap was created”
  timestamp: BigInt!
  "total position liquidity”
  liquidity: BigInt!
  "amount of token 0 ever deposited to position”
  depositedToken0: BigDecimal!
  "amount of token 1 ever deposited to position”
  depositedToken1: BigDecimal!
  "amount of token 0 ever withdrawn from position (without fees)”
  withdrawnToken0: BigDecimal!
  "amount of token 1 ever withdrawn from position (without fees)”
  withdrawnToken1: BigDecimal!
  "all time collected fees in token0”
  collectedFeesToken0: BigDecimal!
  "all time collected fees in token1”
  collectedFeesToken1: BigDecimal!
  "tx in which the snapshot was initialized”
  transaction: Transaction!
  "internal vars needed for fee computation”
  feeGrowthInside0LastX128: BigInt!
  feeGrowthInside1LastX128: BigInt!
}
```

### Transaction entity

```graphql
{
  "txn hash”
  id: ID!
  "block txn was included in”
  blockNumber: BigInt!
  "timestamp txn was confirmed”
  timestamp: BigInt!
  "gas used during txn execution”
  gasLimit: BigInt!
  gasPrice: BigInt!
  "derived values”
  mints: [Mint!]! @derivedFrom(field: "transaction")
  burns: [Burn!]! @derivedFrom(field: "transaction")
  swaps: [Swap!]! @derivedFrom(field: "transaction")
  flashed: [Flash!]! @derivedFrom(field: "transaction")
  collects: [Collect!]! @derivedFrom(field: "transaction")
}
```

### Mint entity

```graphql
{
  "transaction hash + "#" + index in mints Transaction array”
  id: ID!
  "which txn the mint was included in”
  transaction: Transaction!
  "time of txn”
  timestamp: BigInt!
  "pool position is within”
  pool: Pool!
  "allow indexing by tokens”
  token0: Token!
  "allow indexing by tokens”
  token1: Token!
  "owner of position where liquidity minted to”
  owner: Bytes!
  "the address that minted the liquidity”
  sender: Bytes
  "txn origin”
  origin: Bytes! "the EOA that initiated the txn
  "amount of liquidity minted”
  amount: BigInt!
  "amount of token 0 minted”
  amount0: BigDecimal!
  "amount of token 1 minted”
  amount1: BigDecimal!
  "derived amount based on available prices of tokens”
  amountUSD: BigDecimal
  "lower tick of the position”
  tickLower: BigInt!
  "upper tick of the position”
  tickUpper: BigInt!
  "order within the txn”
  logIndex: BigInt
}
```

### Burn entity

```graphql
{
  "transaction hash + "#" + index in mints Transaction array”
  id: ID!
  "txn burn was included in”
  transaction: Transaction!
  "pool position is within”
  pool: Pool!
  "allow indexing by tokens”
  token0: Token!
  "allow indexing by tokens”
  token1: Token!
  "need this to pull recent txns for specific token or pool”
  timestamp: BigInt!
  "owner of position where liquidity was burned”
  owner: Bytes
  "txn origin”
  origin: Bytes! "the EOA that initiated the txn
  "amouny of liquidity burned”
  amount: BigInt!
  "amount of token 0 burned”
  amount0: BigDecimal!
  "amount of token 1 burned”
  amount1: BigDecimal!
  "derived amount based on available prices of tokens”
  amountUSD: BigDecimal
  "lower tick of position”
  tickLower: BigInt!
  "upper tick of position”
  tickUpper: BigInt!
  "position within the transactions”
  logIndex: BigInt
}
```

### Swap entity

```graphql
{
  "transaction hash + "#" + index in swaps Transaction array”
  id: ID!
  "pointer to transaction”
  transaction: Transaction!
  "timestamp of transaction”
  timestamp: BigInt!
  "pool swap occured within”
  pool: Pool!
  "allow indexing by tokens”
  token0: Token!
  "allow indexing by tokens”
  token1: Token!
  "sender of the swap”
  sender: Bytes!
  "recipient of the swap”
  recipient: Bytes!
  "txn origin”
  origin: Bytes! "the EOA that initiated the txn
  "delta of token0 swapped”
  amount0: BigDecimal!
  "delta of token1 swapped”
  amount1: BigDecimal!
  "derived info”
  amountUSD: BigDecimal!
  "The sqrt(price) of the pool after the swap, as a Q64.96”
  price: BigInt!
  "the tick after the swap”
  tick: BigInt!
  "index within the txn”
  logIndex: BigInt
}
```

### Collect entity

```graphql
{
  "transaction hash + "#" + index in collect Transaction array”
  id: ID!
  "pointer to txn”
  transaction: Transaction!
  "timestamp of event”
  timestamp: BigInt!
  "pool collect occured within”
  pool: Pool!
  "owner of position collect was performed on”
  owner: Bytes
  "amount of token0 collected”
  amount0: BigDecimal!
  "amount of token1 collected”
  amount1: BigDecimal!
  "derived amount based on available prices of tokens”
  amountUSD: BigDecimal
  "lower tick of position”
  tickLower: BigInt!
  "uppper tick of position”
  tickUpper: BigInt!
  "index within the txn”
  logIndex: BigInt
}
```

### Flash entity

```graphql
{
  "transaction hash + "-" + index in collect Transaction array
  id: ID!
  "pointer to txn”
  transaction: Transaction!
  "timestamp of event”
  timestamp: BigInt!
  "pool collect occured within”
  pool: Pool!
  "sender of the flash”
  sender: Bytes!
  "recipient of the flash”
  recipient: Bytes!
  "amount of token0 flashed”
  amount0: BigDecimal!
  "amount of token1 flashed”
  amount1: BigDecimal!
  "derived amount based on available prices of tokens”
  amountUSD: BigDecimal!
  "amount token0 paid for flash”
  amount0Paid: BigDecimal!
  "amount token1 paid for flash”
  amount1Paid: BigDecimal!
  "index within the txn”
  logIndex: BigInt
}
```



### AlgebraDayData entity

Data accumulated and condensed into day stats for each pool

```graphql
{
  "timestamp rounded to current day by dividing by 86400”
  id: ID!
  "timestamp rounded to current day by dividing by 86400”
  date: Int!
  "total daily volume in Algebra derived in terms of Matic”
  volumeMatic: BigDecimal!
  "total daily volume in Algebra derived in terms of USD”
  volumeUSD: BigDecimal!
  "total daily volume in Algebra derived in terms of USD untracked”
  volumeUSDUntracked: BigDecimal!
  "fees in USD”
  feesUSD: BigDecimal!
  "number of daily transactions”
  txCount: BigInt!
  "tvl in terms of USD”
  tvlUSD: BigDecimal!
}
```



### PoolDayData entity

```graphql
{
  # timestamp rounded to current day by dividing by 86400
  id: ID!
  # timestamp rounded to current day by dividing by 86400
  date: Int!
  # pointer to pool
  pool: Pool!
  # in range liquidity at end of period
  liquidity: BigInt!
  # current price tracker at end of period
  sqrtPrice: BigInt!
  #
  untrackedVolumeUSD: BigDecimal!
  # price of token0 - derived from sqrtPrice
  token0Price: BigDecimal!
  # price of token1 - derived from sqrtPrice
  token1Price: BigDecimal!
  # current tick at end of period
  tick: BigInt
  # tracker for global fee growth
  feeGrowthGlobal0X128: BigInt!
  # tracker for global fee growth
  feeGrowthGlobal1X128: BigInt!
  # tvl derived in USD at end of period
  tvlUSD: BigDecimal!
  feesToken0: BigDecimal!
  feesToken1: BigDecimal!
  # volume in token0
  volumeToken0: BigDecimal!
  # volume in token1
  volumeToken1: BigDecimal!
  # volume in USD
  volumeUSD: BigDecimal!
  # fees in USD
  feesUSD: BigDecimal!
  # numebr of transactions during period
  txCount: BigInt!
  # opening price of token0
  open: BigDecimal!
  # high price of token0
  high: BigDecimal!
  # low price of token0
  low: BigDecimal!
  # close price of token0
  close: BigDecimal!
}
```

### PoolFeeData entity

```graphql
{
  “format: <pool address><timestamp>”
  id: ID!
  “pool address”
  pool: String
  “change timestamp”
  timestamp: BigInt!
  “fee amount, where 1 = 0.0001%”
  fee: BigInt!
}
```

### PoolHourData entity

```graphql
{
  "format: <pool address>-<timestamp>”
  id: ID!
  "unix timestamp for start of hour”
  periodStartUnix: Int!
  "pointer to pool”
  pool: Pool!
  "in range liquidity at end of period”
  liquidity: BigInt!
  "current price tracker at end of period”
  sqrtPrice: BigInt!
  "price of token0 - derived from sqrtPrice”
  token0Price: BigDecimal!
  "price of token1 - derived from sqrtPrice”
  token1Price: BigDecimal!
  "current tick at end of period”
  tick: BigInt
  "tracker for global fee growth”
  feeGrowthGlobal0X128: BigInt!
  "tracker for global fee growth”
  feeGrowthGlobal1X128: BigInt!
  "tvl derived in USD at end of period”
  tvlUSD: BigDecimal!
  "volume in token0”
  volumeToken0: BigDecimal!
  "volume in token1”
  volumeToken1: BigDecimal!
  "volume in USD”
  volumeUSD: BigDecimal!
  "fees in USD”
  feesUSD: BigDecimal!
  "numebr of transactions during period”
  txCount: BigInt!
  "opening price of token0”
  open: BigDecimal!
  "high price of token0”
  high: BigDecimal!
  "low price of token0”
  low: BigDecimal!
  "close price of token0”
  close: BigDecimal!
}
```

### TickHourData entity

```graphql
{
  "format: <pool address>-<tick index>-<timestamp>”
  id: ID!
  "unix timestamp for start of hour”
  periodStartUnix: Int!
  "pointer to pool”
  pool: Pool!
  "pointer to tick”
  tick: Tick!
  "total liquidity pool has as tick lower or upper at end of period”
  liquidityGross: BigInt!
  "how much liquidity changes when tick crossed at end of period”
  liquidityNet: BigInt!
  "hourly volume of token0 with this tick in range”
  volumeToken0: BigDecimal!
  "hourly volume of token1 with this tick in range”
  volumeToken1: BigDecimal!
  "hourly volume in derived USD with this tick in range”
  volumeUSD: BigDecimal!
  "fees in USD”
  feesUSD: BigDecimal!
}
```

### TickDayData entity

```graphql
{
  "format: <pool address>-<tick index>-<timestamp>”
  id: ID!
  "timestamp rounded to current day by dividing by 86400”
  date: Int!
  "pointer to pool”
  pool: Pool!
  "pointer to tick”
  tick: Tick!
  "total liquidity pool has as tick lower or upper at end of period”
  liquidityGross: BigInt!
  "how much liquidity changes when tick crossed at end of period”
  liquidityNet: BigInt!
  "hourly volume of token0 with this tick in range”
  volumeToken0: BigDecimal!
  "hourly volume of token1 with this tick in range”
  volumeToken1: BigDecimal!
  "hourly volume in derived USD with this tick in range”
  volumeUSD: BigDecimal!
  "fees in USD”
  feesUSD: BigDecimal!
  "vars needed for fee computation”
  feeGrowthOutside0X128: BigInt!
  feeGrowthOutside1X128: BigInt!
}
```

### TokenDayData entity

```graphql
{
  "token address concatendated with date”
  id: ID!
  "timestamp rounded to current day by dividing by 86400”
  date: Int!
  "pointer to token”
  token: Token!
  "volume in token units”
  volume: BigDecimal!
  "volume in derived USD”
  volumeUSD: BigDecimal!
  "volume in USD even on pools with less reliable USD values”
  untrackedVolumeUSD: BigDecimal!
  "liquidity across all pools in token units”
  totalValueLocked: BigDecimal!
  "liquidity across all pools in derived USD”
  totalValueLockedUSD: BigDecimal!
  "price at end of period in USD”
  priceUSD: BigDecimal!
  "fees in USD”
  feesUSD: BigDecimal!
  "opening price USD”
  open: BigDecimal!
  "high price USD”
  high: BigDecimal!
  "low price USD”
  low: BigDecimal!
  "close price USD”
  close: BigDecimal!
}
```

### TokenHourData entity

```graphql
{
  "token address concatendated with date”
  id: ID!
  "unix timestamp for start of hour”
  periodStartUnix: Int!
  "pointer to token”
  token: Token!
  "volume in token units”
  volume: BigDecimal!
  "volume in derived USD”
  volumeUSD: BigDecimal!
  "volume in USD even on pools with less reliable USD values”
  untrackedVolumeUSD: BigDecimal!
  "liquidity across all pools in token units”
  totalValueLocked: BigDecimal!
  "liquidity across all pools in derived USD”
  totalValueLockedUSD: BigDecimal!
  "price at end of period in USD”
  priceUSD: BigDecimal!
  "fees in USD”
  feesUSD: BigDecimal!
  "opening price USD”
  open: BigDecimal!
  "high price USD”
  high: BigDecimal!
  "low price USD”
  low: BigDecimal!
  "close price USD”
  close: BigDecimal!
}
```

### FeeHourData entity

```graphql
{
  #
  id: ID!
  # pointer to pool
  pool: String!
  # sum of all fees for hour
  fee: BigInt!
  # count of fees changes for hour
  changesCount: BigInt!
  # time
  timestamp: BigInt!
  # min fee for hour
  minFee: BigInt!
  # max fee for hour
  maxFee: BigInt!
  # fee at the start timestamp of hour
  startFee: BigInt!
  # fee at the end of hour
  endFee: BigInt! 
}
```
