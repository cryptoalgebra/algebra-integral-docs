# Changes after V1

The new version of our protocol, **Agebra Integral**,  contains a lot of new functions and capabilities, as well as various kinds of optimizations. This version is significantly different from Algebra V1. To simplify migration and integration of the new version, we briefly describe the major differences in this document.

Currently Algebra Integral is located in the following branch of the repository:

[Algebra Integral dev branch](https://github.com/cryptoalgebra/Algebra/tree/dev)

At the same time, the V1 was moved to a separate repository (legacy):

[AlgebraV1.9 (legacy)](https://github.com/cryptoalgebra/AlgebraV1.9)

[AlgebraV1 (legacy)](https://github.com/cryptoalgebra/AlgebraV1)

#### Solidity 0.8.20

The most basic change that affected almost all contracts is the transition to a new version of Solidity (0.8.20). In order to maintain backward compatibility, the version directive of interfaces has been kept as >=0.5.0 where possible.

<mark style="color:orange;">It is important to note that interface</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`IAlgebraPool`</mark> <mark style="color:orange;"></mark><mark style="color:orange;">requires a version of at least 0.8.4 due to the use of custom errors. For those who use older versions of the compiler, a special version of the interface without custom errors has been made:</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`IAlgebraPoolLegacy`</mark><mark style="color:orange;">.</mark>

#### Gas optimizations

Almost all contracts have been significantly reworked, which has resulted in gas savings while expanding the capabilities of the protocol.

### AlgebraPool changes

#### Supported tokens

The new version of the protocol **does not** support tokens whose total supply can exceed "2^128 - 1" (max value of `uint128`). This limitation allows us to make contracts more gas-efficient and predictable for the vast majority of tokens.

The new version **has wider support for rebase tokens**: in the event of a rebase and an increase in the pool balance, the resulting excess will be distributed among active liquidity providers. In the first version of Algebra, as in Uniswap V3 and other implementations of concentrated liquidity, this excess of tokens would simply be stuck on the pool contract forever.

#### Usage of plugins

The functionality of each Algebra Integral pool can be expanded through various **plugins**. For more details, see the corresponding section of the documentation.

#### TWAP oracle moved to plugins

The oracle has been removed from the core protocol and moved to an optional plugin. This avoids spending gas on the oracle if it is not needed.

Now, to get data about timepoints, you need to access the corresponding plugin contract directly.

#### Adaptive fee moved to plugins

Our dynamic commission was separated from the pools into a separate plugin. Such a change allows us to update the logic of the dynamic fee, use other approaches, or completely turn off the dynamic fee, using the static one and reducing gas costs. Thanks to this, Algebra Integral can be more flexibly customized to specific needs.

#### GlobalState structure

Due to other pool changes, the `globalState` structure has changed. Added field `pluginConfig` - tightly packed bitmap with plugin cofiguration. Field `timepointIndex` was removed together with the TWAP oracle. Fields `communityFeeToken0` and `communityFeeToken1` replaced with single field `communityFee`.

#### Changeable tickspacing

In the first version of the protocol, tickspacing was an immutable constant within pools. In Algebra Integral, tickspacing can be changed at any time. This allows the protocol to more accurately and efficiently set up pools for different kinds of tokens. The default tickspacing is 60. Contracts interacting with Algebra Integral pools must take into account the possibility that tickspacing will change. MAX\_LIQUIDITY\_PER\_TICK is fixed based on tickspacing value 1.

Ticks are stored in a bitmap without transformations associated with tickspacing: _internally_, the pool always uses tickspacing equal to one. This does not introduce any additional overhead, since a doubly linked list is now used to navigate ticks.

#### Data in ticks

Removed fields `outerTickCumulative`, `outerSecondsSpent` and `outerSecondsPerLiquidity` from `Tick` structure. Added fields required for the implementation of a doubly linked list. See the contract documentation for details.

#### Tick data structure

Now ticks are stored as a doubly linked list: each tick contains the indices of the next (`nextTick`) and previous (`prevTick`) ticks. The `tickTable` mapping is saved and used in the search tree when inserting and deleting new ticks. The `tickTable` mapping structure is similar to the same mapping in the first version of the protocol.

#### Position structure

Removed the `lastLiquidityAddTimestamp` field due to the removal of the entire liquidity cooldown mechanism. Changed the data type of the `liquidity` field to `uint256`.

#### Custom errors

The second version of the protocol uses custom errors added to Solidity instead of of text messages in `require`. This allows us to increase the readability and informativeness of errors, while reducing the size of the bytecode. Pool errors are placed in the `IAlgebraPoolErrors` interface.

<mark style="color:orange;">It is important to note that interface</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`IAlgebraPool`</mark> <mark style="color:orange;"></mark><mark style="color:orange;">requires a version of at least 0.8.4 due to the use of custom errors. For those who use older versions of the compiler, a special version of the interface without custom errors has been made:</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`IAlgebraPoolLegacy`</mark><mark style="color:orange;">.</mark>

#### Community fee setup and transfer

1. Now community fee is regulated by one parameter (`communityFee`) instead of two.
2. Governance can also now configure what value of `communityFee` will be set in new pools by default. This makes it easier to set up new pools.
3. In order to save gas, community fee is no longer sent with each swap, but is accumulated and sent at a specified frequency.
4. Now all community fee is sent to a pre-specified contract `AlgebraCommunityVault`. This greatly reduces gas costs and increases the security of the protocol.

#### Liquidity cooldown removed

The new version of the protocol removed the liquidity cooldown mechanic, which was intended to protect against JIT-liquidity in the first version.&#x20;

### Farming changes

#### Farming as optional plugin

In Algebra Integral, the standard plugin provided by the protocol is now optional and can be connected using the plugin. This allows to use different farming formats, connect new ones, or completely disable this functionality.

#### Limit farmings removed

In the new version of the protocol, one type of farming is left: eternal (V2-like) farming. Limit farms were almost never used and their removal allows to significantly simplify contracts and increase security.

#### Transfer of position NFT is no longer needed

In the new version, users no longer need to send the NFT of their position to the farming address. In addition, users can increase and decrease liquidity without having to exit and re-enter farming.

### Roles and AccessControl

Now the AlgebraFactory contract implements the IAccessControl interface and the role system. This allows the protocol to fine-tune the permissions to perform individual actions in the protocol.

The `owner` of the AlgebraFactory is a superuser and has the rights to all actions in the standard contracts of the Algebra Integral protocol. We recommend transferring the owner's rights to a multisig or a special contract to reduce risks.

Protocol can assign the following roles to different addresses:

* `keccak256('POOLS_ADMINISTRATOR')` - allows to change tickspacing, community fee, plugin and plugin configuration in pools
* `keccak256('COMMUNITY_FEE_WITHDRAWER')` - allows to withdraw accumulated community fees from the `AlgebraCommunityVault` contract
* `keccak256('COMMUNITY_FEE_VAULT_ADMINISTRATOR')` - allows change configuration of the `AlgebraCommunityVault` contract
* `keccak256('NONFUNGIBLE_POSITION_MANAGER_ADMINISTRATOR_ROLE')` - allows to change FarmingCenter address in NonfungiblePositionManager contract
* `keccak256('ALGEBRA_BASE_PLUGIN_MANAGER')` - allows to change adaptive fee configuration in corresponding plugin&#x20;
* `keccak256('INCENTIVE_MAKER_ROLE')` - allows to create new farmings, change rates and deactivate old farmings in AlgebraEternalFarming contract
* `keccak256('FARMINGS_ADMINISTRATOR_ROLE')` - allows to change FarmingCenter address in AlgebraEternalFarming contract and withdraw unspent rewards from farmings

In addition, the protocol can create new roles and use them in its other contracts.

### CommunityVault

Accumulated community fee is now sent to a pre-specified contract AlgebraCommunityVault. This greatly reduces gas costs and increases the security of the protocol. An address with the appropriate rights (AlgebraFactory owner or `keccak256('COMMUNITY_FEE_WITHDRAWER')`) can withdraw tokens from this vault.

### Adaptive fee plugin

As already mentioned, the standard version of the dynamic commission of the Algebra protocol has been moved to a separate plugin.

The adaptive fee configuration has been simplified. Params `volumeBeta` and `volumeGamma` have been removed along with the corresponding sigmoid.

The corresponding contract code has been largely redesigned. Thanks to this, it was possible to drastically reduce gas consumption and improve the accuracy of the adaptive commission.
