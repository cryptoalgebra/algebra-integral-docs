# Table of contents

* [Intro](README.md)

## Integration of Algebra Integral protocol

* [Specification and description of contracts](integration-of-algebra-integral-protocol/specification-and-description-of-contracts.md)
* [Contracts API](integration-of-algebra-integral-protocol/contracts-api/README.md)
  * [Algebra Pool](integration-of-algebra-integral-protocol/contracts-api/Core/AlgebraPool.md)
  * [Algebra Factory](integration-of-algebra-integral-protocol/contracts-api/Core/AlgebraFactory.md)
* [Interaction with pools](integration-of-algebra-integral-protocol/interaction-with-pools/README.md)
  * [Getting data from pools](integration-of-algebra-integral-protocol/interaction-with-pools/getting-data-from-pools.md)
* [Subgraphs and analytics](integration-of-algebra-integral-protocol/subgraphs-and-analytics/README.md)
  * [Examples of queries](integration-of-algebra-integral-protocol/subgraphs-and-analytics/examples-of-queries.md)
* [Technical guides](integration-of-algebra-integral-protocol/technical-guides/README.md)
  * [Intro](integration-of-algebra-integral-protocol/technical-guides/intro.md)
  * [Swaps](integration-of-algebra-integral-protocol/technical-guides/swaps/README.md)
    * [Single swaps](integration-of-algebra-integral-protocol/technical-guides/swaps/single-swaps.md)
    * [Multihop swaps](integration-of-algebra-integral-protocol/technical-guides/swaps/multihop-swaps.md)
  * [Providing liquidity](integration-of-algebra-integral-protocol/technical-guides/liquidity-providing/README.md)
    * [Setting up your contract](integration-of-algebra-integral-protocol/technical-guides/liquidity-providing/setting-up-your-contract.md)
    * [Mint a new position](integration-of-algebra-integral-protocol/technical-guides/liquidity-providing/mint-a-new-position.md)
    * [Collect fees](integration-of-algebra-integral-protocol/technical-guides/liquidity-providing/collect-fees.md)
    * [Decrease liquidity](integration-of-algebra-integral-protocol/technical-guides/liquidity-providing/decrease-liquidity.md)
    * [Increase liquidity](integration-of-algebra-integral-protocol/technical-guides/liquidity-providing/increase-liquidity.md)
    * [Final contract](integration-of-algebra-integral-protocol/technical-guides/liquidity-providing/the-full-contract.md)
  * [Flashloans](integration-of-algebra-integral-protocol/technical-guides/flashloans/README.md)
    * [Setting up your contract](integration-of-algebra-integral-protocol/technical-guides/flashloans/Inheritance-constructors.md)
    * [Calling flash](integration-of-algebra-integral-protocol/technical-guides/flashloans/calling-flash.md)
    * [Flash callback](integration-of-algebra-integral-protocol/technical-guides/flashloans/flash-callback.md)
    * [Fina contract](integration-of-algebra-integral-protocol/technical-guides/flashloans/final-contract.md)
* [Migration from UniswapV3](integration-of-algebra-integral-protocol/migration-from-uniswapv3.md)

## Core logic

* [Pool overview](core-logic/pool-overview.md)
* [Swap calculation](core-logic/swap-calculation.md)
* [Liquidity and positions](core-logic/liquidity-and-positions.md)
* [Ticks](core-logic/ticks/README.md)
  * [Ticks search tree](core-logic/ticks/ticks-search-tree.md)
* [Reserves](core-logic/reserves.md)
* [Flash](core-logic/flash.md)
* [Plugins](core-logic/plugins.md)
* [AlgebraFactory and roles](core-logic/algebrafactory-and-roles.md)

## Plugins

* [Intro](plugins/intro.md)

***

* [Changes after V1](changes-after-v1.md)
