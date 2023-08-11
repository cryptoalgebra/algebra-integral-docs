# Flash

Relevant and important files:

* AlgebraPool.sol

**Flashloans** are a special type of borrowing that allows users to borrow tokens in arbitrary amounts to be returned along with fees in a single transaction, without having to provide any collateral or guarantees.

Algebra Integral allows borrowing tokens from pools - a user can request a number of tokens from a pool up to the pool's balance. Flushloan fees will be distributed to active liquidity positions. In addition, any excess tokens that were transferred in the callback above the requested amount will also be distributed to active liquidity providers as a fee. If there is currently no active liquidity, the collected commission will go to the first liquidity positions that become active in the future.



**It is important to note** that not only active liquidity is available to receive flashloan, but the entire token balance available to the pool.

<mark style="color:red;">Flashloan utilizes a callback mechanism. When creating your own implementation of a contract that uses flashloan Algebra Integral, you should check that the contract causing the callback is actually the expected Algebra Integral pool.</mark>

The functionality of flashloan (`flash`) can be extended by [plugins](plugins.md) that use appropriate hooks: `beforeFlash` and `afterFlash`.
