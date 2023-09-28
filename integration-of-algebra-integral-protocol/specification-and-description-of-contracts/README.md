# Specification and API of contracts

A conceptual, higher-level description of the Core part logic can be found in [the corresponding section of this documentation](broken-reference).

This subsection contains a condensed API specification of the most important protocol contracts. A more detailed API specification, including all contracts, is located in the protocol repository: [the Algebra Integral protocol repository](https://github.com/cryptoalgebra/Algebra/tree/dev/docs/Contracts/Core). This specification is generated automatically using natspec annotations of functions and state variables of smart contracts.

* [algebrapool.md](algebrapool.md "mention") - liquidity pool of Algebra Integral
* [algebrafactory.md](algebrafactory.md "mention") - factory of liquidity pools (and access manager)
* [swaprouter.md](swaprouter.md "mention") - default peripheral router for swaps
* [nonfungiblepositionmanager.md](nonfungiblepositionmanager.md "mention") - default position management contract (with NFT)
* [quoter.md](quoter.md "mention") - allows getting the expected amount out or amount in for a given swap without executing the swap (also off-chain)
* [quoterv2.md](quoterv2.md "mention") - version of Quoter returning more data
* [ticklens.md](ticklens.md "mention") - a contract that allows conveniently fetch information about ticks in the pool

Information about custom errors that the pool can throw:

[#errors](algebrapool.md#errors "mention")
