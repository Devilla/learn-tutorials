# Introduction

The Avalanche Name Service (ANS) is a decentralised naming system based on the
Avalanche blockchain. ANS’s job is to map human-readable names like ‘dev.avax’ to machine-readable identifiers such as
Avalanche C-Chain addresses, other cryptocurrency addresses, content hashes, and metadata.

Top-level domains ‘.avax’ is owned by smart contracts called registrars, which specify rules
governing the allocation of its subdomains. Anyone may, by following the rules imposed by these registrar
contracts, obtain ownership of a domain for their own use. ANS also supports importing in DNS names
already owned by the user on ANS.
Because of the hierarchal nature of ANS, anyone who owns a domain at any level may configure
subdomains - for themselves or others - as desired. For instance, if Dev owns 'dev.avax', he can create like
'iam.dev.avax' and configure it as he/she desires.

# Prerequisites

- [Avalanche](https://docs.avax.network/) is an EVM compatible blockchain which work on POS mechanism.
- Familiarity with [Ethereum Name Services](https://docs.ens.domains/) is recommended.

# Requirements

* [NodeJS](https://nodejs.org/en)
* [ReactJS](https://reactjs.org/)
* [Solidity](https://docs.soliditylang.org/)
* Hardhat, which you can install with `npm install -g hardhat`
* Install [Metamask extension](https://metamask.io/) in your browser.

# ANS Architecture

ANS has three main components: the `ans-contracts` , `ans-subgrapgh` and `ans-app` for their respective work like `registry of names`, `querying the data` and `user interaction` on Avalanche.

## ANS Contracts

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

**Add Avalanche network config in `hardhat.config.js`**

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

First, let's **deploy the baseRegstrar contracts with 'avax'** as top level domain from `deploy/basregistrar/00_deploy_base_registrar.js`

Import ethers library from hardhat
```
const { ethers } = require("hardhat");
```
Create constants ZERO_ADDRESS, ZERO_HASH
```
const ZERO_ADDRESS = "0x0000000000000000000000000000000000000000";
const ZERO_HASH = "0x0000000000000000000000000000000000000000000000000000000000000000"
```
Import namehash and sha3 hashing methods from eth-ens-namehash, web3-utils packages respectively
```
const namehash = require('eth-ens-namehash');
const sha3 = require('web3-utils').sha3;
```

Export modules for deployment BaseRegistrarImplementation and initialise deployments, deployer, owner and ENSRegistry
```
module.exports = async ({getNamedAccounts, deployments, network}) => {
    const {deploy} = deployments;
    const {deployer, owner} = await getNamedAccounts();

    const ens = await ethers.getContract('ENSRegistry')
```

Deploy BaseRegistrarImplementation contract with params deployer, args as (ens.address, namehash.hash('avax')) and log as true
```
    await deploy('BaseRegistrarImplementation', {
        from: deployer, 
        args: [ens.address, namehash.hash('avax')],
        log: true
    })
```
Create constants `base` for BaseRegistrarImplementation and `transactions` array 
```
    const base = await ethers.getContract('BaseRegistrarImplementation');
    const transactions = []
```

Push transactions to add Controller on base registrar and set Subnode Owner on ens registry
```
    transactions.push(await base.addController(owner, {from: deployer}))
    transactions.push(await ens.setSubnodeOwner(ZERO_HASH, sha3('avax'), base.address))
```

Log while waiting for Promise for transaction to finish
```
    console.log(`Waiting on ${transactions.length} transactions setting base registrar`);
    await Promise.all(transactions.map((tx) => tx.wait()));
}
```
Export modules tags and dependencies for baseregistrar and registry
```
module.exports.tags = ['baseregistrar'];
module.exports.dependencies = ['registry']
```

**Deploy Reverse Registrar contracts for registration** from `deploy/basregistrar/10_deploy_reverse_registrar.js`.

Import the libraries and utils
```
const { ethers } = require("hardhat");
const ZERO_HASH = "0x0000000000000000000000000000000000000000000000000000000000000000";
const sha3 = require('web3-utils').sha3;
const namehash = require('eth-ens-namehash');
```
Export module ReverseRegistrar deployment and initilise constant deployments, deployer and owner
```
module.exports = async ({getNamedAccounts, deployments, network}) => {
    const {deploy} = deployments;
    const {deployer, owner} = await getNamedAccounts();
```
Assign constants `ens` and `resolver` by getting contracts for ENSRegistry and PublicResolver respectively using ethers library
```
    const ens = await ethers.getContract('ENSRegistry');
    const resolver = await ethers.getContract('PublicResolver');
```
Await deployment for ReverseRegistrar with account as deployer and args as `ens.address`, `resolver.address`
```
    await deploy('ReverseRegistrar', {
        from: deployer, 
        args:[ens.address, resolver.address]
    })
```
Create constants `reverseRegistrar` for ReverseRegistrar and `transactions` array 
```
    const reverseRegistrar = await ethers.getContract('ReverseRegistrar');
    const transactions = []
```
Push transctions and map inside Promise.all() to setting Subnode Owner on reverse namehash for reverseRegistrar
```
    transactions.push(await ens.setSubnodeOwner(ZERO_HASH, sha3('reverse'), deployer))
    transactions.push(await ens.setSubnodeOwner(namehash.hash('reverse'),sha3('addr'),reverseRegistrar.address))
    console.log(`Waiting on settings to take place of reverse registrar ${transactions.length}`)
    await Promise.all(transactions.map((tx) => tx.wait()));
}
```
Export module tags as 'reverse-registrar' and dependencies as 'registry', 'public-resolver'
```
module.exports.tags = ['reverse-registrar'];
module.exports.dependencies = ['registry', 'public-resolver']
```

**Deploy `ETHRegistrarController` with  `StablePriceOracle` contracts** for '.avax' as root domain from `deploy/ethregistrar/00_deploy_eth_registrar.js`

Import the ethers library from hardhat package
```
const { ethers } = require("hardhat");
```
Create a zero hash constant for initalization of an address
```
const ZERO_HASH = "0x0000000000000000000000000000000000000000000000000000000000000000"
```
Import sha3 hashing method from web3-utils package
```
const sha3 = require('web3-utils').sha3;
```
Export module for deployment and initialize constants deployments, deployer, owner etc.
```
module.exports = async ({getNamedAccounts, deployments, network}) => {
    const {deploy} = deployments;
    const {deployer, owner} = await getNamedAccounts();
```

Initialse constants `baseRegistrar` and `priceOracle` for BaseRegistrarImplementation and StablePriceOracle
```
    const baseRegistrar = await ethers.getContract('BaseRegistrarImplementation');

    const priceOracle = await ethers.getContract('StablePriceOracle')
```

Deploy ETHRegistrarController contract with baseRegistrar and priceOracle
```
    await deploy('ETHRegistrarController', {
        from: deployer, 
        args: [baseRegistrar.address, priceOracle.address, 600, 86400],
        log: true
    })
```
    
Initialise constants *controller*, *ens* and *transactions* for *ETHRegistrarController*, *ENSRegistry* and transaction array
```
    const controller = await ethers.getContract('ETHRegistrarController')
    const ens = await ethers.getContract('ENSRegistry')
    const transactions = []
```

Push transactions for controller to set ens SubnodeOwner for 'avax' on ZERO_HASH address
```
    transactions.push(await ens.setSubnodeOwner(ZERO_HASH,sha3('avax'),controller.address));
```
Push transactions for baseRegistrar to  add controller 
```
    transactions.push(await baseRegistrar.addController(controller.address, {from: deployer}));
```

Push transactions for controller to set PriceOracle using deployer
```
    transactions.push(await controller.setPriceOracle(priceOracle.address, {from: deployer}));
```
add Promise, wait and log while waiting transactions to complete
```
    console.log(`Waiting on settings to take place ${transactions.length}`)
    await Promise.all(transactions.map((tx) => tx.wait()));
}
```

Export module tags as 'eth-registrar' and dependencies as 'registry', 'oracles'
```
module.exports.tags = ['eth-registrar'];
module.exports.dependencies = ['registry', 'oracles']
```

Deploy *DummyOracle* and *StablePriceOracle* from deploy/registry/00_deploy_registry.js

```
const { ethers } = require("hardhat");

module.exports = async ({getNamedAccounts, deployments, network}) => {
    const {deploy} = deployments;
    const {deployer, owner} = await getNamedAccounts();
    const oracle = await deploy('DummyOracle', {
        from: deployer, 
        args:[1000],
        log:true
    })

    await deploy('StablePriceOracle', {
        from: deployer, 
        args:[oracle.address,[1]],
        log:true
    })
    

}

module.exports.tags = ['oracles'];
module.exports.dependencies = ['registry']
```

Deploy Reverse Registrar from deploy/publicresolver/00_deploy_reverse_resolver.js

```
const { ethers } = require("hardhat");
ZERO_HASH= "0x0000000000000000000000000000000000000000000000000000000000000000"
const sha3 = require('web3-utils').sha3;

module.exports = async ({getNamedAccounts, deployments, network}) => {
    const {deploy} = deployments;
    const {deployer, owner} = await getNamedAccounts();

    const ens = await ethers.getContract('ENSRegistry')

    await deploy('DefaultReverseResolver', {
        from: deployer, 
        args: [ens.address],
        log: true
    })

    const baseRegistrar = await ethers.getContract('BaseRegistrarImplementation')
    const controller = await ethers.getContract('ETHRegistrarController')
    const priceOracle = await ethers.getContract('StablePriceOracle')

    const transactions = []

    transactions.push(await ens.setSubnodeOwner(ZERO_HASH,sha3('avax'),controller.address));
    transactions.push(await baseRegistrar.addController(controller.address, {from: deployer}));
    // ESTIMATE GAS -->
    transactions.push(await controller.setPriceOracle(priceOracle.address, {from: deployer}));
    console.log(`Waiting on settings to take place on reverse-registrar ${transactions.length}`)
    await Promise.all(transactions.map((tx) => tx.wait()));



}

module.exports.dependencies = ['registry', 'baseregistrar', 'eth-registrar']
```

Deploy Public Resolver from deploy/publicresolver/10_deploy_public_resolver.js

```
const { ethers } = require("hardhat");
const ZERO_ADDRESS = "0x0000000000000000000000000000000000000000";
const ZERO_HASH = "0x0000000000000000000000000000000000000000000000000000000000000000"
const sha3 = require('web3-utils').sha3;
const namehash = require('eth-ens-namehash');
module.exports = async ({getNamedAccounts, deployments, network}) => {
    const {deploy} = deployments;
    const {deployer, owner} = await getNamedAccounts();

    const ens = await ethers.getContract('ENSRegistry')

    await deploy('PublicResolver', {
        from: deployer, 
        args: [ens.address, ZERO_ADDRESS],
        log: true
    })

    const resolver = await ethers.getContract('PublicResolver')
    const ethregistrar = await ethers.getContract('ETHRegistrarController')

    const transactions = []
    transactions.push(await ens.setSubnodeOwner(ZERO_HASH, sha3('avax'), owner))
    transactions.push(await ens.setResolver(namehash.hash('avax'), resolver.address))
    transactions.push(await resolver['setAddr(bytes32,address)'](namehash.hash('avax'), resolver.address))
    transactions.push(await resolver.setInterface(namehash.hash('avax'), "0x018fac06", ethregistrar.address))
    console.log(`Waiting on settings to take place on resolvers ${transactions.length}`)
    await Promise.all(transactions.map((tx) => tx.wait()));    

}

module.exports.tags = ['public-resolver'];
module.exports.dependencies = ['registry', 'eth-registrar']
```

Deploy Regsitry from deploy/registry/00_deploy_registry.js

```
const { ethers } = require("hardhat");

const ZERO_HASH = "0x0000000000000000000000000000000000000000000000000000000000000000";

module.exports = async ({getNamedAccounts, deployments, network}) => {
    const {deploy} = deployments;
    const {deployer, owner} = await getNamedAccounts();

    

    if(network.tags.legacy) {
        const contract = await deploy('LegacyENSRegistry', {
            from: deployer,
            args: [],
            log: true,
            contract: await deployments.getArtifact('ENSRegistry')
        });
        await deploy('ENSRegistry', {
            from: deployer,
            args: [contract.address],
            log: true,
            contract: await deployments.getArtifact('ENSRegistryWithFallback')
        });    
    } else {
        await deploy('ENSRegistry', {
            from: deployer,
            args: [],
            log: true,
        }); 
        
    }

    if(!network.tags.use_root) {
        const registry = await ethers.getContract('ENSRegistry');
        const rootOwner = await registry.owner(ZERO_HASH);
        switch(rootOwner) {
        case deployer:
            const tx = await registry.setOwner(ZERO_HASH, owner, {from: deployer});
            console.log("Setting final owner of root node on registry (tx:${tx.hash})...");
            await tx.wait();
            break;
        case owner:
            break;
        default:
            console.log(`WARNING: ENS registry root is owned by ${rootOwner}; cannot transfer to owner`);
        }
    }

    return true;
};
module.exports.tags = ['registry'];
module.exports.id = "ens";
```

Deploy Root from deploy/root/00_deploy_root.js
```
const { ethers } = require("hardhat");

const ZERO_HASH = '0x0000000000000000000000000000000000000000000000000000000000000000';

module.exports = async ({getNamedAccounts, deployments, network}) => {
    const {deploy} = deployments;
    const {deployer, owner} = await getNamedAccounts();

    if(!network.tags.use_root) {
        return true;
    }

    const registry = await ethers.getContract('ENSRegistry');

    await deploy('Root', {
        from: deployer,
        args: [registry.address],
        log: true,
    });

    const root = await ethers.getContract('Root');

    let tx = await registry.setOwner(ZERO_HASH, root.address);
    console.log(`Setting owner of root node to root contract (tx: ${tx.hash})...`);
    await tx.wait();
    
    const rootOwner = await root.owner();
    switch(rootOwner) {
    case deployer:
        tx = await root.attach(deployer).transferOwnership(owner);
        console.log(`Transferring root ownership to final owner (tx: ${tx.hash})...`);
        await tx.wait();
    case owner:
        if(!await root.controllers(owner)) {
            tx = await root.attach(owner).setController(owner, true);
            console.log(`Setting final owner as controller on root contract (tx: ${tx.hash})...`);
            await tx.wait();
        }
        break;
    default:
        console.log(`WARNING: Root is owned by ${rootOwner}; cannot transfer to owner account`);
    }

    return true;
};
module.exports.id = "root";
module.exports.tags = ['root'];
module.exports.dependencies = ['registry'];
```

Compile and deploy smart contracts using hardhat on Fuji Testnet
```
npx harhat compile
```
After successfull compilation, deploy the contracts for development network
```
npx hardhat deploy --network devnet
```
and for fuji test network
```
npx hardhat deploy --network fuji
```

Deployment output:
```
reusing "BaseRegistrarImplementation" at 0x01EB97a856686a7ab01F8196015265Ef743b2672
Waiting on 2 transactions setting base registrar
reusing "DummyOracle" at 0xcfA95eb1dF4642310d7b6e0354419CacAee190E1
reusing "StablePriceOracle" at 0x8F15e41e2F45914Cc3063586dB65D5033f5Eb8d6
reusing "ETHRegistrarController" at 0x163CDD36Db18be7dd0E51f9CD03771CF843058Ee
Waiting on settings to take place 3
reusing "PublicResolver" at 0x171F9B5D86D25D9064F2c78E6Cc90ccEC907d6A4
Waiting on settings to take place on resolvers 4
Waiting on settings to take place of reverse registrar 2
reusing "ReverseRegistrar" at 0xbAc25db0aD5D96B1fe3250a5335e25D62aB78f1C
Waiting on settings to take place on reverse-registrar 3
deploying "DefaultReverseResolver" (tx: 0xe6ab0be56dc99d4c06c1534b4a8fb5175ac9930d1e84a517ee83dfdb4c391888)...: deployed at 0xfa1a8df76f363067BE5d66066e2DCF563E81615E with 418037 gas
```

## ANS Subgraph

We will use [The Graph protocol](https://thegraph.com/docs) for quering address resources i.e. human-readable domain names (Eg. dev.avax) from the blockchain, access domains and transfer history.

Clone the [Git Repository](https://github.com/Devilla/ans-subgraph).

```
git clone https://github.com/Devilla/ans-subgraph
```
Go to the repository:
```
cd ans-subgraph
```
Install the required depencencies and run codegen
```
yarn

yarn codegen
```
To get the subgraph working locally run:
```
yarn create-local
```

Then you can deploy the subgraph:
```
yarn deploy-local
```

This will start up a GraphiQL interface at http://127.0.0.1:8000/ in your browser, check it for querying the data.

or you can use Hosted Service provided by [thegraph protocol](https://thegraph.com/docs/en/hosted-service/what-is-hosted-service). Follow the docs and create your subgraph and add that in the `deploy` scrpts `package.json`.

```    
"deploy": "graph deploy --node https://api.thegraph.com/deploy/ <USERNAME>/<SUBGRAPH NAME> --ipfs https://api.thegraph.com/ipfs/",
```

But before deploying, you need to map the contract ABIs and addresses for the fuji testnet in the `subgraph.yaml`
```
specVersion: 0.0.2
description: >-
  A secure & decentralized way to address resources on and off the blockchain
  using simple, human-readable names. Access domains and transfer history.
repository: 'https://github.com/Devilla/ans-subgraph'
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum/contract
    name: ENSRegistry
    network: fuji
    source:
      address: '0x36479f8eB17e2C7e05427E3c121E80ff3f8017C9'
      abi: EnsRegistry
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.5
      language: wasm/assemblyscript
      file: ./src/ensRegistry.ts
      entities:
        - Domain
        - Account
        - Resolver
      abis:
        - name: EnsRegistry
          file: ./abis/Registry.json
      eventHandlers:
        - event: 'Transfer(indexed bytes32,address)'
          handler: handleTransfer
        - event: 'NewOwner(indexed bytes32,indexed bytes32,address)'
          handler: handleNewOwner
        - event: 'NewResolver(indexed bytes32,address)'
          handler: handleNewResolver
        - event: 'NewTTL(indexed bytes32,uint64)'
          handler: handleNewTTL
  - kind: ethereum/contract
    name: ENSRegistryOld
    network: fuji
    source:
      address: '0x1F68692D58bABeDC2549a49a2F73cf05fc078af1'
      abi: EnsRegistry
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.5
      language: wasm/assemblyscript
      file: ./src/ensRegistry.ts
      entities:
        - Domain
        - Account
        - Resolver
      abis:
        - name: EnsRegistry
          file: ./abis/Registry.json
      eventHandlers:
        - event: 'Transfer(indexed bytes32,address)'
          handler: handleTransferOldRegistry
        - event: 'NewOwner(indexed bytes32,indexed bytes32,address)'
          handler: handleNewOwnerOldRegistry
        - event: 'NewResolver(indexed bytes32,address)'
          handler: handleNewResolverOldRegistry
        - event: 'NewTTL(indexed bytes32,uint64)'
          handler: handleNewTTLOldRegistry
  - kind: ethereum/contract
    name: Resolver
    network: fuji
    source:
      abi: Resolver
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.5
      language: wasm/assemblyscript
      file: ./src/resolver.ts
      entities:
        - AddrChanged
        - MulticoinAddrChanged
        - NameChanged
        - AbiChanged
        - PubkeyChanged
        - Textchanged
        - ContenthashChanged
        - InterfaceChanged
        - AuthorisationChanged
      abis:
        - name: Resolver
          file: ./abis/PublicResolver.json
      eventHandlers:
        - event: 'ABIChanged(indexed bytes32,indexed uint256)'
          handler: handleABIChanged
        - event: 'AddrChanged(indexed bytes32,address)'
          handler: handleAddrChanged
        - event: 'AddressChanged(indexed bytes32,uint256,bytes)'
          handler: handleMulticoinAddrChanged
        - event: 'AuthorisationChanged(indexed bytes32,indexed address,indexed address,bool)'
          handler: handleAuthorisationChanged
        - event: 'ContenthashChanged(indexed bytes32,bytes)'
          handler: handleContentHashChanged
        - event: 'InterfaceChanged(indexed bytes32,indexed bytes4,address)'
          handler: handleInterfaceChanged
        - event: 'NameChanged(indexed bytes32,string)'
          handler: handleNameChanged
        - event: 'PubkeyChanged(indexed bytes32,bytes32,bytes32)'
          handler: handlePubkeyChanged
        - event: 'TextChanged(indexed bytes32,indexed string,string)'
          handler: handleTextChanged
  - kind: ethereum/contract
    name: BaseRegistrar
    network: fuji
    source:
      address: '0xcfCBE461A002FFc38905e7A24b71E1Ad01980EB4'
      abi: BaseRegistrar
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.5
      language: wasm/assemblyscript
      file: ./src/ethRegistrar.ts
      entities:
        - Registration
        - NameRegistered
        - NameRenewed
        - NameTransferred
      abis:
        - name: BaseRegistrar
          file: ./abis/BaseRegistrar.json
      eventHandlers:
        - event: 'NameRegistered(indexed uint256,indexed address,uint256)'
          handler: handleNameRegistered
        - event: 'NameRenewed(indexed uint256,uint256)'
          handler: handleNameRenewed
        - event: 'Transfer(indexed address,indexed address,indexed uint256)'
          handler: handleNameTransferred
  - kind: ethereum/contract
    name: EthRegistrarController
    network: fuji
    source:
      address: '0x4BEf85c6063bf5b1fB13816f1d383C50aA698566'
      abi: EthRegistrarController
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.5
      language: wasm/assemblyscript
      file: ./src/ethRegistrar.ts
      entities:
        - Registration
      abis:
        - name: EthRegistrarController
          file: ./abis/EthRegistrarController.json
      eventHandlers:
        - event: 'NameRegistered(string,indexed bytes32,indexed address,uint256,uint256)'
          handler: handleNameRegisteredByController
        - event: 'NameRenewed(string,indexed bytes32,uint256,uint256)'
          handler: handleNameRenewedByController
```

**Build and deploy subgraph**
Build and check for errors in typescript it'll throw some type errors for some variables
```
TS2322: Type 'src/types/schema/Domain | null' is not assignable to type 'src/types/schema/Domain'.
```

Map these types for particular schema say Domain.owner, add `| null` in the get() and set() method as shown below

```
  get owner(): string | null {
    let value = this.get("owner");
    return value!.toString();
  }
```
if !value is true you'll need to add `this.unset("owner")` and same for other property say `this.unset("property")` and else part is same everywhere, lookout for `value` datatype in `this.set("owner", Value.fromString(<string>value));`. It'll vary for diferrent datatypes say string, array, BigInt etc.
```
  set owner(value: string | null) {
    if (!value) {
      this.unset("owner");
    } else {
    this.set("owner", Value.fromString(<string>value));
    }
  }
 ```
 Run the build 
 ```
 $ yarn build
 ```
 
 Successfull Build output!
```
yarn run v1.22.17
$ graph build
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4
  Skip migration: Bump mapping apiVersion from 0.0.4 to 0.0.5
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Compile data source: ENSRegistry => build/ENSRegistry/ENSRegistry.wasm
  Compile data source: ENSRegistryOld => build/ENSRegistry/ENSRegistry.wasm (already compiled)
  Compile data source: Resolver => build/Resolver/Resolver.wasm
  Compile data source: BaseRegistrar => build/BaseRegistrar/BaseRegistrar.wasm
  Compile data source: EthRegistrarController => build/BaseRegistrar/BaseRegistrar.wasm (already compiled)
✔ Compile subgraph
  Copy schema file build/schema.graphql
  Write subgraph file build/ENSRegistry/abis/Registry.json
  Write subgraph file build/ENSRegistryOld/abis/Registry.json
  Write subgraph file build/Resolver/abis/PublicResolver.json
  Write subgraph file build/BaseRegistrar/abis/BaseRegistrar.json
  Write subgraph file build/EthRegistrarController/abis/EthRegistrarController.json
  Write subgraph manifest build/subgraph.yaml
✔ Write compiled subgraph to build/
```
Build completed: /home/dev/workspace/ans/ans-subgraph/build/subgraph.yaml

On successfull complation, deploy the sbgraph on fuji using the graph protocol
 ```
 yarn deploy
 ```
 
 Successfull deployment output:
 ```
 yarn run v1.22.17
$ graph deploy --node https://api.thegraph.com/deploy/ devilla/ans --ipfs https://api.thegraph.com/ipfs/
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4
  Skip migration: Bump mapping apiVersion from 0.0.4 to 0.0.5
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Compile data source: ENSRegistry => build/ENSRegistry/ENSRegistry.wasm
  Compile data source: ENSRegistryOld => build/ENSRegistry/ENSRegistry.wasm (already compiled)
  Compile data source: Resolver => build/Resolver/Resolver.wasm
  Compile data source: BaseRegistrar => build/BaseRegistrar/BaseRegistrar.wasm
  Compile data source: EthRegistrarController => build/BaseRegistrar/BaseRegistrar.wasm (already compiled)
✔ Compile subgraph
  Copy schema file build/schema.graphql
  Write subgraph file build/ENSRegistry/abis/Registry.json
  Write subgraph file build/ENSRegistryOld/abis/Registry.json
  Write subgraph file build/Resolver/abis/PublicResolver.json
  Write subgraph file build/BaseRegistrar/abis/BaseRegistrar.json
  Write subgraph file build/EthRegistrarController/abis/EthRegistrarController.json
  Write subgraph manifest build/subgraph.yaml
✔ Write compiled subgraph to build/
  Add file to IPFS build/schema.graphql
                .. QmYTpNtiL3aowFvApEi8Lmy71dGVft51n19o8GEumZ2UPu
  Add file to IPFS build/ENSRegistry/abis/Registry.json
                .. QmRTphmVWBbKAVNwuc8tjJjdxzJsxB7ovpGHyUUCE6Rnsb
  Add file to IPFS build/ENSRegistryOld/abis/Registry.json
                .. QmRTphmVWBbKAVNwuc8tjJjdxzJsxB7ovpGHyUUCE6Rnsb (already uploaded)
  Add file to IPFS build/Resolver/abis/PublicResolver.json
                .. QmYyNtiPDxt8Y4Zmn1uHw5xaz2j8KFrndzftMgHhtzAcny
  Add file to IPFS build/BaseRegistrar/abis/BaseRegistrar.json
                .. QmWqXtDCvMtpnV28Z5f6P6rPCaCA2d8gKnacGaohDWgALN
  Add file to IPFS build/EthRegistrarController/abis/EthRegistrarController.json
                .. Qmb6zHLHHGd1wpe2CuyVijkPrxG1M9VZKwK7aGhY1o3xKf
  Add file to IPFS build/ENSRegistry/ENSRegistry.wasm
                .. Qmb4CJV77u8GKSQRfyLoxqMfYXh1cfDiq5r29Z1FdRmUwu
  Add file to IPFS build/ENSRegistry/ENSRegistry.wasm
                .. Qmb4CJV77u8GKSQRfyLoxqMfYXh1cfDiq5r29Z1FdRmUwu (already uploaded)
  Add file to IPFS build/Resolver/Resolver.wasm
                .. QmTxsfui3km5Ez9vAi6AS7nFSH4Thi6jB17ck4HHcFVNXA
  Add file to IPFS build/BaseRegistrar/BaseRegistrar.wasm
                .. QmZ62zsok1RcmZKLSKYFTVTaBiuQwg8LdbCkcCRpEXeXBM
  Add file to IPFS build/BaseRegistrar/BaseRegistrar.wasm
                .. QmZ62zsok1RcmZKLSKYFTVTaBiuQwg8LdbCkcCRpEXeXBM (already uploaded)
✔ Upload subgraph to IPFS
```
Build completed: QmT6VYNbCfjUL2QJsNdPqU4Bnu1Fw3E5vWtfqMauA55zx5

Deployed to https://thegraph.com/explorer/subgraph/devilla/ans

Subgraph endpoints:
Queries (HTTP):     https://api.thegraph.com/subgraphs/name/devilla/ans

Subscriptions (WS): wss://api.thegraph.com/subgraphs/name/devilla/ans

