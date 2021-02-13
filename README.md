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

Optimistic Virtual Machine Alpha: A high level introduction and overview of the OVM
Optimism: Keeping Ethereum Half-Full: Video introduction with some more depth
OVM Deep Dive: An in-depth article covering the challenges faced and solutions used by the OVM
The Porting Process
Most existing Solidity projects will have three categories of changes required to get things running on the OVM.

Tooling updates - The OVM currently works with the Waffle v3 testing framework. If you’re using an older version of Waffle, you’ll need to upgrade. If you’re using a different test framework, you may need to migrate, though future versions of the OVM may support other frameworks in addition to Waffle. In this guide, we’ll upgrade Uniswap from Waffle v2 to Waffle v3.

Test suite updates - In addition to updating tooling, some tests themselves will need to be modified to account for minor differences in the EVM and the OVM. These include considerations such as gas differences and chain identifiers, which we’ll touch on in this guide. Our test harness will also have to support running on a local OVM node.

Contract and compiler modifications - Some differences between the EVM and the OVM precipitate changes to the Solidity contracts themselves or the compiler settings. In general, though, most Solidity written for the EVM should “just work” for the OVM. While we’ll touch on some of the cases that might require contract changes, we’ll find that the Uniswap V2 contracts will compile and run unmodified.

Getting Started
Start by cloning the Uniswap V2 Core repo. Install dependencies with yarn and run tests with yarn test to make sure all tests pass.

NOTE: This repo currently has a dependency resolution problem when installing with a lockfile. If a yarn.lock file exists, it is recommended to delete it before running yarn.

Package Upgrades
Because we want to support running tests on the EVM and OVM, let’s setup the tasks for these two test suites with the following changes to package.json:

- "test": "mocha",
- "prepublishOnly": "yarn test"
+ "prepublishOnly": "yarn test",
+ "test:evm": "yarn compile && mocha",
+ "test:ovm": "yarn compile && echo OVM TESTS NOT IMPLEMENTED"
We’ll also need to update a few dependencies. Uniswap uses Waffle v2 for their test suite, but the Optimism tooling requires Waffle v3. So let’s make one more change to package.json:

- "ethereum-waffle": "^2.4.1",
+ "ethereum-waffle": "^3.2.1",
Run yarn to update our dependencies, and run the new yarn test:evm command to run the standard EVM test suite. It will now fail with many errors, due to breaking changes introduced with the Waffle upgrade.

In the next section, we’ll resolve the errors caused by these dependency upgrades, including our jump from v4 to v5 of the ethers JavaScript library.

EVM Test Suite Updates
To resolve the breaking changes introduced by upgrading our test suite, make the changes detailed below. Note that this section is unrelated to the core changes required to deploy and test Uniswap on the OVM, but is still required.

For more information on the breaking changes between Waffle v2 and v3, and between ethers v4 and v5, see the Waffle and ethers migration guides.

In test/UniswapV2ERC20.spec.ts:

- import { MaxUint256 } from 'ethers/constants'
+ import { MaxUint256 } from '@ethersproject/constants'
- import { bigNumberify, hexlify, keccak256, defaultAbiCoder, toUtf8Bytes } from 'ethers/utils'
+ import { BigNumber } from '@ethersproject/bignumber'
+ import { defaultAbiCoder } from '@ethersproject/abi'
+ import { hexlify } from '@ethersproject/bytes'
+ import { keccak256 } from '@ethersproject/keccak256'
+ import { toUtf8Bytes } from '@ethersproject/strings'

const provider = new MockProvider({
+ ganacheOptions: {
    hardfork: 'istanbul',
    mnemonic: 'horn horn horn horn horn horn horn horn horn horn horn horn',
    gasLimit: 9999999
+ }    
})
  
- expect(await token.nonces(wallet.address)).to.eq(bigNumberify(1))
+ expect(await token.nonces(wallet.address)).to.eq(BigNumber.from(1))
In test/shared/utilities.ts:

- import { Web3Provider } from 'ethers/providers'
+ import { Web3Provider } from '@ethersproject/providers'

