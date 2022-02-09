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

First, let's deploy the baseRegstrar contracts with 'avax' as top level domain from `deploy/basregistrar/00_deploy_base_registrar.js`

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

Deploy Reverse Registrar contracts for `.avax` as top-level domain for registration from `deploy/basregistrar/10_deploy_reverse_registrar.js`.

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

```
    transactions.push(await ens.setSubnodeOwner(ZERO_HASH, sha3('reverse'), deployer))
    transactions.push(await ens.setSubnodeOwner(namehash.hash('reverse'),sha3('addr'),reverseRegistrar.address))
    console.log(`Waiting on settings to take place of reverse registrar ${transactions.length}`)
    await Promise.all(transactions.map((tx) => tx.wait()));
}

module.exports.tags = ['reverse-registrar'];
module.exports.dependencies = ['registry', 'public-resolver']
```
Deploy `ETHRegistrarController` with  `StablePriceOracle` contracts for '.avax' as root domain

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
