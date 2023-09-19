---
ID: "0"
title: "Calling Flash"
---

## Parameter Structs

To call `flash`, the flash parameters for the initial call are required, as well as any other parameters you’d like to pass through to the callback.

The `FlashParams` struct will contain the addresses of the tokens and the amounts we want to extract from the pool.

```solidity
  struct FlashParams {
        address token0;
        address token1;
        uint256 amount0;
        uint256 amount1;
    }
```

The `FlashCallbackData` struct must contain the data we want to send to the callback. This includes `poolKey`, which expresses the sorted tokens returned by the PoolAddress library.

```solidity
    struct FlashCallbackData {
        uint256 amount0;
        uint256 amount1;
        address payer;
        PoolAddress.PoolKey poolKey;
    }
```

## Pool Key

From this point, we are starting our function by assigning the appropriate parameters from the `Flashparams` (which we have declared in memory as `params`) to our variable `poolKey`

```solidity
    function initFlash(FlashParams memory params) external {
        PoolAddress.PoolKey memory poolKey =
            PoolAddress.PoolKey({token0: params.token0, token1: params.token1);
    }
```

The next step is to declare `pool` as type [IAlgebraPool], which allows us to call `flash` on the desired pool contract.

```solidity
        IAlgebraPool pool = IAlgebraPool(PoolAddress.computeAddress(factory, poolKey));
```

## Calling Flash

Finally, we call `flash` on the previously declared `pool`. In the last parameter, we abi.encode the `FlashCallbackData`, which will be decoded in the callback and aimed at informing about the next steps of the particular transaction.

```solidity
        pool.flash(
            address(this),
            params.amount0,
            params.amount1,
            abi.encode(
                FlashCallbackData({
                    amount0: params.amount0,
                    amount1: params.amount1,
                    payer: msg.sender,
                    poolKey: poolKey
                })
            )
        );
    }
```

The full function:

```solidity

    struct FlashParams {
        address token0;
        address token1;
        uint256 amount0;
        uint256 amount1;
    }

    struct FlashCallbackData {
        uint256 amount0;
        uint256 amount1;
        address payer;
        PoolAddress.PoolKey poolKey;
    }

function initFlash(FlashParams memory params) external {
        PoolAddress.PoolKey memory poolKey =
            PoolAddress.PoolKey({token0: params.token0, token1: params.token1});
        IAlgebraPool pool = IAlgebraPool(PoolAddress.computeAddress(factory, poolKey));
        pool.flash(
            address(this),
            params.amount0,
            params.amount1,
            abi.encode(
                FlashCallbackData({
                    amount0: params.amount0,
                    amount1: params.amount1,
                    payer: msg.sender,
                    poolKey: poolKey
                })
            )
        );
    }
```
