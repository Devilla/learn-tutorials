# Introduction

A Lending Marketplace provides a secure, flexible, open-source foundation for a decentralized loan marketplace on the Polygon blockchain. It provides the pieces necessary to create a decentralized lending exchange, including the requisite lending assets, repayments, and collateral infrastructure, enabling third parties to build applications for lending.

# Prerequisites

- [Polygon](https://docs.polygon.technology/) is a blockchain that is EVM compatible.
- Familiarity with [Solidity](https://docs.soliditylang.org/) and [Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) are recommended.

# Requirements

- [MetaMask](https://metamask.io/) is a browser-based blockchain wallet that can be used to store any kind of digital assets and cryptocurrency.Extension can be installed from [here](https://metamask.io/download)

- [Node.js](https://nodejs.org/en/) enables the development of fast web servers in JavaScript by bringing event-driven programming to web servers.Make sure to have NodeJS 12.0.1+ version installed.

- [ReactJS](https://nodejs.org/), a Javascript library for building user interfaces.

# Introduction to User Interface with React and web3

Clone this [Git Repository](https://github.com/Devilla/cryptolend.ui)

```text
git clone https://github.com/Devilla/cryptolend.ui.git
```

Browse the project directory:

```text
cd cryptolend.ui
```

Install dependencies required for the project:

```text
npm i
```

Run the server to be able to access the application UI in your browser:

```text
npm start
```

In the metamask supported browser create a [Custom RPC](https://medium.com/stakingbits/setting-up-metamask-for-polygon-matic-network-838058f6d844) in the Networks as follows:

- Network Name: Polygon
- New RPC URL: https://rpc-mainnet.matic.network or
- ChainID: 137
- Symbol: MATIC
- Block Explorer URL: https://polygonscan.com/


Open the lending dapp in browser and go to the `/myloans` path and connect to Polygon Network in the Metamask extention. Now feel free to go throgh the various features of thhe lending marketplace like creating a loan request on `/request` path and loan offer on `/offer`, which can also be browsed from the Navigation bar.
