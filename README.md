>yarn install
>yarn prepublishOnly

That's it. Optismism Rollup UniswapV2-core example yarn test:ovm

if issue:
-check package.json Project works better using Volta. Might need to install 
-On most Unix systems, you can install Volta with a single command:

>curl https://get.volta.sh | bash

---------------------------------------------------------------------------------------------------------------------------------------------
Original document supporting this repo https://hackmd.io/@scopelift/Hy853dTsP#Porting-Solidity-Contracts-to-Optimism-A-Guide-Using-Uniswap-V2
---------------------------------------------------------------------------------------------------------------------------------------------

Porting Solidity Contracts to Optimism: A Guide Using Uniswap V2
This guide will walk through the process of porting an existing Solidity project to support execution on the Optimism Layer 2 network. For our example codebase, we’ll use the Uniswap V2 decentralized exchange.

We’ll go through the process of modifying the Uniswap repository so it can support deployment on the Optimistic Virtual Machine (OVM) and the Ethereum Virtual Machine (EVM).

At the end of this guide, you’ll have a test suite that runs on both the EVM and the OVM! You’ll also have a good idea of what it takes to get your Solidity project up and running on the OVM.

Porting Solidity Contracts to Optimism: A Guide Using Uniswap V2
Background
Prerequisites
Optimistic Rollups
The Porting Process
Getting Started
Package Upgrades
EVM Test Suite Updates
Optimism Setup
Dependencies
Compiling for the OVM
OVM vs. EVM: Constructor Arguments
OVM vs. EVM: Block Timestamps
Trade Deadlines, permit signatures, and auctions
Price Oracle
Testing on the OVM
Provider and Chain ID Fixes
Gas and Compiler Fixes
OVM vs. EVM: Gas
OVM vs. EVM: Compiling
Final Gas Tweaks
Going Forward
OVM vs. EVM: Contract Wallets
OVM vs. EVM: Gas, Part II
Wrapping Up
Background
Prerequisites
This guide is targeted at existing smart contract developers. It assumes a basic knowledge of Solidity development and the tools commonly used along with it. This includes writing our tests in JavaScript and using yarn for package management.

If you’re brand new to smart contract development, we recommend checking out some of the great resources for getting started, including the Solidity docs themselves. The good news is, almost all of your new knowledge will eventually translate directly to building contracts for the OVM.

Optimistic Rollups
At a very high level, the OVM is an optimistic implementation of the EVM. Transactions are executed on the OVM, which enables full EVM support, and the resulting state transitions are optimistically assumed to be valid. If someone does not believe a state transition is valid, they can submit a fraud proof to be verified by the EVM. As a result, the EVM only needs to execute computations when there is a dispute about a transaction’s legitimacy.

If you are unfamiliar with Optimistic Rollups and the OVM, these resources can help you learn more:
-----------------------------------------------------------------------------------
Optimistic Virtual Machine Alpha: A high level introduction and overview of the OVM
https://medium.com/ethereum-optimism/optimistic-virtual-machine-alpha-cdf51f5d49e
-----------------------------------------------------------------------------------
Optimism: Keeping Ethereum Half-Full: Video introduction with some more depth
https://www.youtube.com/watch?v=eYeOW4ePgZE
-----------------------------------------------------------------------------------
OVM Deep Dive: An in-depth article covering the challenges faced and solutions used by the OVM
https://medium.com/ethereum-optimism/ovm-deep-dive-a300d1085f52
...............................................................
---------------------------------------------------------------------------------------------------------------------------------------------
Original document supporting this repo https://hackmd.io/@scopelift/Hy853dTsP#Porting-Solidity-Contracts-to-Optimism-A-Guide-Using-Uniswap-V2
---------------------------------------------------------------------------------------------------------------------------------------------


