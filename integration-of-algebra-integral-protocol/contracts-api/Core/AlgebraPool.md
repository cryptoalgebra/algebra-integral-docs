

# AlgebraPool


Algebra concentrated liquidity pool

This contract is responsible for liquidity positions, swaps and flashloans

*Developer note: Version: Algebra Integral*

## Modifiers
### onlyValidTicks

```solidity
modifier onlyValidTicks(int24 bottomTick, int24 topTick)
```

Check that the lower and upper ticks do not violate the boundaries of allowed ticks and are specified in the correct order

| Name | Type | Description |
| ---- | ---- | ----------- |
| bottomTick | int24 |  |
| topTick | int24 |  |


## Structs
### GlobalState

The struct with important state values of pool

*Developer note: fits into one storage slot*

```solidity
struct GlobalState {
  uint160 price;
  int24 tick;
  uint16 lastFee;
  uint8 pluginConfig;
  uint16 communityFee;
  bool unlocked;
}
```

| Name | Description |
| ---- | ----------- |
| price | The square root of the current price in Q64.96 format |
| tick | The current tick (price(tick) <= current price). May not always be equal to SqrtTickMath.getTickAtSqrtRatio(price) if the price is on a tick boundary |
| lastFee | The current (last known) fee in hundredths of a bip, i.e. 1e-6 (so 100 is 0.01%). May be obsolete if using dynamic fee plugin |
| pluginConfig | The current plugin config as bitmap. Each bit is responsible for enabling/disabling the hooks, the last bit turns on/off dynamic fees logic |
| communityFee | The community fee represented as a percent of all collected fee in thousandths, i.e. 1e-3 (so 100 is 10%) |
| unlocked | Reentrancy lock flag, true if the pool currently is unlocked, otherwise - false |

### Position

```solidity
struct Position {
  uint256 liquidity;
  uint256 innerFeeGrowth0Token;
  uint256 innerFeeGrowth1Token;
  uint128 fees0;
  uint128 fees1;
}
```

| Name | Description |
| ---- | ----------- |
| liquidity | The amount of liquidity in the position |
| innerFeeGrowth0Token | Fee growth of token0 inside the tick range as of the last mint/burn/poke |
| innerFeeGrowth1Token | Fee growth of token1 inside the tick range as of the last mint/burn/poke |
| fees0 | The computed amount of token0 owed to the position as of the last mint/burn/poke |
| fees1 | The computed amount of token1 owed to the position as of the last mint/burn/poke |

## Public variables
### maxLiquidityPerTick
```solidity
uint128 constant maxLiquidityPerTick = 191757638537527648490752896198553
```
**Selector**: `0x70cf754a`

The maximum amount of position liquidity that can use any tick in the range

*Developer note: This parameter is enforced per tick to prevent liquidity from overflowing a uint128 at any point, and
also prevents out-of-range liquidity from being used to prevent adding in-range liquidity to a pool*

### factory
```solidity
address immutable factory
```
**Selector**: `0xc45a0155`

The Algebra factory contract, which must adhere to the IAlgebraFactory interface


### token0
```solidity
address immutable token0
```
**Selector**: `0x0dfe1681`

The first of the two tokens of the pool, sorted by address


### token1
```solidity
address immutable token1
```
**Selector**: `0xd21220a7`

The second of the two tokens of the pool, sorted by address


### communityVault
```solidity
address immutable communityVault
```
**Selector**: `0x53e97868`

The contract to which community fees are transferred


### totalFeeGrowth0Token
```solidity
uint256 totalFeeGrowth0Token
```
**Selector**: `0x6378ae44`

The fee growth as a Q128.128 fees of token0 collected per unit of liquidity for the entire life of the pool

*Developer note: This value can overflow the uint256*

### totalFeeGrowth1Token
```solidity
uint256 totalFeeGrowth1Token
```
**Selector**: `0xecdecf42`

The fee growth as a Q128.128 fees of token1 collected per unit of liquidity for the entire life of the pool

*Developer note: This value can overflow the uint256*

### globalState
```solidity
struct AlgebraPoolBase.GlobalState globalState
```
**Selector**: `0xe76c01e4`

The globalState structure in the pool stores many values but requires only one slot
and is exposed as a single method to save gas when accessed externally.

*Developer note: **important security note: caller should check &#x60;unlocked&#x60; flag to prevent read-only reentrancy***

