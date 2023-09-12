# Examples of queries

This doc will teach you how to query Algebra analytics by writing GraphQL queries on the subgraph. You can fetch data points like :

* [collected fees for a position](examples-of-queries.md#general-position-data)
* [current liquidity of a pool](examples-of-queries.md#pool-data)
* [volume on a certain day](examples-of-queries.md#historical-global-data)

and much more. Below are some example queries. To run a query copy and paste it into the [algebra explorer](https://graph.testnet.mantle.xyz/subgraphs/name/cryptoalgebra/swapsicle-analytics-v1) to get fresh data.

## Explore

All queries will be presented using our partner Swapsicle on mantle testnet as an example

* Graphql Endpoint: https://graph.testnet.mantle.xyz/subgraphs/name/cryptoalgebra/swapsicle-analytics-v1

## Global Data

Global data refers to data points about the Algebra protocol as a whole. Some examples of global data points are total value locked in the protocol, total pools deployed, or total transaction counts. Thus, to query global data you must pass in the Algebra Factory address `0x2209c0bccBDd1f750750023ba2bAcdC82D031EbE` and select the desired fields. Reference the full [factory schema](./#factory-entity) to see all possible fields.

### Current Global Data

An example querying total pool count, transaction count, and total volume in USD and native token:

```graphql
{
  factory(id: "0x2209c0bccBDd1f750750023ba2bAcdC82D031EbE" ) {
    poolCount
    txCount
    totalVolumeUSD
    totalVolumeMatic
  }
}
```

### Historical Global Data

You can also query historical data by specifying a block number.

```graphql
{
  factory(id: "0x2209c0bccBDd1f750750023ba2bAcdC82D031EbE", block: {number: 9682441}){
    poolCount
    txCount
    totalVolumeUSD
    totalVolumeMatic
  }
}
```

TheGraph cannot track entity changes by timestamp, only by block, so to convert timestamp to block number blocklytics subgraph is used:

* Graphql Endpoint: https://graph.testnet.mantle.xyz/subgraphs/name/cryptoalgebra/mantle-blocks

```graphql
{
  blocks(where:{timestamp: 1694529744}) {
    id
    number
    timestamp
    parentHash
  }
}
```

## Pool Data

To get data about a certain pool, pass in the pool address. Reference the full [pool schema](./#pool-entity) and adjust the query fields to retrieve the data points you want.

### General Pool Query

The query below returns the fee, spot price, and liquidity for the WMNT-USDC pool.

```graphql
  pool(id: "0xd5cefe146781d42f2d74a9c26af83bd6c9fe0d74") {
    tick
    token0 {
      symbol
      id
      decimals
    }
    token1 {
      symbol
      id
      decimals
    }
    fee
    sqrtPrice
    liquidity
  }
}
```

<mark style="color:orange;">⚠ All addresses must be in lower case</mark>

### All Possible Pools

The maxiumum items you can query at once is 1000. Thus to get all possible pools, you can interate using the skip variable. To get pools beyond the first 1000 you can also set the skip as shown below.

### Skipping First 1000 Pools

This query sets the skip value and returns the first 10 responses after the first 1000.

```graphql
{
  pools(first:10, skip:1000){
    id
    token0 {
      id
      symbol
    }
    token1 {
      id
      symbol
    }
  }
}
```

### Creating a Skip Variable

This next query sets a skip variable. In your language and environment of choice you can then iterate through a loop, query to get 1000 pools each time, and continually adjust skip by 1000 until all pool responses are returned.

Note: This query will not work in the graph explorer and more resembles the structure of a query you'd pass to some graphql middleware like Apollo.

```graphql
{
  query pools( $skip: Int!) {
    pools(
      first: 1000
      skip: $skip
      orderDirection: asc
    ) {
      id
      sqrtPrice
      token0 {
        id
      }
    	token1{
      id
    }
    }
}

```

### Most Liquid Pools

Retrieve the top 1000 most liquid pools. You can use this similar set up to orderBy other variables like number of swaps or volume.

```graphql
{
 pools(first: 1000, orderBy: totalValueLockedUSD, orderDirection: desc) {
   id
 }
}
```

### Pool Daily Aggregated

This query returns daily aggregated data for the first 10 days since the given timestamp for the WETH-USDC pool.

```graphql
{
  poolDayDatas(first: 10, orderBy: date, where: {
    pool: "0xd5cefe146781d42f2d74a9c26af83bd6c9fe0d74",
    date_gt: 1694030070 
  } ) {
    date
    liquidity
    sqrtPrice
    token0Price
    token1Price
    volumeToken0
    volumeToken1
  }
}
```

## Swap Data

### General Swap Data

To query data about a particular swap, input the transaction hash + "#" + the index in the swaps the transaction array. This is the reference for the full [swap schema](./#swap-entity).

This query fetches data about the sender, receiver, amounts, active liquidity of pool after swap, transaction data, and timestamp for a particular swap.

```graphql
{
   swap(id: "0x060570bc6627127d2b7668b00fc1875c058c95829402611212890f6708af258d#5") {
    sender
    recipient
    amount0
    amount1
    liquidity
    transaction {
      id
      blockNumber
      gasLimit
      gasPrice
    }
    timestamp
    token0 {
      id
      symbol
    }
    token1 {
      id
      symbol
    }
   }
 }
```

### Recent Swaps Within a Pool

You can set the `where` field to filter swap data by pool address. This example fetches data about multiple swaps for the WETH-USDC pool, ordered by timestamp.

```graphql
{
swaps(orderBy: timestamp, orderDirection: desc, where:
 { pool: "0xd5cefe146781d42f2d74a9c26af83bd6c9fe0d74" }
) {
  pool {
    token0 {
      id
      symbol
    }
    token1 {
      id
      symbol
    }
  }
  sender
  recipient
  amount0
  amount1
 }
}
```

## Token Data

Input the token contract address to fetch token data. Any token that exists in at least one Algebra pool can be queried. The output will aggregate data across all pools that include the token.

### General Token Data

This queries the decimals, symbol, name, pool count, and volume for the USDC token. Reference the full [token schema](./#token-entity) for all possible fields you can query.

```graphql
{
  token(id:"0x893388ba29248261a0f13371bd4ae3700ce06ec9") {
    symbol
    name
    decimals
    volumeUSD
    poolCount
  }
}
```

### Token Daily Aggregated

You can fetch aggregate data about a specific token over a 24-hour period. This query gets 10-days of the 24-hour volume data for the ALGB token ordered from oldest to newest.

```graphql
{
  tokenDayDatas(first: 10, where: {token: "0x893388ba29248261a0f13371bd4ae3700ce06ec9"}, orderBy: date, orderDirection: asc) {
    date
    token {
      id
      symbol
    }
    volumeUSD
  }
}
```

### All Tokens

Similar to retrieving all pools, you can fetch all tokens by using skip. Note: This query will not work in the graph sandbox and more resembles the structure of a query you'd pass to some graphql middleware like Apollo.

```graphql
query tokens($skip: Int!) {
  tokens(first: 1000, skip: $skip) {
    id
    symbol
    name
  }
}
```

## Position Data

### General Position Data

To get data about a specific position, input the NFT tokenId. This queries the collected fees for token0 and token1 and current liquidity for the position with tokedId 3. Reference the full [position schema](./#position-entity) to see all fields.

```graphql
{
  position(id:3) {
    id
    collectedFeesToken0
    collectedFeesToken1
    liquidity
    token0 {
      id
      symbol
    }
    token1
    {
      id
      symbol
    }
  }
}
```

### Position pool data

Since positions can be created not only via NonfungiblePositionManager, a poolPositions entity has been added which contains the basic information about the pool position.

```graphql
{
  poolPositions(where:{pool:"0xd5cefe146781d42f2d74a9c26af83bd6c9fe0d74"}){
    id
    liquidity
    lowerTick {
      tickIdx
    }
    upperTick {
      tickIdx
    }
    liquidity
  }
}
```
