# Migration from UniswapV3

Algebra Integral is an AMM that implements a concentrated liquidity mechanism. We have made great efforts to implement contracts in a way that maintains the greatest compatibility with UniswapV3 interfaces. For this reason, the rich set of tools and solutions developed for UniswapV3 can be easily ported and used with Algebra Integral.

The following will describe important differences from the UniswapV3 interface.

## Solidity 0.8.20

Algebra Integra is actively using new possibilities of Solidity. Therefore key contracts use version 0.8.20. At the same time, to increase compatibility, the interfaces allow the use of older versions of solidity.

<mark style="color:orange;">It is important to note that interface</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`IAlgebraPool`</mark> <mark style="color:orange;"></mark><mark style="color:orange;">requires a version of at least 0.8.4 due to the use of custom errors. For those who use older versions of the compiler, a special version of the interface without custom errors has been made:</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`IAlgebraPoolLegacy`</mark><mark style="color:orange;">.</mark>

## AlgebraFactory

The Algebra Integral pool factory has a number of important differences from the UniswapV3 pool factory.

* The direct deployer of pools is not a `AlgebraFactory` contract, but a separate `AlgebraPoolDeployer` contract. This is important for calculating pool addresses using `create2`.
* Factory exposes publically `POOL_INIT_CODE_HASH` which can simplify the external calculation of pool addresses using `create2`. In addition, this allows to verify the versions of the contracts used between different instances of the protocol.
* Factory exposes publically `computePoolAddress` function, which calculates the pool address for a given pair of tokens using `create2`.

In addition, the AlgebraFactory implements the `IAccessControl` interface and is an AccessManager for the entire protocol.

## One pool for each pair

Algebra Integral creates one pool for each pair of tokens. Because of this, determining the pool address and all interactions that require choosing a pool are simplified (no need to specify the commission level, as in Uniswap V3). Examples of differences due to this:

* No need to use fee value in `create2` style address calculation
* No need to pass fee value in `SwapRouter` or `NonfungiblePositionManager`
* No need to pass fee in `createPool` function of AlgebraFactory

## AlgebraPool

The Algebra Integral liquidity pool is based on the same math and the same formulas as the UniswapV3 pool. At the same time, the internal logic differs in many details, which leads to differences in the external interfaces.

### Plugins

Algebra Integral makes it possible to expand the functionality of the pool using additional plugins. For this reason, the behavior of different protocol instances may have a different set of additional functionality.

### Main state struct: `globalState` instead of `slot0`

The main values of the pool state are stored in a structure called `globalState`:

```solidity
struct GlobalState {
  uint160 price; // The square root of the current price in Q64.96 format
  int24 tick; // The current tick
  uint16 fee; // The current fee in hundredths of a bip, i.e. 1e-6
  uint8 pluginConfig; // The current plugin config as a bitmap
  uint16 communityFee; // The community fee represented as a percent of all collected fee in thousandths (1e-3)
  bool unlocked; // True if the contract is unlocked, otherwise - false
}

GlobalState public override globalState;
```

### Mint

The mint function is different: Algebra Integral can handle cases where, as a result of a callback, the pool receives fewer tokens than expected. This makes it possible to use specific tokens, such as fee-on-transfer tokens.

For this reason, the mint function in Algebra Integral has one additional argument `address leftoversRecipient`: the address to which excess tokens should be returned. In addition, the function also returns one additional value `uint128 liquidityActual`:  the final liquidity value that was added.

### Burn

The burn function in Algebra Integral has one additional argument `bytes calldata data`: this allows to pass additional arbitrary data to the plugin if needed.

### Special version of `swap` function with payment in advance

Algebra Integral pools contain two versions of the swap function. First is standard `swap`, which corresponds to a similar function in UniswapV3. Second is special version named `swapWithPaymentInAdvance`.

This special version requires to send tokens to the pool before the swap internal calculation, which makes it possible, for example, to swap fee-on-transfer tokens.

### Flash

Algebra Integral allows to take flashloans even if the current liquidity is zero, if there are tokens in the pool. UniswapV3 prohibits taking a flashloan in such a situation.

Algebra Integral uses a fixed fee for a flashloan equal to 0.01%

### TWAP in a separate optional plugin

The Algebra Integral pool by default does not contain an oracle and does not waste gas on writing data, unlike UniswapV3. For this reason, the pools are missing anything related to the oracle. If you need to receive data from such an oracle, you must use the corresponding plugin if it is connected to the pool.

### Changeable tickspacing

Unlike UniswapV3, tickspacing in Algebra Integral is not an immutable value: it can change. At the same time, the change of tickspacing plays a role only for new liquidity positions; the previous ones will continue to function.

### Changeable fee

Unlike UniswapV3, fуу in Algebera Integral pools is not an immutable value: it can change. The fee value can be changed by the governance, or, if there is an appropriate plugin connected, it can be subject to the rules of the dynamic fee.

Pool exposes `fee()` getter function which should return the current fee value at the moment. If a plugin with dynamic fee is used, this method will request the current value from that plugin.

### tickTable instead of tickBitmap

Algebra Integral uses a doubly linked list to navigate through active ticks. At the same time, to simplify the insertion of new ticks, there is a special search tree. The leaves of this search are organized in a structure similar to the `tickBitmap` array in UniswapV3 and it's called `tickTable`.

Unlike tickBitmap, tickTable does not take into account tickspacing - when defining the corresponding word number, tick indexes without “compression” must be used.

In addition, Algebra Integral pool exposes `prevTickGlobal()` and `nextTickGlobal()`, which make it possible to find out the previous and next active tick without using a bitmap.



## Periphery

The periphery almost completely corresponds to the periphery of UniswapV3, with the exception of some of the features mentioned above. Including:

* Use of Solidity 0.8.20 in contracts (>=0.7.6 in interfaces)
* No need to pass fee level anywhere
* Quoter additionaly returns the fee value that will be used during the swap
* Additional function `exactInputSingleSupportingFeeOnTransferTokens` in SwapRouter contract



