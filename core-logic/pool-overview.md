# Pool overview

### One pool for each pair

The liquidity pool is a key component of the Algebra Integral protocol. This smart contract implements the most important functions that provide swaps, liquidity management, flashloans and other protocol functionality.&#x20;

For each pair of tokens in the Algebra protocol, one unique pool is created. This approach minimises liquidity fragmentation, simplifies the construction of optimal routes for swaps, and simplifies liquidity management for liquidity providers.

<div data-full-width="false">

<figure><img src="../.gitbook/assets/Algebra factory.drawio (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

The address of each pool is determined deterministically by the `create2` mechanism using token addresses as salt.

Liquidity pools are not intended for direct use by ordinary users - for convenient use of the protocol functionality, users use peripheral contracts that implement additional calculations and security checks.

### Requirements for tokens

The liquidity pool expects the token to be ERC20 standard. In addition, there are the following limitations:

1. The Algebra Integral protocol does not support tokens that can arbitrarily reduce balances at addresses.
2. The Algebra Integral protocol does not guarantee full functionality when using tokens whose total supply exceeds $$2^{128} -1$$(maximum value uint128).
3. The Algebra Integral protocol does not guarantee full performance with tokens that have the ability to selfdestruct or upgradeable code.

### Pool Description

The main purpose of the pool is to enable swaps using concentrated liquidity.&#x20;

Below is a high-level description of the main functionality implemented in the pool.

<mark style="color:red;">It is important to note that a significant part of interactions with the pool uses the callback mechanism. When creating your own implementation of the periphery and interacting with pools directly, you should make sure that the contract causing the callback is actually the expected Algebra Integral pool.</mark>

#### Token Swap

The main task of AMM is to provide token swap. Swapping in Algebra Integral is performed using concentrated liquidity technology. More detailed description of the swap process in [the article about swap calculations](swap-calculation.md).

In Algebra Integral there are two variants of swap - with prepayment and with payment after settlement.

**Swap with payment after computation (`swap`)** is the main variant that should be used for most token pairs. First, the calculations required for swap are performed on the specified parameters, then the user receives the output tokens, after which he must pay for the input tokens inside the callback (`algebraSwapCallback`).

**`SwapWithPaymentInAdvance`** is an additional variant of swap that can be useful for some classes of tokens (e.g., tokens that charge a transfer fee). First, the user must send the required number of input tokens in a callback (algebraSwapCallback), then calculations are performed, then the user receives the output tokens.

#### Liquidity positions

Users can provide tokens as liquidity at some given price range. A liquidity position, as long as it is active, receives a portion of the commissions that are collected on each swap. See [the article on liquidity and liquidity positions](liquidity-and-positions.md) for a more detailed description of the logic and calculations.

Additionally, a liquidity position can receive a part of commissions from flashloans and tokens sent directly to the token pool (using [the reserve mechanism](reserves.md)).

Basic actions with liquidity positions:

* `mint` - increase liquidity in the position by providing more tokens. If the corresponding position does not exist, it is created. Whoever calls the method must provide tokens to the pool during the callback (`algebraMintCallback`). If the pool receives fewer tokens than expected, the added liquidity will be recalculated accordingly.\
  When adding liquidity, the tickspacing restriction is taken into account - you cannot add liquidity to a position if the indices of the corresponding ticks are not divided by `tickSpacing`.
* `burn` - reduction of liquidity in the position. As a result of liquidity reduction, tokens to be received by the user are written to the liquidity position as a fee (added to the commission) and can be withdrawn using the `collect` method.
* `collect` - the user receives the tokens that are recorded as a fee in the corresponding liquidity position. This method **does not update** the values of the fees collected by the position, so it is recommended to call `burn` with zero liquidity change (burn 0 liquidity and recalculate the fees) before it.

#### **Flashloans**

The pool provides the possibility of flashloans - the user can request any number of tokens from the pool (not exceeding the pool balance) and must return them with a commission in the subsequent callback (`algebraFlashCallback`), otherwise the transaction will be reverted.

The flashloan fee will be distributed among active liquidity positions. In addition, any excess tokens that were transferred in the callback above the required amount will also be distributed to active liquidity providers as a fee. If there is currently no active liquidity, the collected commission will go to the first liquidity positions that become active in the future (through the use of [the reserve mechanism](reserves.md)).

It is important to note that not only active liquidity is available for receiving a flashloan, but the entire token balance available to the pool.

#### Pool Customization

Each Algebra Integral pool has a number of parameters that can be changed by the protocol team, DAO or authorized persons. Persons authorized to change pool parameters (see [the AlgebraFactory and roles article](algebrafactory-and-roles.md) for more information on roles and access control):

1. AlgebraFactory owner&#x20;
2. owner of the `POOLS_ADMINISTRATOR` role

Customizable parameters:

* `communityFee` - share of the collected commissions that is sent to AlgebraCommunityVault for the needs of the protocol.
* `tickSpacing` - limit on ticks that can be used as boundaries of the liquidity position. New liquidity can only be added to positions whose upper and lower ticks are divided wholly by `tickSpacing`. This means that if `tickSpacing` is equal to 60, then only every 60th tick (... -60, 0, 60 ...) can be used as a new liquidity position boundary.
* `plugin` - it is possible to replace the plugin that is connected to the pool, connect a new one or disable the plugin altogether.
* `pluginConfig` - customize plugin configuration, which defines how the pool should interact with the plugin. More details in [the article about plugins](plugins.md).
* `fee` - if dynamic commission mode is not enabled (see [the article about plugins](plugins.md)), the commission for swaps in the pool can be changed manually.
