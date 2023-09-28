# Intro

## Intro

In this step-by-step guide, we'll try to provide all the necessary pieces of information, hints, and tools for you to create a whole development environment, that you can further use to build the rest of the examples in the Guides section of the docs, or start your own project with our integration!

In order not to waste time and to provide the most comfortable structure to all readers, we have created two different options. You can find the Quick Start section below, in which you can clone a boilerplate and start building on your own, without diving deep into the base, or you can start from scratch and learn all the underlying concepts, by clicking on the Start from Scratch section. Both sections are ok as long as you know exactly what are your goals at this point. Let’s get started!

## Quick Start

[The Algebra boilerplate repo](https://github.com/cryptoalgebra/boilerplate) provides those who want to build with our codebase with a basic Hardhat environment with required imports already preloaded for you. You can simply clone it and install the dependencies:

```bash
git clone https://github.com/cryptoalgebra/boilerplate
cd boilerplate
npm install 
```

Then go to the `Local Node with a Polygon Fork` to complete your set-up and start building.

## Start from Scratch

Node is considered to be one of the most common Javascript execution mechanisms. That means that for our goals, it will provide scripting that we can further use to compile and test our contracts. If you haven’t already done so, install NodeJS and its package manager NPM before the start.

### Set Up Dependencies

Node is one of the most common Javascript runtimes. For our purposes it will provide scripting we can use to compile and test our contracts. If you haven’t already, install NodeJS and its package manager NPM.

[Instructions](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)

Once those dependencies are set up, we can initialize our project:

```bash
$ npm init
```

[Hardhat](https://hardhat.org/)

This is the Ethereum's development toolset – a vital development environment, providing quite a lot of features, such as Solidity compilation, testing and deployment. All of them are contained in a single convenient wrapper, making it easy to use. We’ll use NPM to add Hardhat to our project:

```bash
$ npm add --save-dev hardhat
```

By installing Hardhat, we can bring it up to create a development environment. When you first run Hardhat, you’ll have three options of running a project: start with a templated Javascript; start with a Typescript project, or an empty project. Taking into account that Hardhat relies heavily on folder structure, we strongly recommend you to start with any of the templated options. Initialize Hardhat, follow every prompt to make your selection, and answer yes to the follow-up prompts:

```bash
$ npx hardhat init
```

After Hardhat has finished initializing, take a close look at what has been installed. The folder structure should be intuitive and easy to navigate through. Keep in mind that ./contracts is where you’ll write your Solidity contracts, ./test is where you’ll write your tests and ./scripts is where you can write scripts to perform actions like deploying. Remember! Hardhat is designed to use this exact folder structure, so make sure you don’t change anything unless you know what you’re doing.

For the next step, we’ll use NPM to add the Algebra contracts, which will allow us to seamlessly integrate with the protocol in our new contracts:

```bash
$ npm add @cryptoalgebra/integral-periphery @cryptoalgebra/integral-core
```

The Algebra contracts were written using a past version of the solidity compiler. Since we’re building integrations using the Algebra codebase, we have to make Hardhat to use the correct compiler to build these files. Go to the `./hardhat.config.js` file and change the Solidity version to “0.8.20”:

```jsx
// ...
module.exports = {
  solidity: "0.8.20",
};
```

Now, we’re done! After all these steps, you should have a functional development environment, allowing you to start building on-chain Algebra integrations. For now, let’s run a quick test to make sure everything is set up just as it should be.

### Creating a Main Contract

To make sure that our environment is set up correctly, we’ll try to compile a basic Swap. For that purpose, create a new file, `./contracts/Swap.sol` and paste the following code into it.

[Single Swaps Contract Guide](swaps/single-swaps.md)

```jsx
// SPDX-License-Identifier: GPL-2.0-or-later
pragma solidity =0.8.20;

import '@cryptoalgebra/integral-periphery/contracts/interfaces/ISwapRouter.sol';
import '@cryptoalgebra/integral-periphery/contracts/libraries/TransferHelper.sol';

contract SimpleSwap {
    ISwapRouter public immutable swapRouter;
    address public constant DAI = 0x8f3Cf7ad23Cd3CaDbD9735AFf958023239c6A063;
    address public constant WMATIC = 0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270;
    
    constructor(ISwapRouter _swapRouter) {
        swapRouter = _swapRouter;
    }
    
    function swapMATICForDAI(uint256 amountIn) external returns (uint256 amountOut) {

        // Transfer the specified amount of MATIC to this contract.
        TransferHelper.safeTransferFrom(WMATIC, msg.sender, address(this), amountIn);
        // Approve the router to spend MATIC.
        TransferHelper.safeApprove(WMATIC, address(swapRouter), amountIn);
        // Note: To use this example, you should explicitly set slippage limits, omitting for simplicity
        const minOut = /* Calculate min output */ 0; 
        const priceLimit = /* Calculate price limit */ 0; 
        // Create the params that will be used to execute the swap
        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
                tokenIn: WMATIC,
                tokenOut: DAI,
                recipient: msg.sender,
                deadline: block.timestamp,
                amountIn: amountIn,
                amountOutMinimum: minOut,
                sqrtPriceLimitX96: priceLimit
            });
        // The call to `exactInputSingle` executes the swap.
        amountOut = swapRouter.exactInputSingle(params);
    }
}
```

In order to compile all the contracts in the `./contracts` folder, we will have to use the Hardhat compile command:

```bash
$ npx hardhat compile
```

If the environment is compiled properly, you’ll see the message pop up:

```bash
Compiled { x } Solidity files successfully
```

## Local Node with a Polygon Fork

In the processes of building and testing integrations with on-chain protocols, developers might face a major issue: liquidity on the live chain is critical to thoroughly testing their code, but testing against a live network like Polygon can be extremely expensive.

Luckily, Hardhat has a great feature that allows developers to run a local Polygon test node that uses a fork of Polygon. This way, it is possible for us to test against simulated liquidity for free.

As a prerequisite, we’ll need an RPC that supports Forking. Alchemy includes forking in its free tier, so it’s a great place to start.

[Sign up and get an Alchemy API key](https://www.alchemy.com/)

After that, you're free to run the following Hardhat command to start your node:

```bash
$ npx hardhat node --fork https://polygon-mainnet.g.alchemy.com/v2{YOUR_API_KEY}
```

With your local node is running, you are able to use the `--network localhost` flag in tests to direct the Hardhat testing suite to that local host:

```bash
$ npx hardhat test --network localhost
```

## Next Steps

Once your development environment is set up, you’re all ready to start building. Enter the Guides section to learn more about the Algebra functions that you can integrate with. Don’t forget to add all contracts (.sol files) to the `./contracts` folder and their subsequent tests to the `./tests` folder. You can then test them against your local forked node by running:

```bash
$ npx hardhat test --network localhost
```
