# Getting data from pools

## How to determine which token is token0 and which is token1?

The order of the tokens is determined in such a way that:

&#x20;`address(token0) < address(token1)`

## Price

### How to get current price in pool?

Information about the current instant price of the tokens in pool is in the `globalState` structure:

```solidity
// AlgebraPool contract

struct GlobalState {
  uint160 price; // The square root of the current price in Q64.96 format
  int24 tick; // The current tick
  uint16 fee; // The current fee in hundredths of a bip, i.e. 1e-6
  uint8 pluginConfig; // The current plugin config as a bitmap
  uint16 communityFee; // The community fee represented as a percent of all collected fee in thousandths (1e-3)
  bool unlocked; // True if the contract is unlocked, otherwise - false
}

/// @inheritdoc IAlgebraPoolState
GlobalState public override globalState;
```

So you can get price in this way:

```solidity
(uint160 price, , , , , ) = IAlgebraPool(poolAddress).globalState();
```

Next, it is important to understand what exactly is meant by price in liquidity pools. The pool stores the **square root** of the price of token0 relative to token1 in format [`Q64.96`](https://en.wikipedia.org/wiki/Q\_\(number\_format\)). So, in pseudocode, price value in pool can be expressed as:

```python
price_pool = sqrt(reserve_1_virtual / reserve_0_virtual) * 2**96
```

To get the usual instant price from the price value in the pool, you need to make the following transformations:

```python
price_instant = (price_pool / 2**96)**2
```

Further, if you need to bring the price to a human-readable form, it is necessary to take into account the difference in token decimals, if any.

### Can i use "price" value in pool as real price?

Shortly: <mark style="color:red;">**no**</mark>, you should not rely on this raw value for anything other than internal pool calculations. This value in the pool reflects only its internal state at the current moment in time, can change and is subject to manipulation using swaps. In addition, when swapping, the actual execution price will differ from this value due to price impact and fee charged.&#x20;

To get the real price of the swap, see the paragraph [**How to get actual execution price for swap?**](getting-data-from-pools.md#how-to-get-actual-execution-price-for-swap)

If you need to get the average price in a pool for a certain period of time, then for this you can use data from a plugin with a TWAP-oracle, **if it is connected to the pool**.

### How to get actual execution price for swap?

To do this, you can use a special peripheral contract called Quoter. For example, here is description of `quoteExactInputSingle`:

```solidity
/// @inheritdoc IQuoter
function quoteExactInputSingle(
  address tokenIn,
  address tokenOut,
  uint256 amountIn,
  uint160 limitSqrtPrice
) public override returns (uint256 amountOut, uint16 fee) {
...
}
```

You can do a **static call** to this method _(and it is better not to use it on-chain due to gas costs)_. As a result, the number of tokens at the output and the real pool fee will be obtained.
