# Algebra Factory

Algebra factory

Is used to deploy pools and its plugins

_Version: Algebra Integral_

**Inherits:** [IAlgebraFactory](../contracts-api/Core/interfaces/IAlgebraFactory.md) [Ownable2Step](https://docs.openzeppelin.com/contracts/4.x/) [AccessControlEnumerable](https://docs.openzeppelin.com/contracts/4.x/)

## Public variables

### POOLS\_ADMINISTRATOR\_ROLE

```solidity
bytes32 constant POOLS_ADMINISTRATOR_ROLE = 0xb73ce166ead2f8e9add217713a7989e4edfba9625f71dfd2516204bb67ad3442
```

**Selector**: `0xb500a48b`

role that can change communityFee and tickspacing in pools

### poolDeployer

```solidity
address immutable poolDeployer
```

**Selector**: `0x3119049a`

Returns the current poolDeployerAddress

### communityVault

```solidity
address immutable communityVault
```

**Selector**: `0x53e97868`

Returns the current communityVaultAddress

### defaultCommunityFee

```solidity
uint16 defaultCommunityFee
```

**Selector**: `0x2f8a39dd`

Returns the default community fee

### defaultFee

```solidity
uint16 defaultFee
```

**Selector**: `0x5a6c72d0`

Returns the default fee

### defaultTickspacing

```solidity
int24 defaultTickspacing
```

**Selector**: `0x29bc3446`

Returns the default tickspacing

### renounceOwnershipStartTimestamp

```solidity
uint256 renounceOwnershipStartTimestamp
```

**Selector**: `0x084bfff9`

### defaultPluginFactory

```solidity
contract IAlgebraPluginFactory defaultPluginFactory
```

**Selector**: `0xd0ad2792`

Return the current pluginFactory address

### poolByPair

```solidity
mapping(address => mapping(address => address)) poolByPair
```

**Selector**: `0xd9a641e1`

Returns the pool address for a given pair of tokens, or address 0 if it does not exist

_Developer note: tokenA and tokenB may be passed in either token0/token1 or token1/token0 order_

### POOL\_INIT\_CODE\_HASH

```solidity
bytes32 constant POOL_INIT_CODE_HASH = 0x177d5fbf994f4d130c008797563306f1a168dc689f81b2fa23b4396931014d91
```

**Selector**: `0xdc6fd8ab`

returns keccak256 of AlgebraPool init bytecode.

_Developer note: keccak256 of AlgebraPool init bytecode. Used to compute pool address deterministically_

## Functions

### constructor

```solidity
constructor(address _poolDeployer) public
```

| Name           | Type    | Description |
| -------------- | ------- | ----------- |
| \_poolDeployer | address |             |

### owner

```solidity
function owner() public view returns (address)
```

**Selector**: `0x8da5cb5b`

Returns the current owner of the factory

_Developer note: Can be changed by the current owner via transferOwnership(address newOwner)_

**Returns:**

| Name | Type    | Description                      |
| ---- | ------- | -------------------------------- |
| \[0] | address | The address of the factory owner |

### hasRoleOrOwner

```solidity
function hasRoleOrOwner(bytes32 role, address account) public view returns (bool)
```

**Selector**: `0xe8ae2b69`

Returns \`true\` if \`account\` has been granted \`role\` or \`account\` is owner.

| Name    | Type    | Description                               |
| ------- | ------- | ----------------------------------------- |
| role    | bytes32 | The hash corresponding to the role        |
| account | address | The address for which the role is checked |

**Returns:**

| Name | Type | Description                                                     |
| ---- | ---- | --------------------------------------------------------------- |
| \[0] | bool | bool Whether the address has this role or the owner role or not |

### defaultConfigurationForPool

```solidity
function defaultConfigurationForPool() external view returns (uint16 communityFee, int24 tickSpacing, uint16 fee)
```

**Selector**: `0x25b355d6`

Returns the default communityFee and tickspacing

**Returns:**

| Name         | Type   | Description                                   |
| ------------ | ------ | --------------------------------------------- |
| communityFee | uint16 | which will be set at the creation of the pool |
| tickSpacing  | int24  | which will be set at the creation of the pool |
| fee          | uint16 | which will be set at the creation of the pool |

### computePoolAddress

```solidity
function computePoolAddress(address token0, address token1) public view returns (address pool)
```

**Selector**: `0xd8ed2241`

Deterministically computes the pool address given the token0 and token1

_Developer note: The method does not check if such a pool has been created_

| Name   | Type    | Description  |
| ------ | ------- | ------------ |
| token0 | address | first token  |
| token1 | address | second token |

**Returns:**

| Name | Type    | Description                              |
| ---- | ------- | ---------------------------------------- |
| pool | address | The contract address of the Algebra pool |

### createPool

```solidity
function createPool(address tokenA, address tokenB) external returns (address pool)
```

**Selector**: `0xe3433615`

Creates a pool for the given two tokens

_Developer note: tokenA and tokenB may be passed in either order: token0/token1 or token1/token0. The call will revert if the pool already exists or the token arguments are invalid._

| Name   | Type    | Description                                     |
| ------ | ------- | ----------------------------------------------- |
| tokenA | address | One of the two tokens in the desired pool       |
| tokenB | address | The other of the two tokens in the desired pool |

**Returns:**

| Name | Type    | Description                           |
| ---- | ------- | ------------------------------------- |
| pool | address | The address of the newly created pool |

### setDefaultCommunityFee

```solidity
function setDefaultCommunityFee(uint16 newDefaultCommunityFee) external
```

**Selector**: `0x8d5a8711`

_Developer note: updates default community fee for new pools_

| Name                   | Type   | Description                                             |
| ---------------------- | ------ | ------------------------------------------------------- |
| newDefaultCommunityFee | uint16 | The new community fee, _must_ be <= MAX\_COMMUNITY\_FEE |

### setDefaultFee

```solidity
function setDefaultFee(uint16 newDefaultFee) external
```

**Selector**: `0x77326584`

_Developer note: updates default fee for new pools_

| Name          | Type   | Description                                 |
| ------------- | ------ | ------------------------------------------- |
| newDefaultFee | uint16 | The new fee, _must_ be <= MAX\_DEFAULT\_FEE |

### setDefaultTickspacing

```solidity
function setDefaultTickspacing(int24 newDefaultTickspacing) external
```

**Selector**: `0xf09489ac`

_Developer note: updates default tickspacing for new pools_

| Name                  | Type  | Description                                                                    |
| --------------------- | ----- | ------------------------------------------------------------------------------ |
| newDefaultTickspacing | int24 | The new tickspacing, _must_ be <= MAX\_TICK\_SPACING and >= MIN\_TICK\_SPACING |

### setDefaultPluginFactory

```solidity
function setDefaultPluginFactory(address newDefaultPluginFactory) external
```

**Selector**: `0x2939dd97`

_Developer note: updates pluginFactory address_

| Name                    | Type    | Description                   |
| ----------------------- | ------- | ----------------------------- |
| newDefaultPluginFactory | address | address of new plugin factory |

### startRenounceOwnership

```solidity
function startRenounceOwnership() external
```

**Selector**: `0x469388c4`

Starts process of renounceOwnership. After that, a certain period of time must pass before the ownership renounce can be completed.

### stopRenounceOwnership

```solidity
function stopRenounceOwnership() external
```

**Selector**: `0x238a1d74`

Stops process of renounceOwnership and removes timer.

### renounceOwnership

```solidity
function renounceOwnership() public
```

**Selector**: `0x715018a6`

_Developer note: Leaves the contract without owner. It will not be possible to call \`onlyOwner\` functions anymore. Can only be called by the current owner if RENOUNCE\_OWNERSHIP\_DELAY seconds have passed since the call to the startRenounceOwnership() function._

## Events

### RenounceOwnershipStart

```solidity
event RenounceOwnershipStart(uint256 timestamp, uint256 finishTimestamp)
```

Emitted when a process of ownership renounce is started

| Name            | Type    | Description                                                      |
| --------------- | ------- | ---------------------------------------------------------------- |
| timestamp       | uint256 | The timestamp of event                                           |
| finishTimestamp | uint256 | The timestamp when ownership renounce will be possible to finish |

### RenounceOwnershipStop

```solidity
event RenounceOwnershipStop(uint256 timestamp)
```

Emitted when a process of ownership renounce cancelled

| Name      | Type    | Description            |
| --------- | ------- | ---------------------- |
| timestamp | uint256 | The timestamp of event |

### RenounceOwnershipFinish

```solidity
event RenounceOwnershipFinish(uint256 timestamp)
```

Emitted when a process of ownership renounce finished

| Name      | Type    | Description                             |
| --------- | ------- | --------------------------------------- |
| timestamp | uint256 | The timestamp of ownership renouncement |

### Pool

```solidity
event Pool(address token0, address token1, address pool)
```

Emitted when a pool is created

| Name   | Type    | Description                                        |
| ------ | ------- | -------------------------------------------------- |
| token0 | address | The first token of the pool by address sort order  |
| token1 | address | The second token of the pool by address sort order |
| pool   | address | The address of the created pool                    |

### DefaultCommunityFee

```solidity
event DefaultCommunityFee(uint16 newDefaultCommunityFee)
```

Emitted when the default community fee is changed

| Name                   | Type   | Description                         |
| ---------------------- | ------ | ----------------------------------- |
| newDefaultCommunityFee | uint16 | The new default community fee value |

### DefaultTickspacing

```solidity
event DefaultTickspacing(int24 newDefaultTickspacing)
```

Emitted when the default tickspacing is changed

| Name                  | Type  | Description                       |
| --------------------- | ----- | --------------------------------- |
| newDefaultTickspacing | int24 | The new default tickspacing value |

### DefaultFee

```solidity
event DefaultFee(uint16 newDefaultFee)
```

Emitted when the default fee is changed

| Name          | Type   | Description               |
| ------------- | ------ | ------------------------- |
| newDefaultFee | uint16 | The new default fee value |

### DefaultPluginFactory

```solidity
event DefaultPluginFactory(address defaultPluginFactoryAddress)
```

Emitted when the defaultPluginFactory address is changed

| Name                        | Type    | Description                          |
| --------------------------- | ------- | ------------------------------------ |
| defaultPluginFactoryAddress | address | The new defaultPluginFactory address |
