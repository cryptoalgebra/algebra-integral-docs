# AlgebraFactory and roles

The AlgebraFactory contract is the central contract in the Algebra Integral protocol and is responsible for creating new liquidity pools and managing access to sensitive actions.

### Pool creation

With Algebra Factory, any user can create a pool for a token pair. Only one liquidity pool can exist for each token pair.

When created, the pool will have default values for a number of settings (default values can be changed in Algebra Factory).

If a plugin factory is connected to Algebra Factory, the corresponding plugin will be created and connected for the new pool.

**It is important to note** that Algebra Factory is not a direct pool creator (important for calculating addresses using `create2`). Algebra Factory creates pools using a special smart contract called AlgebraPoolDeployer.

### Access control and role mechanism

Algebra Factory uses OpenZeppelin's implementation of the role mechanism (`AccessControlEnumerable`). Because of this, Algebra Factory can be used as a role manager for various smart contracts in the protocol.

It is important to note that for most secure owner actions Algebra Factory is superuser and has access rights. For security reasons, it is recommended to use a more granular role policy for different tasks and pass the title owner of Algebra Factory to a multisig / DAO / special smart contract.

Standard roles in the Core part of the Algebra Integral protocol:

* `POOLS_ADMINISTRATOR`  - the owner of this role can change parameters in pools. This includes the right to change plugins and plugin settings in the pool.
* `COMMUNITY_FEE_WITHDRAWER` - the holder of this role can retrieve accumulated communityFee from the AlgebraCommunityVault.&#x20;

The `AccessControlEnumerable` mechanism allows you to create new roles and use them for protocol needs.
