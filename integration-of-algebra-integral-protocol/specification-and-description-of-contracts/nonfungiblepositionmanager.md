# Nonfungible Position Manager

NFT positions

Wraps Algebra positions in the ERC721 non-fungible token interface

_Developer note: Credit to Uniswap Labs under GPL-2.0-or-later license: https://github.com/Uniswap/v3-periphery_

**Inherits:** [INonfungiblePositionManager](../contracts-api/Periphery/interfaces/INonfungiblePositionManager.md) [Multicall](../contracts-api/Periphery/base/Multicall.md) [ERC721Permit](../contracts-api/Periphery/base/ERC721Permit.md) [PeripheryImmutableState](../contracts-api/Periphery/base/PeripheryImmutableState.md) [PoolInitializer](../contracts-api/Periphery/base/PoolInitializer.md) [LiquidityManagement](../contracts-api/Periphery/base/LiquidityManagement.md) PeripheryValidation [SelfPermit](../contracts-api/Periphery/base/SelfPermit.md)

## Modifiers

### isAuthorizedForToken

```solidity
modifier isAuthorizedForToken(uint256 tokenId)
```

| Name    | Type    | Description |
| ------- | ------- | ----------- |
| tokenId | uint256 |             |

## Structs

### Position

```solidity
struct Position {
  uint88 nonce;
  address operator;
  uint80 poolId;
  int24 tickLower;
  int24 tickUpper;
  uint128 liquidity;
  uint256 feeGrowthInside0LastX128;
  uint256 feeGrowthInside1LastX128;
  uint128 tokensOwed0;
  uint128 tokensOwed1;
}
```

### MintParams

```solidity
struct MintParams {
  address token0;
  address token1;
  int24 tickLower;
  int24 tickUpper;
  uint256 amount0Desired;
  uint256 amount1Desired;
  uint256 amount0Min;
  uint256 amount1Min;
  address recipient;
  uint256 deadline;
}
```

### IncreaseLiquidityParams

```solidity
struct IncreaseLiquidityParams {
  uint256 tokenId;
  uint256 amount0Desired;
  uint256 amount1Desired;
  uint256 amount0Min;
  uint256 amount1Min;
  uint256 deadline;
}
```

### DecreaseLiquidityParams

```solidity
struct DecreaseLiquidityParams {
  uint256 tokenId;
  uint128 liquidity;
  uint256 amount0Min;
  uint256 amount1Min;
  uint256 deadline;
}
```

### CollectParams

```solidity
struct CollectParams {
  uint256 tokenId;
  address recipient;
  uint128 amount0Max;
  uint128 amount1Max;
}
```

## Public variables

### NONFUNGIBLE\_POSITION\_MANAGER\_ADMINISTRATOR\_ROLE

```solidity
bytes32 constant NONFUNGIBLE_POSITION_MANAGER_ADMINISTRATOR_ROLE = 0xff0e0466f109fcf4f5660899d8847c592e1e8dea30ffbe040704b23ad381d762
```

**Selector**: `0xb227aa79`

_Developer note: The role which has the right to change the farming center address_

### farmingCenter

```solidity
address farmingCenter
```

**Selector**: `0xdd56e5d8`

Returns the address of currently connected farming, if any

### farmingApprovals

```solidity
mapping(uint256 => address) farmingApprovals
```

**Selector**: `0x2d0b22de`

Returns the address of farming that is approved for this token, if any

### tokenFarmedIn

```solidity
mapping(uint256 => address) tokenFarmedIn
```

**Selector**: `0xe7ce18a3`

Returns the address of farming in which this token is farmed, if any

## Functions

### constructor

```solidity
constructor(address _factory, address _WNativeToken, address _tokenDescriptor_, address _poolDeployer) public
```

| Name              | Type    | Description |
| ----------------- | ------- | ----------- |
| \_factory         | address |             |
| \_WNativeToken    | address |             |
| _tokenDescriptor_ | address |             |
| \_poolDeployer    | address |             |

### multicall

```solidity
function multicall(bytes[] data) external payable returns (bytes[] results)
```

**Selector**: `0xac9650d8`

