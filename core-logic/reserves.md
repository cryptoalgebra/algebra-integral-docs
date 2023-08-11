# Reserves

Relevant and important files:

* base/ReservesManager.sol
* AlgebraPool.sol

### Reserves definition

A reserves mechanism was introduced to track increases in pool balances. Reserves reflect the last known pool balances - after the last major pool action has been performed (`mint`/`burn`/`swap`/`collect`/`flash`). Immediately after interaction with the pool, under normal conditions, reserves should have the following values:

$$reserveToken0 = balanceToken0$$

$$reserveToken1 = balanceToken1$$

Synchronization of reserves and balances after interaction with the pool occurs based on the expected change in balances - using the number of tokens sent or requested by the pool. For example, sending tokens to a user ($$tokenAmount^{\{0, 1\}}$$) causes the next change in the corresponding reserve:

$$reserveToken^{\{0,1\}}_{new} = reserveToken^{\{0,1\}}_{old}  - tokenAmount^{\{0,1\}}$$

On the other hand, when $$tokenAmount^{\{0, 1\}}$$ is received, the reserve is increased accordingly:

$$reserveToken^{\{0,1\}}_{new} = reserveToken^{\{0,1\}}_{old}  + tokenAmount^{\{0,1\}}$$

Thus, the value of the reserve at any given moment reflects the number of tokens the pool **expects** to have on its balance.



### Reserves synchronization

At the beginning of the main pool interactions, it is checked whether the pool balances match the expected reserve values. In case of discrepancy, the corresponding token surplus (tokenDelta) is calculated and distributed among active liquidity positions as additional fees if the current liquidity $$L$$ is not zero:

$$token0Delta = balanceToken0 - reserveToken0$$

$$token1Delta = balanceToken1 - reserveToken1$$



$$totalFeeGrowth0_{new} = totalFeeGrowth_{old} + token0Delta / L$$

$$totalFeeGrowth1_{new} = totalFeeGrowth_{old} + token1Delta / L$$



If current liquidity is zero ($$L = 0$$), no synchronization occurs until liquidity becomes non-zero.

Thanks to this mechanism it became possible to support token rebase by the Algebra protocol - the pool is able to detect an increase in token balances due to external reasons and distribute the excess tokens among active liquidity providers.

In addition, tokens directly sent to the pool's contract for any reason will not be permanently blocked in the pool, and are distributed among active liquidity positions.

Another advantage of the reserve mechanism, is that it provides the ability to flashloan regardless of current liquidity - it is possible to use flashloan even if the current liquidity in the pool is zero, but there is some amount of tokens on the balance.



### Limits

The reserve value for each of the tokens is limited to the maximum value of data type `uint128`: $$2^{128} - 1$$. This restriction allows us to minimize the cost of gas when interacting with the pool.

However, this restriction means that Algebra Integral does not guarantee full pooling performance when using tokens with totalSupply greater than the maximum value of `uint128`. For most pool interactions, if the pool interaction results in the reserve exceeding the maximum allowed value, the transaction will be reverted.

In case the maximum allowed reserve is exceeded for external reasons (sending tokens directly to the pool or overpaying in a flashloan), all excess tokens will be sent to AlgebraCommunityVault as communityFee.



### Pending communityFee

To minimize the number of unnecessary transfers, communityFee is not sent to the AlgebraCommunityVault at each swap. Instead, the accumulated part of the commissions is recorded in the variables `communityFeePending{0,1}`.&#x20;

The communityFee is sent at a set frequency (`COMMUNITY_FEE_TRANSFER_FREQUENCY`), as well as in case of overflow of values `communityFeePending{0,1}`.

Accumulation and sending of communityFee is taken into account in tracking pool reserves.
