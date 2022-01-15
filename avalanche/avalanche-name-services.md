# Introduction

The Avalanche Name Service (ANS) is a distributed, open, and extensible naming system based on the
Avalanche blockchain. ANS’s job is to map human-readable names like ‘dev.avax’ to machine-readable identifiers such as
Avalanche C Chain addresses, other cryptocurrency addresses, content hashes, and metadata.
ANS has similar goals to DNS, the Internet’s Domain Name Service, but has significantly different
architecture due to the capabilities and constraints provided by the Avalanche blockchain. Like DNS, ANS
operates on a system of dot-separated hierarchical names called domains, with the owner of a domain
having full control over subdomains.
Top-level domains, like ‘.avax’ and ‘.test’, are owned by smart contracts called registrars, which specify rules
governing the allocation of their subdomains. Anyone may, by following the rules imposed by these registrar
contracts, obtain ownership of a domain for their own use. ANS also supports importing in DNS names
already owned by the user for use on ANS.
Because of the hierarchal nature of ANS, anyone who owns a domain at any level may configure
subdomains - for themselves or others - as desired. For instance, if Dev owns 'dev.avax', he can create
'pay.dev.avax' and configure it as he desires.
ANS is deployed on the Avalanche main network and on Fuji test networks.
It's similar to ENS architecture since it's a fork but on Avalanche blockchain.

# Prerequisites

- [Avalanche](https://docs.avax.network/) is an EVM compatible blockchain which work on POS mechanism.
- Familiarity with [Solidity](https://docs.soliditylang.org/) and [ReactJS](https://reactjs.org/) are recommended.

# Requirements

* [NodeJS](https://nodejs.org/en)
* [ReactJS](https://reactjs.org/)
* Hardhat, which you can install with `npm install -g hardhat`
* Install [Metamask extension](https:// https://github.com/Devilla/ans-contractsmetamask.io/download.html) in your browser.

# ANS Architecture

ANS has three main components: the `ans-contracts` , `ans-subgrapgh` and `ans-app` for their respective work like `registry of names`, `querying the data` and `user interaction` on Avalanche.

**Introduction to smart contracts**

Clone the [Git Repository](https://github.com/Devilla/ans-contracts) and [Create a Local Test Network](https://learn.figment.io/tutorials/create-a-local-test-network) following the tutorial.

```
git clone https://github.com/Devilla/ans-contracts
```
Go to the repository:
```
cd ans-contracts
```
Install the required depencencies:
```
yarn
```

**Add Avalanche network config in `hardhat.config.ja`**

For Development Network:
```
devnet: {
      url: 'http://localhost:9650/ext/bc/C/rpc',
      gasPrice: 225000000000,
      gas:5e6,
      blockGasLimit: 8000000,
      accounts: []
    }
```

For Fuji Testnet:
```
    fuji: {
      url: 'https://api.avax-test.network/ext/bc/C/rpc',
      gasPrice: 25000000000,
      gas:8000000,
      chainId: 43113,
      accounts: {
        mnemonic: ""
      }
    }
```


