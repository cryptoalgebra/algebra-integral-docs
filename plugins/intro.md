# Intro

## Overview

Plugins for Algebra Integral are smart contracts that can expand or limit the functionality of liquidity pools. The plugin can be connected to a liquidity pool, then the pool will be able to call the corresponding plugin methods before and after the main actions in the pool:

* swap
* mint
* burn
* flash

A more detailed general description of plugins can be found in the section: [Core logic - plugins](../core-logic/plugins.md)

## Template repository

To simplify the creation of new plugins, you can use a special template: [algebra-plugin-template](https://github.com/cryptoalgebra/algebra-plugin-template)

### Installation

Clone repository:

```bash
git clone https://github.com/cryptoalgebra/algebra-plugin-template --recursive
cd algebra-plugin-template
```

Install dependencies:

```bash
npm i
```

Now you can use the template to create new plugins.
