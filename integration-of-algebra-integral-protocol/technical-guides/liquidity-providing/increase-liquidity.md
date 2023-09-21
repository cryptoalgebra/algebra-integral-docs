---
ID: "4"
title: "Increase Liquidity"
---

## Increase Liquidity Within The Current Range

- Before getting into this section, take a closer look at the [first guide](./setting-up-your-contract.md) to make sure that all the previous steps are completed correctly.

- The example below implies that the contract already has custody of the NFT.

- Remember that we cannot change the boundaries of a given liquidity position using the Algebra protocol; the `increaseLiquidity` function can just increase the liquidity of a position.

- In production, `amount0Min` and `amount1Min` should be adjusted to create slippage protections.

```solidity
    /// @notice Increases liquidity in the current range
    /// @dev Pool must be initialized already to add liquidity
    /// @param tokenId The id of the erc721 token
    /// @param amount0 The amount to add of token0
    /// @param amount1 The amount to add of token1
    function increaseLiquidityCurrentRange(
        uint256 tokenId,
        uint256 amountAdd0,
        uint256 amountAdd1
    )
        external
        returns (
            uint128 liquidity,
            uint256 amount0,
            uint256 amount1
        )
    {
        INonfungiblePositionManager.IncreaseLiquidityParams memory params =
            INonfungiblePositionManager.IncreaseLiquidityParams({
                tokenId: tokenId,
                amount0Desired: amountAdd0,
                amount1Desired: amountAdd1,
                amount0Min: 0,
                amount1Min: 0,
                deadline: block.timestamp
            });

        (liquidity, amount0, amount1) = nonfungiblePositionManager.increaseLiquidity(params);
    }
```