- import { BigNumber, bigNumberify, getAddress, keccak256, defaultAbiCoder, toUtf8Bytes, solidityPack } from 'ethers/utils'
+ import { BigNumber } from '@ethersproject/bignumber'
+ import { defaultAbiCoder } from '@ethersproject/abi'
+ import { getAddress } from '@ethersproject/address';
+ import { keccak256 } from '@ethersproject/keccak256'
+ import { pack as solidityPack } from '@ethersproject/solidity'
+ import { toUtf8Bytes } from '@ethersproject/strings'

export function expandTo18Decimals(n: number): BigNumber {
-  return bigNumberify(n).mul(bigNumberify(10).pow(18))
+  return BigNumber.from(n).mul(BigNumber.from(10).pow(18))
}

- ;(provider._web3Provider.sendAsync as any)(
+ ;(provider.provider.sendAsync as any)(

export function encodePrice(reserve0: BigNumber, reserve1: BigNumber) {
-  return [reserve1.mul(bigNumberify(2).pow(112)).div(reserve0), reserve0.mul(bigNumberify(2).pow(112)).div(reserve1)]
+  return [reserve1.mul(BigNumber.from(2).pow(112)).div(reserve0), reserve0.mul(BigNumber.from(2).pow(112)).div(reserve1)]
}
In test/UniswapV2Factory.spec.ts:

- import { AddressZero } from 'ethers/constants'
+ import { AddressZero } from '@ethersproject/constants'
- import { bigNumberify } from 'ethers/utils'
+ import { BigNumber } from '@ethersproject/bignumber';

const provider = new MockProvider({
+ ganacheOptions: {
    hardfork: 'istanbul',
    mnemonic: 'horn horn horn horn horn horn horn horn horn horn horn horn',
    gasLimit: 9999999
+ }    
})

- const loadFixture = createFixtureLoader(provider, [wallet, other])
+ const loadFixture = createFixtureLoader([wallet, other], provider)

await expect(factory.createPair(...tokens))
      .to.emit(factory, 'PairCreated')
-     .withArgs(TEST_ADDRESSES[0], TEST_ADDRESSES[1], create2Address, bigNumberify(1))
+     .withArgs(TEST_ADDRESSES[0], TEST_ADDRESSES[1], create2Address, BigNumber.from(1))
In test/shared/fixtures.ts:

- import { Web3Provider } from 'ethers/providers'
+ import { Web3Provider } from '@ethersproject/providers'

- export async function factoryFixture(_: Web3Provider, [wallet]: Wallet[]): Promise<FactoryFixture> {
+ export async function factoryFixture([wallet]: Wallet[], _: Web3Provider): Promise<FactoryFixture> {
    const factory = await deployContract(wallet, UniswapV2Factory, [wallet.address], overrides)
    return { factory }
  }
  
- export async function pairFixture(provider: Web3Provider, [wallet]: Wallet[]): Promise<PairFixture> {
+ export async function pairFixture([wallet]: Wallet[], provider: Web3Provider): Promise<PairFixture> {
-   const { factory } = await factoryFixture(provider, [wallet])
+   const { factory } = await factoryFixture([wallet], provider)
    ...
    return { factory, token0, token1, pair }
  }
In test/UniswapV2Pair.spec.ts:

- import { BigNumber, bigNumberify } from 'ethers/utils'
+ import { BigNumber } from '@ethersproject/bignumber'
- import { AddressZero } from 'ethers/constants'
+ import { AddressZero } from '@ethersproject/constants'

const provider = new MockProvider({
+ ganacheOptions: {
    hardfork: 'istanbul',
    mnemonic: 'horn horn horn horn horn horn horn horn horn horn horn horn',
    gasLimit: 9999999
+ }    
})

- const loadFixture = createFixtureLoader(provider, [wallet, other])
+ const loadFixture = createFixtureLoader([wallet, other], provider)

// Make the below change everywhere you see `bigNumberify`
- bigNumberify
+ BigNumber.from
You can now run yarn test:evm and all tests should pass!

Optimism Setup
Dependencies
Now that we’ve migrated to Waffle v3, let’s install the tools we’ll need to work with Optimism. These tools require Node v10. We recommend using a Node version manager, such as Volta.

If you are using Volta, run volta pin node@10 && volta pin yarn. This will automatically update package.json to specify the required node version. (Note that if we only pin node, but not yarn, volta will complain).

There’s two Optimism packages we’ll need:

Typically you compile contracts to be deployed on the EVM, but here we’ll use @eth-optimism/solc to compile the contracts for the OVM. The Uniswap contracts use Solidity 0.5.16, so when installing this package we must specify that to ensure we get the right version. From the npm page, we see the most recent version of 0.5.16 is called 0.5.16-alpha.7
@eth-optimism/ovm-toolchain provides wrappers and plugins for common tools. For example, it provides custom implementations of Ganache, Waffle V3, and Buidler which are compatible with the OVM. (Note: even though Buidler was replaced by Hardhat, this package is not yet compatible with Hardhat).
Let’s install both of these with:

$ yarn add --dev @eth-optimism/solc@0.5.16-alpha.7 @eth-optimism/ovm-toolchain 
Compiling for the OVM
We’re just about ready to compile our contracts for the OVM! Let’s prepare a few final things first.

When compiling, we need to make sure to use the OVM compiler we just installed. The existing compile script reads the EVM compiler settings from .waffle.json. We’ll create another file to manage OVM compiler settings, called .waffle-ovm.json, and point it to the OVM compiler as follows:

{
  "compilerVersion": "./node_modules/@eth-optimism/solc"
}
All compiler settings can be specified in this file, but for now this will be the only setting.

Finally, let’s setup our OVM package.json scripts. To be explicit, we’ll append :evm to the existing scripts in package.json, and add a few new :ovm ones. The result should look like this:

"scripts": {
  "lint": "yarn prettier ./test/*.ts --check",
  "lint:fix": "yarn prettier ./test/*.ts --write",
  "clean": "rimraf ./build/",
  "precompile:evm": "yarn clean",
  "precompile:ovm": "yarn clean",
  "compile:evm": "waffle .waffle.json",
  "compile:ovm": "waffle .waffle-ovm.json",
  "pretest:evm": "yarn compile:evm",
  "pretest:ovm": "yarn compile:ovm",
  "test:evm": "mocha",
  "test:ovm": "echo OVM TESTS NOT IMPLEMENTED",
  "prepublishOnly": "yarn test:evm && yarn test:ovm"
},
Let’s make sure things are working. Check that all tests still pass when running yarn test:evm.

Now run yarn test:ovm. This won’t run any tests yet, but it will compile the contracts for the OVM. If you did everything correctly, you should see OVM TESTS NOT IMPLEMENTED in the terminal, along with two compiler warnings:

contracts/UniswapV2Factory.sol:15:46: Warning: OVM: Taking arguments in constructor may result in unsafe code.
    constructor(address _feeToSetter) public {
                                             ^

contracts/test/ERC20.sol:6:43: Warning: OVM: Taking arguments in constructor may result in unsafe code.
    constructor(uint _totalSupply) public {
These are new compiler warnings specific to the OVM, so let’s understand what’s going on here.

OVM vs. EVM: Constructor Arguments
In this section we’ll look at the warnings we saw above when compiling for the OVM. While, this guide focuses on some of the OVM vs. EVM differences that specifically impact Uniswap V2, you can read about other OVM vs. EVM differences here and here.

UniswapV2Factory.sol is responsible for deploying new pairs, and its constructor takes one argument which the compiler warns may result in unsafe code. (Learn more about this argument in the Uniswap docs). This warning is emitted because not all EVM opcodes are supported in the OVM! As a result, when your encoded constructor arguments are added to the contract bytecode, there is a chance this encoding contains unsupported opcodes. If it does, contract deployment will fail. As explained here:

Only a few opcodes are banned, so this is a relatively unlikely event. However, if you have a strong requirement that your contract can be successfully deployed multiple times, with absolutely any parameters, it is problematic. In that case, you will need to remove constructor arguments and replace them with an initialize(...) method which can only be called once on the deployed code.

Because the UniswapV2Factory constructor simply saves the constructor argument to storage, we can safely ignore the warning here. Similarly, the test ERC20 contract only mints _totalSupply tokens at construction, so this operation should also be safe.

OVM vs. EVM: Block Timestamps
The difference between block timestamps on the OVM and the EVM don’t naturally surface during the migration process, but Uniswap does rely on block.timestamp in a few places so it’s important to mention. If your contracts rely on block.timestamp, you’ll want to understand these differences and consider their impact carefully.

The OVM does not have blocks; it just has an ordered list of transactions. As such,block.timestamp returns the “Latest L1 Timestamp”, which corresponds to the timestamp of the last L1 block in which a rollup batch was posted. Be aware that this will lag behind the EVM’s block.timestamp by about 1–10 minutes.

Uniswap uses block.timestamp for a few different purposes, so let’s see how the behavior of each changes as a result of the differing functionality.

Trade Deadlines, permit signatures, and auctions
When executing a trade you can set a deadline, and if your transaction is mined after this deadline the transaction will be reverted. (This actually occurs in the router contract, outside of the repository we cloned, but is worth mentioning anyway). The OVM timestamp lags behind the EVM, so it’s possible that your OVM Uniswap trades execute up 10 minutes after your specified deadline.

Similarly, you can use the permit method to give approval to an address to spend your LP tokens with just a signature. These signatures contain a deadline, and the approval must be sent before that deadline. Just like with trade deadlines, this means it’s possible that this approval is executed after your desired deadline.

One other major area timestamps come into play is with bid and auction durations. MakerDAO used have to bid durations of 10 minutes, and it’s important to know that due to the timestamp lag, these 10 minute durations would be unsafe on the OVM! Note that these durations have since increased since it turns out they weren’t ideal on L1 either.

Price Oracle
Uniswap uses a cumulative-price variable to enable developers to build safe Oracles on top of Uniswap. This variable tracks the “sum of the Uniswap price for every second in the entire history of the contract”, meaning it must know the current time whenever it’s updated. Because timestamps lag, the resulting price from an OVM Uniswap oracle will likely vary a bit from an EVM Uniswap oracle that had identical historical states.

Testing on the OVM
Our final step is getting our tests working for the OVM. If we make the following change to the test:ovm script, so it runs mocha, there will be plenty of test failures. Try it!

  "test:evm": "mocha",
- "test:ovm": "echo OVM TESTS NOT IMPLEMENTED",
+ "test:ovm": "mocha",
Let’s fix those failures using the @eth-optimism/ovm-toolchain package we installed earlier. Remember, this package provides Optimism-specific implementations of Ganache, Waffle, and Buidler.

We’ll need to setup the tests differently depending on whether we are running them against the EVM or OVM, so we’ll use an environment variable called MODE to do this. If MODE=OVM, we’ll use our new OVM settings, otherwise we’ll fallback to the standard EVM configuration. We can do this easily with the following change in package.json:

- "test:ovm": "mocha",
+ "test:ovm": "export MODE=OVM && mocha",
Next we’ll create a file in test/shared called config.ts, where we’ll handle all chain-specific logic. Leave this blank for now, we’ll adjust it in the next section.

Provider and Chain ID Fixes
Waffle starts a local node when a new provider instance is created with new MockProvider(). Typically this is a normal EVM instance, but for the OVM tests we need an OVM instance. As a result we’ll use the MockProvider from @eth-optimism/ovm-toolchain for the OVM tests, instead of Waffles’s standard MockProvider.

You may recall seeing const provider = new MockProvider({...}) in the Uniswap tests, and this is exactly what we’ll update.

Uniswap tokens support the signature-based permit method to enable approvals via meta-transactions, and to avoid replay attacks the signature is based on the chain ID. While Ethereum mainnet uses a chain ID of 1, the OVM has a chain ID of 420. That means tests reliant on the chain ID will fail unless we update them. We can handle both of these changes in our config.ts as shown below:

/**
 * @dev Handles chain-specific configuration based on whether we are running
 * EVM or OVM tests
 */

import { MockProvider } from 'ethereum-waffle' // standard EVM MockProvider from Waffle
import { waffleV3 } from '@eth-optimism/ovm-toolchain' // custom OVM version of Waffle V3

// Determine which network we are on
const isOVM = process.env.MODE === 'OVM'

// Get provider: We keep the same provider config that Uniswap tests were
// already using, but generate the provider instance based on the test mode
const options: any = {
  ganacheOptions: {
    hardfork: 'istanbul',
    mnemonic: 'horn horn horn horn horn horn horn horn horn horn horn horn',
    gasLimit: 9999999
  }
}
const provider = isOVM ? new waffleV3.MockProvider(options) : new MockProvider(options)

// Get Chain ID
const chainId = isOVM ? 420 : 1

export { provider, chainId }


Since @eth-optimism/ovm-toolchain does not have type definitions, the TypeScript compiler will complain since the new MockProvider implicitly has an any type. To avoid having to declare these types ourselves let’s just edit the tsconfig.json to turn off this rule:

{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
+   "noImplicitAny": false
  }
}

We can test out our new provider and chain ID by updating all the instances in the tests, then making sure the EVM tests still run.

Make the following changes in test/UniswapV2ERC20.spec.ts

- import { solidity, MockProvider, deployContract } from 'ethereum-waffle'
+ import { solidity, deployContract } from 'ethereum-waffle'
+ import { provider } from './shared/config'

- const provider = new MockProvider({
-   ganacheOptions: {
-     hardfork: 'istanbul',
-     mnemonic: 'horn horn horn horn horn horn horn horn horn horn horn horn',
-     gasLimit: 9999999
-   }
- })
Make similar changes in test/UniswapV2Factory.spec.ts and test/UniswapV2Pair.spec.ts:

- import { solidity, MockProvider, createFixtureLoader } from 'ethereum-waffle'
+ import { solidity, createFixtureLoader } from 'ethereum-waffle'
+ import { provider } from './shared/config'

- const provider = new MockProvider({
-   ganacheOptions: {
-     hardfork: 'istanbul',
-     mnemonic: 'horn horn horn horn horn horn horn horn horn horn horn horn',
-     gasLimit: 9999999
-   }
- })
We update the chain ID in test/shared/utilities.ts:

+ import { chainId } from './config'

function getDomainSeparator(name: string, tokenAddress: string) {
  return keccak256(
    defaultAbiCoder.encode(
      ['bytes32', 'bytes32', 'bytes32', 'uint256', 'address'],
      [
        keccak256(toUtf8Bytes('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)')),
        keccak256(toUtf8Bytes(name)),
        keccak256(toUtf8Bytes('1')),
-       1
+       chainId,
        tokenAddress
      ]
    )
  )
}
and again in test/UniswapV2ERC20.spec.ts:

- import { provider } from './shared/config'
+ import { provider, chainId } from './shared/config'

expect(await token.DOMAIN_SEPARATOR()).to.eq(
      keccak256(
        defaultAbiCoder.encode(
          ['bytes32', 'bytes32', 'bytes32', 'uint256', 'address'],
          [
            keccak256(
              toUtf8Bytes('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)')
            ),
            keccak256(toUtf8Bytes(name)),
            keccak256(toUtf8Bytes('1')),
-           1            
+           chainId,
            token.address
          ]
        )
      )
    )
Ok, done! Run yarn test:evm to make sure all tests still pass.

Now run yarn test:ovm and some tests pass! But a lot still fail. You should see something like this:

  UniswapV2ERC20
    ✓ name, symbol, decimals, totalSupply, balanceOf, DOMAIN_SEPARATOR, PERMIT_TYPEHASH (1271ms)
    ✓ approve (335ms)
    ✓ transfer (524ms)
    ✓ transfer:fail (381ms)
    ✓ transferFrom (868ms)
    ✓ transferFrom:max (855ms)
    ✓ permit (820ms)

  UniswapV2Factory
    1) "before each" hook for "feeTo, feeToSetter, allPairsLength"

  UniswapV2Pair
    2) "before each" hook for "mint"


  7 passing (19s)
  2 failing

  1) UniswapV2Factory
       "before each" hook for "feeTo, feeToSetter, allPairsLength":
     RuntimeError: VM Exception while processing transaction: out of gas
      at Function.RuntimeError.fromResults (node_modules/ganache-core/lib/utils/runtimeerror.js:94:13)
      at BlockchainDouble.processBlock (node_modules/ganache-core/lib/blockchain_double.js:627:24)
      at process._tickCallback (internal/process/next_tick.js:68:7)

  2) UniswapV2Pair
       "before each" hook for "mint":
     RuntimeError: VM Exception while processing transaction: out of gas
      at Function.RuntimeError.fromResults (node_modules/ganache-core/lib/utils/runtimeerror.js:94:13)
      at BlockchainDouble.processBlock (node_modules/ganache-core/lib/blockchain_double.js:627:24)
      at process._tickCallback (internal/process/next_tick.js:68:7)
Gas and Compiler Fixes
Time to fix the above test failures. The error message is out of gas, so maybe our gas limits aren’t high enough. In config.ts we specify a very high gas limit of 9,999,999. What’s going on?

Notice the failures are in the beforeEach hooks. The only thing we do in those hooks is deploy a contract. It seems our contract deployment is failing with a misleading error message. Let’s dig into why.

OVM vs. EVM: Gas
One important difference between the EVM and the OVM is the concept of “blocks.” While EVM networks consist of an ever-growing chain of blocks, blocks don’t exist on the OVM. As such, there’s no concept of a block gas limit. Instead there is a transaction gas limit, which is currently set to 9,000,000.

You can read about this, and other gas related differences, here.

OVM vs. EVM: Compiling
Spoiler: Our contract deployment is failing because when we compiled it for the OVM, it became too big!

Both the Ethereum mainnet and the OVM have a 24 kB limit on deployed contract size—you cannot deploy contracts bigger than this. Compiling for the OVM results in an increase in contract size compared to compiling for the EVM, and the default compiler settings are especially bloat-inducing. Fortunately, we can shrink it with the optimizer.

Let’s edit our .waffle-ovm.json file to match the compiler settings used by Uniswap, which gives us this:

{
  "compilerVersion": "./node_modules/@eth-optimism/solc",
  "outputType": "all",
  "compilerOptions": {
    "outputSelection": {
      "*": {
        "*": [
          "evm.bytecode.object",
          "evm.deployedBytecode.object",
          "abi",
          "evm.bytecode.sourceMap",
          "evm.deployedBytecode.sourceMap",
          "metadata"
        ],
        "": ["ast"]
      }
    },
    "evmVersion": "istanbul",
    "optimizer": {
      "enabled": true,
      "runs": 999999
    }
  }
}
Run yarn test:ovm, and almost all tests pass! (There’s a few small gas-related failures we’ll adjust later after we review what just happened).

We turned on the optimizer with a setting of 999,999 runs. This shrank our contract size significantly, bringing us back under the 24 kB size limit! It’s important to understand how the Solidity optimizer works, because even though 999,999 runs works here, that may not always be the case.

The runs parameter specifies “roughly how often each opcode of the deployed code will be executed across the lifetime of the contract”. A value of 1 produces shorter code that is more expensive to execute, while large values produce longer code that is cheaper. So you can see how we got a bit lucky in this case—because a large number of runs produces code that’s bigger in size, it’s possible with some contracts that using a large number of runs would result in a contract that is still to big deploy.

When compiling with the OVM, at a minimum be sure to turn on the optimizer with runs set to 1. Even a small value will result in a significant reduction in contract size!

One other note on OVM compilation: The deployedBytecode output of solc is currently broken. This will most likely not affect you, as contract deployments deploy the bytecode, but it’s worth being aware of.

Final Gas Tweaks
There were two tests that failed because they look for an exact gas usage. Because the OVM is not identical to the EVM, gas usage differs a bit. From the failure messages we can see what the expected gas usage should be for the OVM, so let’s fix this quickly.

First we need to be able to identify which mode we’re using in the tests, so let’s export that in config.ts:

- export { provider, chainId }
+ export { provider, chainId, isOVM }
In UniswapV2Factory.spec.ts, let’s update the failing test:

- import { provider } from './shared/config'
+ import { provider, isOVM } from './shared/config'

- expect(receipt.gasUsed).to.eq(2512920)
+ expect(receipt.gasUsed).to.eq(isOVM ? 4121083 : 2512920)
And similarly in UniswapV2Pair.spec.ts:

- import { provider } from './shared/config'
+ import { provider, isOVM } from './shared/config'

- expect(receipt.gasUsed).to.eq(73462)
+ expect(receipt.gasUsed).to.eq(isOVM ? 657108 : 73462)
Finally, run yarn test:evm and yarn test:ovm, and everything should pass!
# optimism-uniswapv2-core
# optimism-uniswapv2-core
# optimism-rollup-uniswapv2-core
# optimism-rollup-uniswapv2-example
