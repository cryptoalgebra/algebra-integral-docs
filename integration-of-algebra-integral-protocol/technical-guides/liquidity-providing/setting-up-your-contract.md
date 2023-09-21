---
ID: "0"
title: "Set up your contract"
---

## Setting up the Contract

First, declare the solidity version used to compile the contract and `abicoder v2` to allow arbitrary nested arrays and structs to be encoded and decoded in calldata, a feature that we use when transacting with a pool.

```solidity
// SPDX-License-Identifier: GPL-2.0-or-later
pragma solidity =0.7.6;
pragma abicoder v2;
```

After that, import the contracts needed from the npm package installation.

```solidity
import '@cryptoalgebra/core/contracts/interfaces/IAlgebraPool.sol';
import '@cryptoalgebra/core/contracts/libraries/TickMath.sol';
import '@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol';
import '@cryptoalgebra/periphery/contracts/interfaces/ISwapRouter.sol';
import '@cryptoalgebra/periphery/contracts/interfaces/INonfungiblePositionManager.sol';
import '@cryptoalgebra/periphery/contracts/libraries/TransferHelper.sol';
import '@cryptoalgebra/periphery/contracts/base/LiquidityManagement.sol';
```

Then, create a contract called `LiquidityExamples` and inherit both `IERC721Receiver` and `LiquidityManagement`.

For this case, we've chosen to hardcode the token contract addresses. Most likely, you would use an input parameter for this in production, allowing you to change the pools and tokens you are interacting with on a per-transaction basis.

```solidity
contract LiquidityExamples is IERC721Receiver {

    address public constant DAI = 0x6B175474E89094C44Da98b954EedeAC495271d0F;
    address public constant USDC = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
```

Declare an immutable public variable `nonfungiblePositionManager` of type `INonfungiblePositionManager`.

```solidity
    INonfungiblePositionManager public immutable nonfungiblePositionManager;
```

## Enabling ERC721 Interactions

EEvery NFT is identified by a unique uint256 ID inside the ERC-721 smart contract, declared as the `tokenId`

[What is NFT?](https://ethereum.org/en/nft/)

To allow deposits of ERC721 expressions of liquidity, create a struct called `Deposit`, a mapping of `uint256` to the `Deposit` struct, then declare that mapping as a public variable `deposits`.

```solidity
    struct Deposit {
        address owner;
        uint128 liquidity;
        address token0;
        address token1;
    }

    mapping(uint256 => Deposit) public deposits;
```

## The Constructor

Declare the constructor here, which is executed once when the contract is deployed. Our constructor hard codes the address of the non-fungible position manager interface, router, and the periphery immutable state constructor, which requires the factory and the address of WMATIC.

```solidity
    constructor(
        INonfungiblePositionManager _nonfungiblePositionManager,
        address _factory,
        address _WMATIC
    ) PeripheryImmutableState(_factory, _WMATIC) {
        nonfungiblePositionManager = _nonfungiblePositionManager;
    }
```

## Allowing custody of ERC721 tokens

To allow the contract to custody ERC721 tokens, implement the `onERC721Received` function within the inherited `IERC721Receiver.sol` contract.

[IERC721Receiver Contract](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/IERC721Receiver.sol)

The `from` identifier may be omitted because it is not used.

```solidity
    function onERC721Received(
        address operator,
        address,
        uint256 tokenId,
        bytes calldata
    ) external override returns (bytes4) {
        // get position information
        _createDeposit(operator, tokenId);
        return this.onERC721Received.selector;
    }
```

## Creating a Deposit

To add a `Deposit` instance to the `deposits` mapping, create an internal function called `_createDeposit` that destructures the `positions` struct returned by `positions` in `nonfungiblePositionManager.sol`. Pass the relevant variables `token0`, `token1` and `liquidity` to the `deposits` mapping.

```solidity
    function _createDeposit(address owner, uint256 tokenId) internal {
        (, , address token0, address token1, , , uint128 liquidity, , , , ) =
            nonfungiblePositionManager.positions(tokenId);

        // set the owner and data for position
        // operator is msg.sender
        deposits[tokenId] = Deposit({owner: owner, liquidity: liquidity, token0: token0, token1: token1});
    }

```

## The Full Contract Setup

```solidity
// SPDX-License-Identifier: GPL-2.0-or-later
pragma solidity =0.7.6;
pragma abicoder v2;

import '@cryptoalgebra/core/contracts/interfaces/IAlgebraPool.sol';
import '@cryptoalgebra/core/contracts/libraries/TickMath.sol';
import '@cryptoalgebra/periphery/contracts/libraries/TransferHelper.sol';
import '@cryptoalgebra/periphery/contracts/interfaces/INonfungiblePositionManager.sol';
import '@cryptoalgebra/periphery/contracts/base/LiquidityManagement.sol';
import '@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol';

contract LiquidityExamples is IERC721Receiver {
    address public constant DAI = 0x8f3Cf7ad23Cd3CaDbD9735AFf958023239c6A063;
    address public constant USDC = 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174;


    INonfungiblePositionManager public immutable nonfungiblePositionManager;

    /// @notice Represents the deposit of an NFT
    struct Deposit {
        address owner;
        uint128 liquidity;
        address token0;
        address token1;
    }

    /// @dev deposits[tokenId] => Deposit
    mapping(uint256 => Deposit) public deposits;

    constructor(
        INonfungiblePositionManager _nonfungiblePositionManager
    ) {
        nonfungiblePositionManager = _nonfungiblePositionManager;
    }

    // Implementing `onERC721Received` so this contract can receive custody of erc721 tokens
    function onERC721Received(
        address operator,
        address,
        uint256 tokenId,
        bytes calldata
    ) external override returns (bytes4) {
        // get position information

        _createDeposit(operator, tokenId);

        return this.onERC721Received.selector;
    }

    function _createDeposit(address owner, uint256 tokenId) internal {
        (, , address token0, address token1, , , uint128 liquidity, , , , ) =
            nonfungiblePositionManager.positions(tokenId);

        // set the owner and data for position
        // operator is msg.sender
        deposits[tokenId] = Deposit({owner: owner, liquidity: liquidity, token0: token0, token1: token1});
    }
}
```
