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
