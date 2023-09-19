# Single swaps

Swaps are the most common interaction with the Algebra Protocol. The following example displays the way of implementing a single-path swap contract that uses two functions that you create:

- `swapExactInputSingle`
- `swapExactOutputSingle`

The `swapExactOutputSingle`  function is used for performing exact output swaps, which swap a minimum possible amount of the token 1 for a fixed amount of the token 2. This function uses the `ExactOutputSingleParams` struct and the `exactOutputSingle` function from the ISwapRouter interface.

[ISwapRouter](https://docs.algebra.finance/en/docs/contracts/API-reference/periphery/ISwapRouter) 

The `swapExactOutputSingle` function is for performing _exact output_ swaps, which swap a minimum possible amount of one token for a fixed amount of another token. This function uses the `ExactOutputSingleParams` struct and the `exactOutputSingle` function from the ISwapRouter interface.

[ISwapRouter](https://docs.algebra.finance/en/docs/contracts/API-reference/periphery/ISwapRouter)

To make it seem more simple, the example hardcodes the token contract addresses, but as explained below the contract could be modified to change pools and tokens on a per-transaction basis.

When trading from a smart contract, the most important thing that you should remember is that access to an external price source is always required. Without that, trades can be frontrun for considerable loss.

**Note:** As always, the swap examples are not production ready code, and are implemented in a simplistic manner for the learning purposes.

## Set Up the Contract

Declare the solidity version used to compile the contract, and `abicoder v2` to allow arbitrary nested arrays and structs to be both encoded and decoded in calldata, a feature that is used when executing a swap.

```solidity
// SPDX-License-Identifier: GPL-2.0-or-later
pragma solidity =0.7.6;
pragma abicoder v2;
```

Import the two relevant contracts from the npm package installation

```solidity
import '@cryptoalgebra/periphery/contracts/interfaces/ISwapRouter.sol';
import '@cryptoalgebra/periphery/contracts/libraries/TransferHelper.sol';
```

Create a contract called `SwapExamples`, and declare an immutable public variable `swapRouter` of type `ISwapRouter`. This allows us to call functions in the `ISwapRouter` interface.

```solidity
contract SwapExamples {
    // For the scope of these swap examples,
    // we will detail the design considerations when using `exactInput`, `exactInputSingle`, `exactOutput`, and  `exactOutputSingle`.
    // It should be noted that for the sake of these examples we pass in the swap router as a constructor argument instead of inheriting it.
    // More advanced example contracts will detail how to inherit the swap router safely.
    // This example swaps DAI/WMATIC for single path swaps and DAI/USDC/WMATIC for multi path swaps.

    ISwapRouter public immutable swapRouter;
```

Hardcode the token contract addresses just for the example. In production, you would likely need to use an input parameter for this and pass the input into a memory variable, allowing the contract to change the pools and tokens it interacts with on a per-transaction basis, but for a better understanding of the process and conceptual simplicity, we are hardcoding them here.

```solidity
    address public constant DAI = 0x8f3Cf7ad23Cd3CaDbD9735AFf958023239c6A063;
    address public constant WMATIC = 0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270;
    address public constant USDC = 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174;


    constructor(ISwapRouter _swapRouter) {
        swapRouter = _swapRouter;
    }
```

## Exact Input Swaps

The caller must `approve` the contract to withdraw the tokens from the calling address's account to execute a swap. Keep in mind that because our contract is a contract itself and not an extension of the caller (us); we must also approve the Algebra Protocol router contract to use the tokens that our contract will be in possession of after they have been withdrawn from the calling address (us).

Then, transfer the `amount` of Dai from the calling address into our contract, and use `amount` as the value passed to the second `approve`.

```solidity
    /// @notice swapExactInputSingle swaps a fixed amount of DAI for a maximum possible amount of WMATIC
    /// using the DAI/WMATIC 0.3% pool by calling `exactInputSingle` in the swap router.
    /// @dev The calling address must approve this contract to spend at least `amountIn` worth of its DAI for this function to succeed.
    /// @param amountIn The exact amount of DAI that will be swapped for WMATIC.
    /// @return amountOut The amount of WMATIC received.
    function swapExactInputSingle(uint256 amountIn) external returns (uint256 amountOut) {
        // msg.sender must approve this contract

        // Transfer the specified amount of DAI to this contract.
        TransferHelper.safeTransferFrom(DAI, msg.sender, address(this), amountIn);

        // Approve the router to spend DAI.
        TransferHelper.safeApprove(DAI, address(swapRouter), amountIn);

```

### Swap Input Parameters

To execute the swap function, we need to fill the `ExactInputSingleParams` with the necessary swap data. These parameters are found in the smart contract interfaces, which can be browsed in ISwapRouter.

[ISwapRouter](https://docs.algebra.finance/en/docs/contracts/API-reference/periphery/ISwapRouter)

A little overview of the parameters:

- `tokenIn` is the contract address of the inbound token
- `tokenOut` is the contract address of the outbound token
- `recipient` is the destination address of the outbound token
- `deadline`: stands for unix time after which a swap will fail, to protect against long-pending transactions and wild swings in prices
- `amountOutMinimum`: is the parameter we are setting to zero, but this is a significant risk in production. For a real deployment, this value should be calculated using our SDK or an onchain price oracle – it’ll help protect against getting an unusually unreasonable price for a trade due to a front running sandwich or another type of price manipulation.
- `sqrtPriceLimitX96`: is also a parameter we are setting to zero - which makes it inactive. In production, this value might be used to set the limit for the price the swap will push the pool to – it can help us protect against price impact or for setting up logic in a variety of price-relevant mechanisms.

### Call the function

```solidity
        // Naively set amountOutMinimum to 0. In production, use an oracle or other data source to choose a safer value for amountOutMinimum.
        // We also set the sqrtPriceLimitx96 to be 0 to ensure we swap our exact input amount.
        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
                tokenIn: DAI,
                tokenOut: WMATIC,
                recipient: msg.sender,
                deadline: block.timestamp,
                amountIn: amountIn,
                amountOutMinimum: 0,
                sqrtPriceLimitX96: 0
            });

        // The call to `exactInputSingle` executes the swap.
        amountOut = swapRouter.exactInputSingle(params);
    }
```

## Exact Output Swaps

Exact Output swaps a minimum possible amount of the input token for a fixed amount of the outbound token. It can be seen as a less common swap style – but we find it quite useful in a variety of cases and circumstances.

As this example transfers in the inbound asset in anticipation of the swap, it is possible that some of the inbound tokens will be left over after the swap is executed, which is why we pay it back to the calling address once the swap is completed.

### Call the function

```solidity
/// @notice swapExactOutputSingle swaps a minimum possible amount of DAI for a fixed amount of WMATIC.
/// @dev The calling address must approve this contract to spend its DAI for this function to succeed. As the amount of input DAI is variable,
/// the calling address will need to approve for a slightly higher amount, anticipating some variance.
/// @param amountOut The exact amount of WMATIC to receive from the swap.
/// @param amountInMaximum The amount of DAI we are willing to spend to receive the specified amount of WMATIC.
/// @return amountIn The amount of DAI actually spent in the swap.
function swapExactOutputSingle(uint256 amountOut, uint256 amountInMaximum) external returns (uint256 amountIn) {
// Transfer the specified amount of DAI to this contract.
TransferHelper.safeTransferFrom(DAI, msg.sender, address(this), amountInMaximum);

        // Approve the router to spend the specified `amountInMaximum` of DAI.
        // In production, you should choose the maximum amount to spend based on oracles or other data sources to achieve a better swap.
        TransferHelper.safeApprove(DAI, address(swapRouter), amountInMaximum);

        ISwapRouter.ExactOutputSingleParams memory params =
            ISwapRouter.ExactOutputSingleParams({
                tokenIn: DAI,
                tokenOut: WMATIC,
                recipient: msg.sender,
                deadline: block.timestamp,
                amountOut: amountOut,
                amountInMaximum: amountInMaximum,
                sqrtPriceLimitX96: 0
            });

        // Executes the swap returning the amountIn needed to spend to receive the desired amountOut.
        amountIn = swapRouter.exactOutputSingle(params);

        // For exact output swaps, the amountInMaximum may not have all been spent.
        // If the actual amount spent (amountIn) is less than the specified maximum amount, we must refund the msg.sender and approve the swapRouter to spend 0.
        if (amountIn < amountInMaximum) {
            TransferHelper.safeApprove(DAI, address(swapRouter), 0);
            TransferHelper.safeTransfer(DAI, msg.sender, amountInMaximum - amountIn);
        }
    }

```

## A Complete Single Swap Contract

```solidity
// SPDX-License-Identifier: GPL-2.0-or-later
pragma solidity =0.7.6;
pragma abicoder v2;

import '@cryptoalgebra/periphery/contracts/libraries/TransferHelper.sol';
import '@cryptoalgebra/periphery/contracts/interfaces/ISwapRouter.sol';

contract SwapExamples {
    // For the scope of these swap examples,
    // we will detail the design considerations when using
    // `exactInput`, `exactInputSingle`, `exactOutput`, and  `exactOutputSingle`.

    // It should be noted that for the sake of these examples, we purposefully pass in the swap router instead of inherit the swap router for simplicity.
    // More advanced example contracts will detail how to inherit the swap router safely.

    ISwapRouter public immutable swapRouter;

    // This example swaps DAI/WMATIC for single path swaps and DAI/USDC/WMATIC for multi path swaps.

    address public constant DAI = 0x8f3Cf7ad23Cd3CaDbD9735AFf958023239c6A063;
    address public constant WMATIC = 0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270;
    address public constant USDC = 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174;

    constructor(ISwapRouter _swapRouter) {
        swapRouter = _swapRouter;
    }

    /// @notice swapExactInputSingle swaps a fixed amount of DAI for a maximum possible amount of WMATIC
    /// using the DAI/WMATIC 0.3% pool by calling `exactInputSingle` in the swap router.
    /// @dev The calling address must approve this contract to spend at least `amountIn` worth of its DAI for this function to succeed.
    /// @param amountIn The exact amount of DAI that will be swapped for WMATIC.
    /// @return amountOut The amount of WMATIC received.
    function swapExactInputSingle(uint256 amountIn) external returns (uint256 amountOut) {
        // msg.sender must approve this contract

        // Transfer the specified amount of DAI to this contract.
        TransferHelper.safeTransferFrom(DAI, msg.sender, address(this), amountIn);

        // Approve the router to spend DAI.
        TransferHelper.safeApprove(DAI, address(swapRouter), amountIn);

        // Naively set amountOutMinimum to 0. In production, use an oracle or other data source to choose a safer value for amountOutMinimum.
        // We also set the sqrtPriceLimitx96 to be 0 to ensure we swap our exact input amount.
        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
                tokenIn: DAI,
                tokenOut: WMATIC,
                recipient: msg.sender,
                deadline: block.timestamp,
                amountIn: amountIn,
                amountOutMinimum: 0,
                sqrtPriceLimitX96: 0
            });

        // The call to `exactInputSingle` executes the swap.
        amountOut = swapRouter.exactInputSingle(params);
    }

    /// @notice swapExactOutputSingle swaps a minimum possible amount of DAI for a fixed amount of WMATIC.
    /// @dev The calling address must approve this contract to spend its DAI for this function to succeed. As the amount of input DAI is variable,
    /// the calling address will need to approve for a slightly higher amount, anticipating some variance.
    /// @param amountOut The exact amount of WMATIC to receive from the swap.
    /// @param amountInMaximum The amount of DAI we are willing to spend to receive the specified amount of WMATIC.
    /// @return amountIn The amount of DAI actually spent in the swap.
    function swapExactOutputSingle(uint256 amountOut, uint256 amountInMaximum) external returns (uint256 amountIn) {
        // Transfer the specified amount of DAI to this contract.
        TransferHelper.safeTransferFrom(DAI, msg.sender, address(this), amountInMaximum);

        // Approve the router to spend the specifed `amountInMaximum` of DAI.
        // In production, you should choose the maximum amount to spend based on oracles or other data sources to acheive a better swap.
        TransferHelper.safeApprove(DAI, address(swapRouter), amountInMaximum);

        ISwapRouter.ExactOutputSingleParams memory params =
            ISwapRouter.ExactOutputSingleParams({
                tokenIn: DAI,
                tokenOut: WMATIC,
                recipient: msg.sender,
                deadline: block.timestamp,
                amountOut: amountOut,
                amountInMaximum: amountInMaximum,
                sqrtPriceLimitX96: 0
            });

        // Executes the swap returning the amountIn needed to spend to receive the desired amountOut.
        amountIn = swapRouter.exactOutputSingle(params);

        // For exact output swaps, the amountInMaximum may not have all been spent.
        // If the actual amount spent (amountIn) is less than the specified maximum amount, we must refund the msg.sender and approve the swapRouter to spend 0.
        if (amountIn < amountInMaximum) {
            TransferHelper.safeApprove(DAI, address(swapRouter), 0);
            TransferHelper.safeTransfer(DAI, msg.sender, amountInMaximum - amountIn);
        }
    }
}
```
