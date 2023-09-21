---
ID: "0"
title: "Inheritance constructor"
---

## Flash Transactions Overview

Flash transactions are an approach to token transfers on Polygon in which token residues are transferred before the necessary conditions for transferring those residues have been met. In the context of a swap, this means that the output is sent from the swap before the input is received.

`Flash` is aimed at withdrawing a specified amount of both `token0` and `token1` to the recipient address. The withdrawn amount, plus the swap fees, will be due to the pool at the end of the transaction. `Flash` includes a fourth parameter, `data`, which allows the caller to abi.encode any necessary data to be passed through the function and decoded later.


```solidity
    function flash(
        address recipient,
        uint256 amount0,
        uint256 amount1,
        bytes calldata data
    ) external override lock noDelegateCall {
```

## The Flash Callback

Now that we already know that `flash` is aimed to withdraw the tokens, we still have a question: how can they be paid back? To find the answer, we need to learn the flash function code in detail. Midway through the flash function, we see this:

```solidity
IAlgebraFlashCallback(msg.sender).algebraFlashCallback(fee0, fee1, data);
```

This step calls the `FlashCallback` function on `msg.sender` - which passes the fee data needed to calculate the balances due to the pool, as well as any data encoded into the `data` parameter.

We can highlight three different, separate callback functions in Algebra. These are `algebraSwapCallback`, `algebraMintCallback`, and `algebraFlashCallback`, and each of them can be easily overridden by custom logic. To write our arbitrage contract, it is required to call `flash`and then override the `algebraFlashCallback` with the steps necessary to complete our transaction.

## Inheriting The Algebra Contracts

Inherit `IAlgebraFlashCallback` and `PeripheryPayments`, as we will use each of the above in our program. Note that these two inherited contracts already extend many of the other contracts we will be using in the future,such as LowGasSafeMath which we [attach](https://docs.soliditylang.org/en/v0.7.6/contracts.html?highlight=using#using-for), to types `uint256` and `int256`.

```solidity
contract PairFlash is IAlgebraFlashCallback, PeripheryPayments {
    using LowGasSafeMath for uint256;
    using LowGasSafeMath for int256;
```

Declare an immutable public variable `swapRouter` of type `ISwapRouter`:

```solidity
    ISwapRouter public Immutable swapRouter;
```

Declare the constructor here, which is executed once when the contract is deployed. Our constructor hardcodes the address of the Algebra router, factory, and the address of wmatic, the ERC-20 wrapper for matic.

```solidity
    constructor(
        ISwapRouter _swapRouter,
        address _factory,
        address _WMATIC9
    ) PeripheryImmutableState(_factory, _WMATIC9) {
        swapRouter = _swapRouter;
    }
```

The full import section and contract declaration:

```solidity
pragma solidity =0.7.6;
pragma abicoder v2;

import '@cryptoalgebra/core/contracts/interfaces/callback/IAlgebraFlashCallback.sol';
import '@cryptoalgebra/core/contracts/libraries/LowGasSafeMath.sol';

import '@cryptoalgebra/periphery/contracts/base/PeripheryPayments.sol';
import '@cryptoalgebra/periphery/contracts/base/PeripheryImmutableState.sol';
import '@cryptoalgebra/periphery/contracts/libraries/PoolAddress.sol';
import '@cryptoalgebra/periphery/contracts/libraries/CallbackValidation.sol';
import '@cryptoalgebra/periphery/contracts/libraries/TransferHelper.sol';
import '@cryptoalgebra/periphery/contracts/interfaces/ISwapRouter.sol';



contract PairFlash is IAlgebraFlashCallback, PeripheryPayments {
    using LowGasSafeMath for uint256;
    using LowGasSafeMath for int256;

    ISwapRouter public immutable swapRouter;

    constructor(
        ISwapRouter _swapRouter,
        address _factory,
        address _WMATIC9
    ) PeripheryImmutableState(_factory, _WMATIC9) {
        swapRouter = _swapRouter;
    }
```