Call multiple functions in the current contract and return the data from all of them if they all succeed

_Developer note: The \`msg.value\` should not be trusted for any method callable from multicall._

| Name | Type     | Description                                                              |
| ---- | -------- | ------------------------------------------------------------------------ |
| data | bytes\[] | The encoded function data for each of the calls to make to this contract |

**Returns:**

| Name    | Type     | Description                                           |
| ------- | -------- | ----------------------------------------------------- |
| results | bytes\[] | The results from each of the calls passed in via data |

### positions

```solidity
function positions(uint256 tokenId) external view returns (uint88 nonce, address operator, address token0, address token1, int24 tickLower, int24 tickUpper, uint128 liquidity, uint256 feeGrowthInside0LastX128, uint256 feeGrowthInside1LastX128, uint128 tokensOwed0, uint128 tokensOwed1)
```

**Selector**: `0x99fbab88`

Returns the position information associated with a given token ID.

_Developer note: Throws if the token ID is not valid._

| Name    | Type    | Description                                      |
| ------- | ------- | ------------------------------------------------ |
| tokenId | uint256 | The ID of the token that represents the position |

**Returns:**

| Name                     | Type    | Description                                                                      |
| ------------------------ | ------- | -------------------------------------------------------------------------------- |
| nonce                    | uint88  | The nonce for permits                                                            |
| operator                 | address | The address that is approved for spending                                        |
| token0                   | address | The address of the token0 for a specific pool                                    |
| token1                   | address | The address of the token1 for a specific pool                                    |
| tickLower                | int24   | The lower end of the tick range for the position                                 |
| tickUpper                | int24   | The higher end of the tick range for the position                                |
| liquidity                | uint128 | The liquidity of the position                                                    |
| feeGrowthInside0LastX128 | uint256 | The fee growth of token0 as of the last action on the individual position        |
| feeGrowthInside1LastX128 | uint256 | The fee growth of token1 as of the last action on the individual position        |
| tokensOwed0              | uint128 | The uncollected amount of token0 owed to the position as of the last computation |
| tokensOwed1              | uint128 | The uncollected amount of token1 owed to the position as of the last computation |

### mint

```solidity
function mint(struct INonfungiblePositionManager.MintParams params) external payable returns (uint256 tokenId, uint128 liquidity, uint256 amount0, uint256 amount1)
```

**Selector**: `0x9cc1a283`

Creates a new position wrapped in a NFT

_Developer note: Call this when the pool does exist and is initialized. Note that if the pool is created but not initialized a method does not exist, i.e. the pool is assumed to be initialized._

| Name   | Type                                          | Description                                                                  |
| ------ | --------------------------------------------- | ---------------------------------------------------------------------------- |
| params | struct INonfungiblePositionManager.MintParams | The params necessary to mint a position, encoded as `MintParams` in calldata |

**Returns:**

| Name      | Type    | Description                                             |
| --------- | ------- | ------------------------------------------------------- |
| tokenId   | uint256 | The ID of the token that represents the minted position |
| liquidity | uint128 | The liquidity delta amount as a result of the increase  |
| amount0   | uint256 | The amount of token0                                    |
| amount1   | uint256 | The amount of token1                                    |

### increaseLiquidity

```solidity
function increaseLiquidity(struct INonfungiblePositionManager.IncreaseLiquidityParams params) external payable returns (uint128 liquidity, uint256 amount0, uint256 amount1)
```

**Selector**: `0x219f5d17`

Increases the amount of liquidity in a position, with tokens paid by the \`msg.sender\`

| Name   | Type                                                       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------ | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| params | struct INonfungiblePositionManager.IncreaseLiquidityParams | tokenId The ID of the token for which liquidity is being increased, amount0Desired The desired amount of token0 to be spent, amount1Desired The desired amount of token1 to be spent, amount0Min The minimum amount of token0 to spend, which serves as a slippage check, amount1Min The minimum amount of token1 to spend, which serves as a slippage check, deadline The time by which the transaction must be included to effect the change |

**Returns:**

| Name      | Type    | Description                                            |
| --------- | ------- | ------------------------------------------------------ |
| liquidity | uint128 | The liquidity delta amount as a result of the increase |
| amount0   | uint256 | The amount of token0 to achieve resulting liquidity    |
| amount1   | uint256 | The amount of token1 to achieve resulting liquidity    |

### decreaseLiquidity

```solidity
function decreaseLiquidity(struct INonfungiblePositionManager.DecreaseLiquidityParams params) external payable returns (uint256 amount0, uint256 amount1)
```

**Selector**: `0x0c49ccbe`

Decreases the amount of liquidity in a position and accounts it to the position

| Name   | Type                                                       | Description                                                                                                                                                                                                                                                                                                                                                                                        |
| ------ | ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| params | struct INonfungiblePositionManager.DecreaseLiquidityParams | tokenId The ID of the token for which liquidity is being decreased, amount The amount by which liquidity will be decreased, amount0Min The minimum amount of token0 that should be accounted for the burned liquidity, amount1Min The minimum amount of token1 that should be accounted for the burned liquidity, deadline The time by which the transaction must be included to effect the change |

**Returns:**

| Name    | Type    | Description                                                  |
| ------- | ------- | ------------------------------------------------------------ |
| amount0 | uint256 | The amount of token0 accounted to the position's tokens owed |
| amount1 | uint256 | The amount of token1 accounted to the position's tokens owed |

### collect

```solidity
function collect(struct INonfungiblePositionManager.CollectParams params) external payable returns (uint256 amount0, uint256 amount1)
```

**Selector**: `0xfc6f7865`

Collects up to a maximum amount of fees owed to a specific position to the recipient

| Name   | Type                                             | Description                                                                                                                                                                                                                  |
| ------ | ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| params | struct INonfungiblePositionManager.CollectParams | tokenId The ID of the NFT for which tokens are being collected, recipient The account that should receive the tokens, amount0Max The maximum amount of token0 to collect, amount1Max The maximum amount of token1 to collect |

**Returns:**

| Name    | Type    | Description                            |
| ------- | ------- | -------------------------------------- |
| amount0 | uint256 | The amount of fees collected in token0 |
| amount1 | uint256 | The amount of fees collected in token1 |

### burn

```solidity
function burn(uint256 tokenId) external payable
```

**Selector**: `0x42966c68`

Burns a token ID, which deletes it from the NFT contract. The token must have 0 liquidity and all tokens must be collected first.

| Name    | Type    | Description                              |
| ------- | ------- | ---------------------------------------- |
| tokenId | uint256 | The ID of the token that is being burned |

### approveForFarming

```solidity
function approveForFarming(uint256 tokenId, bool approve, address farmingAddress) external payable
```

**Selector**: `0x832f630a`

Changes approval of token ID for farming.

| Name           | Type    | Description                                             |
| -------------- | ------- | ------------------------------------------------------- |
| tokenId        | uint256 | The ID of the token that is being approved / unapproved |
| approve        | bool    | New status of approval                                  |
| farmingAddress | address | The address of farming: used to prevent tx frontrun     |

### switchFarmingStatus

```solidity
function switchFarmingStatus(uint256 tokenId, bool toActive) external
```

**Selector**: `0x70227515`

Changes farming status of token to 'farmed' or 'not farmed'

_Developer note: can be called only by farmingCenter_

| Name     | Type    | Description         |
| -------- | ------- | ------------------- |
| tokenId  | uint256 | The ID of the token |
| toActive | bool    | The new status      |

### setFarmingCenter

```solidity
function setFarmingCenter(address newFarmingCenter) external
```

**Selector**: `0x4d10862d`

Changes address of farmingCenter

_Developer note: can be called only by factory owner or NONFUNGIBLE\_POSITION\_MANAGER\_ADMINISTRATOR\_ROLE_

| Name             | Type    | Description                      |
| ---------------- | ------- | -------------------------------- |
| newFarmingCenter | address | The new address of farmingCenter |

### tokenURI

```solidity
function tokenURI(uint256 tokenId) public view returns (string)
```

**Selector**: `0xc87b56dd`

_Developer note: Returns the Uniform Resource Identifier (URI) for \`tokenId\` token._

| Name    | Type    | Description |
| ------- | ------- | ----------- |
| tokenId | uint256 |             |

**Returns:**

| Name | Type   | Description |
| ---- | ------ | ----------- |
| \[0] | string |             |

### getApproved

```solidity
function getApproved(uint256 tokenId) public view returns (address)
```

**Selector**: `0x081812fc`

\*Developer note: Returns the account approved for \`tokenId\` token.

Requirements:

* \`tokenId\` must exist.\*

| Name    | Type    | Description |
| ------- | ------- | ----------- |
| tokenId | uint256 |             |

**Returns:**

| Name | Type    | Description |
| ---- | ------- | ----------- |
| \[0] | address |             |

### isApprovedOrOwner

```solidity
function isApprovedOrOwner(address spender, uint256 tokenId) external view returns (bool)
```

**Selector**: `0x430c2081`

Returns whether \`spender\` is allowed to manage \`tokenId\`

_Developer note: Requirement: \`tokenId\` must exist_

| Name    | Type    | Description |
| ------- | ------- | ----------- |
| spender | address |             |
| tokenId | uint256 |             |

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| \[0] | bool |             |

## Events

### IncreaseLiquidity

```solidity
event IncreaseLiquidity(uint256 tokenId, uint128 liquidityDesired, uint128 actualLiquidity, uint256 amount0, uint256 amount1, address pool)
```

Emitted when liquidity is increased for a position NFT

_Developer note: Also emitted when a token is minted_

| Name             | Type    | Description                                                                                                    |
| ---------------- | ------- | -------------------------------------------------------------------------------------------------------------- |
| tokenId          | uint256 | The ID of the token for which liquidity was increased                                                          |
| liquidityDesired | uint128 | The amount by which liquidity for the NFT position was increased                                               |
| actualLiquidity  | uint128 | the actual liquidity that was added into a pool. Could differ from _liquidity_ when using FeeOnTransfer tokens |
| amount0          | uint256 | The amount of token0 that was paid for the increase in liquidity                                               |
| amount1          | uint256 | The amount of token1 that was paid for the increase in liquidity                                               |
| pool             | address |                                                                                                                |

### DecreaseLiquidity

```solidity
event DecreaseLiquidity(uint256 tokenId, uint128 liquidity, uint256 amount0, uint256 amount1)
```

Emitted when liquidity is decreased for a position NFT

| Name      | Type    | Description                                                           |
| --------- | ------- | --------------------------------------------------------------------- |
| tokenId   | uint256 | The ID of the token for which liquidity was decreased                 |
| liquidity | uint128 | The amount by which liquidity for the NFT position was decreased      |
| amount0   | uint256 | The amount of token0 that was accounted for the decrease in liquidity |
| amount1   | uint256 | The amount of token1 that was accounted for the decrease in liquidity |

### Collect

```solidity
event Collect(uint256 tokenId, address recipient, uint256 amount0, uint256 amount1)
```

Emitted when tokens are collected for a position NFT

_Developer note: The amounts reported may not be exactly equivalent to the amounts transferred, due to rounding behavior_

| Name      | Type    | Description                                                    |
| --------- | ------- | -------------------------------------------------------------- |
| tokenId   | uint256 | The ID of the token for which underlying tokens were collected |
| recipient | address | The address of the account that received the collected tokens  |
| amount0   | uint256 | The amount of token0 owed to the position that was collected   |
| amount1   | uint256 | The amount of token1 owed to the position that was collected   |

### FarmingFailed

```solidity
event FarmingFailed(uint256 tokenId)
```

Emitted if farming failed in call from NonfungiblePositionManager.

_Developer note: Should never be emitted_

| Name    | Type    | Description                   |
| ------- | ------- | ----------------------------- |
| tokenId | uint256 | The ID of corresponding token |

### FarmingCenter

```solidity
event FarmingCenter(address farmingCenterAddress)
```

Emitted after farming center address change

| Name                 | Type    | Description                                 |
| -------------------- | ------- | ------------------------------------------- |
| farmingCenterAddress | address | The new address of connected farming center |
