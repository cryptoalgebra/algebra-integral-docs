# Flashloans

This guide is fully dedicated to the process of writing a smart contract that calls `flash` on a pool and swaps the full amount withdrawn of `token0` and `token1` in the corresponding pools with the same pair of tokens. After the swap is complete, the contract will pay back the pool and transfer profits to the original calling address.