### ticks
```solidity
mapping(int24 => struct TickManagement.Tick) ticks
```
**Selector**: `0xf30dba93`

Look up information about a specific tick in the pool

*Developer note: **important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### communityFeeLastTimestamp
```solidity
uint32 communityFeeLastTimestamp
```
**Selector**: `0x1131b110`

The timestamp of the last sending of tokens to community vault


### plugin
```solidity
address plugin
```
**Selector**: `0xef01df4f`

Returns the address of currently used plugin

*Developer note: The plugin is subject to change*

### tickTable
```solidity
mapping(int16 => uint256) tickTable
```
**Selector**: `0xc677e3e0`

Returns 256 packed tick initialized boolean values. See TickTree for more information


### nextTickGlobal
```solidity
int24 nextTickGlobal
```
**Selector**: `0xd5c35a7e`

The next initialized tick after current global tick

*Developer note: **important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### prevTickGlobal
```solidity
int24 prevTickGlobal
```
**Selector**: `0x050a4d21`

The previous initialized tick before (or at) current global tick

*Developer note: **important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### liquidity
```solidity
uint128 liquidity
```
**Selector**: `0x1a686502`

The currently in range liquidity available to the pool

*Developer note: This value has no relationship to the total liquidity across all ticks.
Returned value cannot exceed type(uint128).max
**important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### tickSpacing
```solidity
int24 tickSpacing
```
**Selector**: `0xd0c93a7c`

The current tick spacing

*Developer note: Ticks can only be initialized by new mints at multiples of this value
e.g.: a tickSpacing of 60 means ticks can be initialized every 60th tick, i.e., ..., -120, -60, 0, 60, 120, ...
However, tickspacing can be changed after the ticks have been initialized.
This value is an int24 to avoid casting even though it is always positive.*

### tickTreeRoot
```solidity
uint32 tickTreeRoot
```
**Selector**: `0x578b9a36`

The root of tick search tree

*Developer note: Each bit corresponds to one node in the second layer of tick tree: &#x27;1&#x27; if node has at least one active bit.
**important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### tickTreeSecondLayer
```solidity
mapping(int16 => uint256) tickTreeSecondLayer
```
**Selector**: `0xd8619037`

The second layer of tick search tree

*Developer note: Each bit in node corresponds to one node in the leafs layer (&#x60;tickTable&#x60;) of tick tree: &#x27;1&#x27; if leaf has at least one active bit.
**important security note: caller should check reentrancy lock to prevent read-only reentrancy***

### positions
```solidity
mapping(bytes32 => struct Positions.Position) positions
```
**Selector**: `0x514ea4bf`

Returns the information about a position by the position&#x27;s key

*Developer note: **important security note: caller should check reentrancy lock to prevent read-only reentrancy***


## Functions
### safelyGetStateOfAMM

```solidity
function safelyGetStateOfAMM() external view returns (uint160 sqrtPrice, int24 tick, uint16 lastFee, uint8 pluginConfig, uint128 activeLiquidity, int24 nextTick, int24 previousTick)
```
**Selector**: `0x97ce1c51`

Safely get most important state values of Algebra Integral AMM

*Developer note: safe from read-only reentrancy getter function*

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| sqrtPrice | uint160 | The current price of the pool as a sqrt(dToken1/dToken0) Q64.96 value |
| tick | int24 | The current global tick of the pool. May not always be equal to SqrtTickMath.getTickAtSqrtRatio(price) if the price is on a tick boundary |
| lastFee | uint16 | The current (last known) pool fee value in hundredths of a bip, i.e. 1e-6 (so '100' is '0.01%'). May be obsolete if using dynamic fee plugin |
| pluginConfig | uint8 | The current plugin config as bitmap. Each bit is responsible for enabling/disabling the hooks, the last bit turns on/off dynamic fees logic |
| activeLiquidity | uint128 | The currently in-range liquidity available to the pool |
| nextTick | int24 | The next initialized tick after current global tick |
| previousTick | int24 | The previous initialized tick before (or at) current global tick |

### isUnlocked

```solidity
function isUnlocked() external view returns (bool unlocked)
```
**Selector**: `0x8380edb7`

Allows to easily get current reentrancy lock status

*Developer note: can be used to prevent read-only reentrancy.
This method just returns &#x60;globalState.unlocked&#x60; value*

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| unlocked | bool | Reentrancy lock flag, true if the pool currently is unlocked, otherwise - false |

### getCommunityFeePending

```solidity
function getCommunityFeePending() external view returns (uint128, uint128)
```
**Selector**: `0x7bd78025`

The amounts of token0 and token1 that will be sent to the vault

*Developer note: Will be sent COMMUNITY_FEE_TRANSFER_FREQUENCY after communityFeeLastTimestamp*

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| [0] | uint128 |  |
| [1] | uint128 |  |

### getReserves

```solidity
function getReserves() external view returns (uint128, uint128)
```
**Selector**: `0x0902f1ac`

The tracked token0 and token1 reserves of pool

*Developer note: If at any time the real balance is larger, the excess will be transferred to liquidity providers as additional fee.
If the balance exceeds uint128, the excess will be sent to the communityVault.*

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| [0] | uint128 |  |
| [1] | uint128 |  |

### fee

```solidity
function fee() external view returns (uint16 currentFee)
```
**Selector**: `0xddca3f43`

The current pool fee value

*Developer note: In case dynamic fee is enabled in the pool, this method will call the plugin to get the current fee.
If the plugin implements complex fee logic, this method may return an incorrect value or revert.
In this case, see the plugin implementation and related documentation.
**important security note: caller should check reentrancy lock to prevent read-only reentrancy***

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| currentFee | uint16 | The current pool fee value in hundredths of a bip, i.e. 1e-6 |

### initialize

```solidity
function initialize(uint160 initialPrice) external
```
**Selector**: `0xf637731d`

Sets the initial price for the pool

*Developer note: Price is represented as a sqrt(amountToken1/amountToken0) Q64.96 value
Initialization should be done in one transaction with pool creation to avoid front-running*

| Name | Type | Description |
| ---- | ---- | ----------- |
| initialPrice | uint160 | The initial sqrt price of the pool as a Q64.96 |

### mint

```solidity
function mint(address leftoversRecipient, address recipient, int24 bottomTick, int24 topTick, uint128 liquidityDesired, bytes data) external returns (uint256 amount0, uint256 amount1, uint128 liquidityActual)
```
**Selector**: `0xaafe29c0`

Adds liquidity for the given recipient/bottomTick/topTick position

*Developer note: The caller of this method receives a callback in the form of [IAlgebraMintCallback#algebraMintCallback](interfaces/callback/IAlgebraMintCallback.md#algebraMintCallback)
in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
on bottomTick, topTick, the amount of liquidity, and the current price.*

| Name | Type | Description |
| ---- | ---- | ----------- |
| leftoversRecipient | address | The address which will receive potential surplus of paid tokens |
| recipient | address | The address for which the liquidity will be created |
| bottomTick | int24 | The lower tick of the position in which to add liquidity |
| topTick | int24 | The upper tick of the position in which to add liquidity |
| liquidityDesired | uint128 | The desired amount of liquidity to mint |
| data | bytes | Any data that should be passed through to the callback |

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| amount0 | uint256 | The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback |
| amount1 | uint256 | The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback |
| liquidityActual | uint128 | The actual minted amount of liquidity |

### burn

```solidity
function burn(int24 bottomTick, int24 topTick, uint128 amount, bytes data) external returns (uint256 amount0, uint256 amount1)
```
**Selector**: `0x3b3bc70e`

Burn liquidity from the sender and account tokens owed for the liquidity to the position

*Developer note: Can be used to trigger a recalculation of fees owed to a position by calling with an amount of 0
Fees must be collected separately via a call to [#collect](#collect)*

| Name | Type | Description |
| ---- | ---- | ----------- |
| bottomTick | int24 | The lower tick of the position for which to burn liquidity |
| topTick | int24 | The upper tick of the position for which to burn liquidity |
| amount | uint128 | How much liquidity to burn |
| data | bytes | Any data that should be passed through to the plugin |

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| amount0 | uint256 | The amount of token0 sent to the recipient |
| amount1 | uint256 | The amount of token1 sent to the recipient |

### collect

```solidity
function collect(address recipient, int24 bottomTick, int24 topTick, uint128 amount0Requested, uint128 amount1Requested) external returns (uint128 amount0, uint128 amount1)
```
**Selector**: `0x4f1eb3d8`

Collects tokens owed to a position

*Developer note: Does not recompute fees earned, which must be done either via mint or burn of any amount of liquidity.
Collect must be called by the position owner. To withdraw only token0 or only token1, amount0Requested or
amount1Requested may be set to zero. To withdraw all tokens owed, caller may pass any value greater than the
actual tokens owed, e.g. type(uint128).max. Tokens owed may be from accumulated swap fees or burned liquidity.*

| Name | Type | Description |
| ---- | ---- | ----------- |
| recipient | address | The address which should receive the fees collected |
| bottomTick | int24 | The lower tick of the position for which to collect fees |
| topTick | int24 | The upper tick of the position for which to collect fees |
| amount0Requested | uint128 | How much token0 should be withdrawn from the fees owed |
| amount1Requested | uint128 | How much token1 should be withdrawn from the fees owed |

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| amount0 | uint128 | The amount of fees collected in token0 |
| amount1 | uint128 | The amount of fees collected in token1 |

### swap

```solidity
function swap(address recipient, bool zeroToOne, int256 amountRequired, uint160 limitSqrtPrice, bytes data) external returns (int256 amount0, int256 amount1)
```
**Selector**: `0x128acb08`

Swap token0 for token1, or token1 for token0

*Developer note: The caller of this method receives a callback in the form of [IAlgebraSwapCallback#algebraSwapCallback](interfaces/callback/IAlgebraSwapCallback.md#algebraSwapCallback)*

| Name | Type | Description |
| ---- | ---- | ----------- |
| recipient | address | The address to receive the output of the swap |
| zeroToOne | bool | The direction of the swap, true for token0 to token1, false for token1 to token0 |
| amountRequired | int256 | The amount of the swap, which implicitly configures the swap as exact input (positive), or exact output (negative) |
| limitSqrtPrice | uint160 | The Q64.96 sqrt price limit. If zero for one, the price cannot be less than this value after the swap. If one for zero, the price cannot be greater than this value after the swap |
| data | bytes | Any data to be passed through to the callback. If using the Router it should contain [SwapRouter#SwapCallbackData](../Periphery/SwapRouter.md#SwapCallbackData) |

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| amount0 | int256 | The delta of the balance of token0 of the pool, exact when negative, minimum when positive |
| amount1 | int256 | The delta of the balance of token1 of the pool, exact when negative, minimum when positive |

### swapWithPaymentInAdvance

```solidity
function swapWithPaymentInAdvance(address leftoversRecipient, address recipient, bool zeroToOne, int256 amountToSell, uint160 limitSqrtPrice, bytes data) external returns (int256 amount0, int256 amount1)
```
**Selector**: `0x9e4e0227`

Swap token0 for token1, or token1 for token0 with prepayment

*Developer note: The caller of this method receives a callback in the form of [IAlgebraSwapCallback#algebraSwapCallback](interfaces/callback/IAlgebraSwapCallback.md#algebraSwapCallback)
caller must send tokens in callback before swap calculation
the actually sent amount of tokens is used for further calculations*

| Name | Type | Description |
| ---- | ---- | ----------- |
| leftoversRecipient | address | The address which will receive potential surplus of paid tokens |
| recipient | address | The address to receive the output of the swap |
| zeroToOne | bool | The direction of the swap, true for token0 to token1, false for token1 to token0 |
| amountToSell | int256 | The amount of the swap, only positive (exact input) amount allowed |
| limitSqrtPrice | uint160 | The Q64.96 sqrt price limit. If zero for one, the price cannot be less than this value after the swap. If one for zero, the price cannot be greater than this value after the swap |
| data | bytes | Any data to be passed through to the callback. If using the Router it should contain [SwapRouter#SwapCallbackData](../Periphery/SwapRouter.md#SwapCallbackData) |

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| amount0 | int256 | The delta of the balance of token0 of the pool, exact when negative, minimum when positive |
| amount1 | int256 | The delta of the balance of token1 of the pool, exact when negative, minimum when positive |

### flash

```solidity
function flash(address recipient, uint256 amount0, uint256 amount1, bytes data) external
```
**Selector**: `0x490e6cbc`

Receive token0 and/or token1 and pay it back, plus a fee, in the callback

*Developer note: The caller of this method receives a callback in the form of [IAlgebraFlashCallback#algebraFlashCallback](interfaces/callback/IAlgebraFlashCallback.md#algebraFlashCallback)
All excess tokens paid in the callback are distributed to currently in-range liquidity providers as an additional fee.
If there are no in-range liquidity providers, the fee will be transferred to the first active provider in the future*

| Name | Type | Description |
| ---- | ---- | ----------- |
| recipient | address | The address which will receive the token0 and token1 amounts |
| amount0 | uint256 | The amount of token0 to send |
| amount1 | uint256 | The amount of token1 to send |
| data | bytes | Any data to be passed through to the callback |

### setCommunityFee

```solidity
function setCommunityFee(uint16 newCommunityFee) external
```
**Selector**: `0x240a875a`

Set the community&#x27;s % share of the fees. Only factory owner or POOLS_ADMINISTRATOR_ROLE role

| Name | Type | Description |
| ---- | ---- | ----------- |
| newCommunityFee | uint16 | The new community fee percent in thousandths (1e-3) |

### setTickSpacing

```solidity
function setTickSpacing(int24 newTickSpacing) external
```
**Selector**: `0xf085a610`

Set the new tick spacing values. Only factory owner or POOLS_ADMINISTRATOR_ROLE role

| Name | Type | Description |
| ---- | ---- | ----------- |
| newTickSpacing | int24 | The new tick spacing value |

### setPlugin

```solidity
function setPlugin(address newPluginAddress) external
```
**Selector**: `0xcc1f97cf`

Set the new plugin address. Only factory owner or POOLS_ADMINISTRATOR_ROLE role

| Name | Type | Description |
| ---- | ---- | ----------- |
| newPluginAddress | address | The new plugin address |

### setPluginConfig

```solidity
function setPluginConfig(uint8 newConfig) external
```
**Selector**: `0xbca57f81`

Set new plugin config

| Name | Type | Description |
| ---- | ---- | ----------- |
| newConfig | uint8 | In the new configuration of the plugin, each bit of which is responsible for a particular hook. Only factory owner or POOLS_ADMINISTRATOR_ROLE role |

### setFee

```solidity
function setFee(uint16 newFee) external
```
**Selector**: `0x8e005553`

Set new pool fee. Can be called by owner if dynamic fee is disabled.
Called by the plugin if dynamic fee is enabled

| Name | Type | Description |
| ---- | ---- | ----------- |
| newFee | uint16 | The new fee value |

## Errors
## locked

```solidity
error locked()
```
**Selector**: `0xcf309012`

Emitted by the reentrancy guard

## arithmeticError

```solidity
error arithmeticError()
```
**Selector**: `0x8995290f`

Emitted if arithmetic error occurred

## alreadyInitialized

```solidity
error alreadyInitialized()
```
**Selector**: `0x52669adc`

Emitted if an attempt is made to initialize the pool twice

## notInitialized

```solidity
error notInitialized()
```
**Selector**: `0x812eb655`

Emitted if an attempt is made to mint or swap in uninitialized pool

## zeroAmountRequired

```solidity
error zeroAmountRequired()
```
**Selector**: `0x79db9840`

Emitted if 0 is passed as amountRequired to swap function

## invalidAmountRequired

```solidity
error invalidAmountRequired()
```
**Selector**: `0x69967402`

Emitted if invalid amount is passed as amountRequired to swap function

## insufficientInputAmount

```solidity
error insufficientInputAmount()
```
**Selector**: `0xfb5b5414`

Emitted if the pool received fewer tokens than it should have

## zeroLiquidityDesired

```solidity
error zeroLiquidityDesired()
```
**Selector**: `0xe6ace6df`

Emitted if there was an attempt to mint zero liquidity

## zeroLiquidityActual

```solidity
error zeroLiquidityActual()
```
**Selector**: `0xbeba2a6c`

Emitted if actual amount of liquidity is zero (due to insufficient amount of tokens received)

## flashInsufficientPaid0

```solidity
error flashInsufficientPaid0()
```
**Selector**: `0x6dbca1fe`

Emitted if the pool received fewer tokens0 after flash than it should have

## flashInsufficientPaid1

```solidity
error flashInsufficientPaid1()
```
**Selector**: `0xc998149f`

Emitted if the pool received fewer tokens1 after flash than it should have

## invalidLimitSqrtPrice

```solidity
error invalidLimitSqrtPrice()
```
**Selector**: `0x16626723`

Emitted if limitSqrtPrice param is incorrect

## tickIsNotSpaced

```solidity
error tickIsNotSpaced()
```
**Selector**: `0x5f6e14f3`

Tick must be divisible by tickspacing

## notAllowed

```solidity
error notAllowed()
```
**Selector**: `0x932984d2`

Emitted if a method is called that is accessible only to the factory owner or dedicated role

## invalidNewTickSpacing

```solidity
error invalidNewTickSpacing()
```
**Selector**: `0xafe09f44`

Emitted if new tick spacing exceeds max allowed value

## invalidNewCommunityFee

```solidity
error invalidNewCommunityFee()
```
**Selector**: `0xa709b9af`

Emitted if new community fee exceeds max allowed value

## dynamicFeeActive

```solidity
error dynamicFeeActive()
```
**Selector**: `0xd39b8e0e`

Emitted if an attempt is made to manually change the fee value, but dynamic fee is enabled

## dynamicFeeDisabled

```solidity
error dynamicFeeDisabled()
```
**Selector**: `0x3a4528ef`

Emitted if an attempt is made by plugin to change the fee value, but dynamic fee is disabled

## pluginIsNotConnected

```solidity
error pluginIsNotConnected()
```
**Selector**: `0x9e727ce3`

Emitted if an attempt is made to change the plugin configuration, but the plugin is not connected

## invalidHookResponse

```solidity
error invalidHookResponse(bytes4 expectedSelector)
```
**Selector**: `0xd3f5153b`

Emitted if a plugin returns invalid selector after hook call

| Name | Type | Description |
| ---- | ---- | ----------- |
| expectedSelector | bytes4 | The expected selector |

## liquiditySub

```solidity
error liquiditySub()
```
**Selector**: `0x1301f748`

Emitted if liquidity underflows

## liquidityAdd

```solidity
error liquidityAdd()
```
**Selector**: `0x997402f2`

Emitted if liquidity overflows

## topTickLowerOrEqBottomTick

```solidity
error topTickLowerOrEqBottomTick()
```
**Selector**: `0xd9a841a7`

Emitted if the topTick param not greater then the bottomTick param

## bottomTickLowerThanMIN

```solidity
error bottomTickLowerThanMIN()
```
**Selector**: `0x746b1fc4`

Emitted if the bottomTick param is lower than min allowed value

## topTickAboveMAX

```solidity
error topTickAboveMAX()
```
**Selector**: `0x1445443d`

Emitted if the topTick param is greater than max allowed value

## liquidityOverflow

```solidity
error liquidityOverflow()
```
**Selector**: `0x25b8364a`

Emitted if the liquidity value associated with the tick exceeds MAX_LIQUIDITY_PER_TICK

## tickIsNotInitialized

```solidity
error tickIsNotInitialized()
```
**Selector**: `0x0d6e0949`

Emitted if an attempt is made to interact with an uninitialized tick

## tickInvalidLinks

```solidity
error tickInvalidLinks()
```
**Selector**: `0xe45ac17d`

Emitted if there is an attempt to insert a new tick into the list of ticks with incorrect indexes of the previous and next ticks

## transferFailed

```solidity
error transferFailed()
```
**Selector**: `0xe465903e`

Emitted if token transfer failed internally

## tickOutOfRange

```solidity
error tickOutOfRange()
```
**Selector**: `0x3c10250f`

Emitted if tick is greater than the maximum or less than the minimum allowed value

## priceOutOfRange

```solidity
error priceOutOfRange()
```
**Selector**: `0x55cf1e23`

Emitted if price is greater than the maximum or less than the minimum allowed value

## Events
### Initialize

```solidity
event Initialize(uint160 price, int24 tick)
```

Emitted exactly once by a pool when #initialize is first called on the pool

*Developer note: Mint/Burn/Swaps cannot be emitted by the pool before Initialize*

| Name | Type | Description |
| ---- | ---- | ----------- |
| price | uint160 | The initial sqrt price of the pool, as a Q64.96 |
| tick | int24 | The initial tick of the pool, i.e. log base 1.0001 of the starting price of the pool |

### Mint

```solidity
event Mint(address sender, address owner, int24 bottomTick, int24 topTick, uint128 liquidityAmount, uint256 amount0, uint256 amount1)
```

Emitted when liquidity is minted for a given position

| Name | Type | Description |
| ---- | ---- | ----------- |
| sender | address | The address that minted the liquidity |
| owner | address | The owner of the position and recipient of any minted liquidity |
| bottomTick | int24 | The lower tick of the position |
| topTick | int24 | The upper tick of the position |
| liquidityAmount | uint128 | The amount of liquidity minted to the position range |
| amount0 | uint256 | How much token0 was required for the minted liquidity |
| amount1 | uint256 | How much token1 was required for the minted liquidity |

### Collect

```solidity
event Collect(address owner, address recipient, int24 bottomTick, int24 topTick, uint128 amount0, uint128 amount1)
```

Emitted when fees are collected by the owner of a position

| Name | Type | Description |
| ---- | ---- | ----------- |
| owner | address | The owner of the position for which fees are collected |
| recipient | address | The address that received fees |
| bottomTick | int24 | The lower tick of the position |
| topTick | int24 | The upper tick of the position |
| amount0 | uint128 | The amount of token0 fees collected |
| amount1 | uint128 | The amount of token1 fees collected |

### Burn

```solidity
event Burn(address owner, int24 bottomTick, int24 topTick, uint128 liquidityAmount, uint256 amount0, uint256 amount1)
```

Emitted when a position&#x27;s liquidity is removed

*Developer note: Does not withdraw any fees earned by the liquidity position, which must be withdrawn via [#collect](#collect)*

| Name | Type | Description |
| ---- | ---- | ----------- |
| owner | address | The owner of the position for which liquidity is removed |
| bottomTick | int24 | The lower tick of the position |
| topTick | int24 | The upper tick of the position |
| liquidityAmount | uint128 | The amount of liquidity to remove |
| amount0 | uint256 | The amount of token0 withdrawn |
| amount1 | uint256 | The amount of token1 withdrawn |

### Swap

```solidity
event Swap(address sender, address recipient, int256 amount0, int256 amount1, uint160 price, uint128 liquidity, int24 tick)
```

Emitted by the pool for any swaps between token0 and token1

| Name | Type | Description |
| ---- | ---- | ----------- |
| sender | address | The address that initiated the swap call, and that received the callback |
| recipient | address | The address that received the output of the swap |
| amount0 | int256 | The delta of the token0 balance of the pool |
| amount1 | int256 | The delta of the token1 balance of the pool |
| price | uint160 | The sqrt(price) of the pool after the swap, as a Q64.96 |
| liquidity | uint128 | The liquidity of the pool after the swap |
| tick | int24 | The log base 1.0001 of price of the pool after the swap |

### Flash

```solidity
event Flash(address sender, address recipient, uint256 amount0, uint256 amount1, uint256 paid0, uint256 paid1)
```

Emitted by the pool for any flashes of token0/token1

| Name | Type | Description |
| ---- | ---- | ----------- |
| sender | address | The address that initiated the swap call, and that received the callback |
| recipient | address | The address that received the tokens from flash |
| amount0 | uint256 | The amount of token0 that was flashed |
| amount1 | uint256 | The amount of token1 that was flashed |
| paid0 | uint256 | The amount of token0 paid for the flash, which can exceed the amount0 plus the fee |
| paid1 | uint256 | The amount of token1 paid for the flash, which can exceed the amount1 plus the fee |

### CommunityFee

```solidity
event CommunityFee(uint16 communityFeeNew)
```

Emitted when the community fee is changed by the pool

| Name | Type | Description |
| ---- | ---- | ----------- |
| communityFeeNew | uint16 | The updated value of the community fee in thousandths (1e-3) |

### TickSpacing

```solidity
event TickSpacing(int24 newTickSpacing)
```

Emitted when the tick spacing changes

| Name | Type | Description |
| ---- | ---- | ----------- |
| newTickSpacing | int24 | The updated value of the new tick spacing |

### Plugin

```solidity
event Plugin(address newPluginAddress)
```

Emitted when the plugin address changes

| Name | Type | Description |
| ---- | ---- | ----------- |
| newPluginAddress | address | New plugin address |

### PluginConfig

```solidity
event PluginConfig(uint8 newPluginConfig)
```

Emitted when the plugin config changes

| Name | Type | Description |
| ---- | ---- | ----------- |
| newPluginConfig | uint8 | New plugin config |

### Fee

```solidity
event Fee(uint16 fee)
```

Emitted when the fee changes inside the pool

| Name | Type | Description |
| ---- | ---- | ----------- |
| fee | uint16 | The current fee in hundredths of a bip, i.e. 1e-6 |