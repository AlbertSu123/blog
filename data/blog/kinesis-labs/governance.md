---
title: Artificial Growth - Giving out $100k in 60 days to grow faster
date: '2023-01-26'
tags: ['kinesis', 'crypto', 'reflections', 'defi']
draft: false
summary: 'A postmortem of the Kinesis Labs Liquidity Mining Governance Proposals.'
---

The easiest way to grow your defi protocol is to start liquidity mining. Normally, projects will distribute their token
https://commonwealth.im/evmos/discussion/4752-bootstrap-liquidity-on-evmos-dapps

https://commonwealth.im/evmos/discussion/6844-temperature-check-start-liquidity-mining-campaign-for-gravity-celer-and-axelar-assets

https://commonwealth.im/evmos/discussion/7017-passed-proposal-for-axelar-network-celer-network-and-gravity-bridge-liquidity-mining

## Appendix

Temperature check - Start Liquidity Mining Campaign for Gravity, Celer, and Axelar assets

Hi Evmos Community!

We are planning to make a proposal next week: Liquidity mining campaigns for stablecoins from Celer Network, Gravity Bridge, and Axelar Network.

The goal of this proposal is to bring stablecoin liquidity back to Evmos. In the past, Kinesis Labs has reached over 255k USD worth of stablecoins in TVL without any incentives, with over 11,000 transactions and 400 unique liquidity providers. All of this was organic liquidity from the Evmos community, there were no funds or deals done to bootstrap this liquidity.

However, with the recent nomad hack, both Kinesis and Evmos need a way to bootstrap liquidity to jump start the Evmos DeFi ecosystem. We propose the following plan to bring stablecoin liquidity back to Evmos:

33k Evmos tokens to be distributed to the Axelar Network Pool over 6 months as soon as they finish their deployment to Evmos mainnet. Please note that the assets in this pool are to be decided as we are still unsure what assets will get adoption on Evmos, but the pool will likely consist of blue chip stablecoins.

33k Evmos tokens to be distributed to the Celer Network Pool over 6 months as soon as the proposal passes. Please note that the assets in this pool are to be decided as we are still unsure what assets will get adoption on Evmos, but the pool will likely consist of blue chip stablecoins.

33k Evmos tokens to be distributed to the Gravity Bridge Pool over 6 months as soon as this proposal passes. Please note that the assets in this pool are to be decided as we are still unsure what assets will get adoption on Evmos, but the pool will likely consist of blue chip stablecoins.

We are requesting a total of 99k Evmos for this proposal to be linearly distributed over 6 months.

All of these funds will be distributed to liquidity providers on Kinesis through the minichef contract, no funds will ever reach an EOA or multisig - these funds are not for the Kinesis Team nor the Axelar, Celer, or Gravity teams. Additionally, the only fee on Kinesis is the 0.04% swap fee which goes entirely to LPs, and Kinesis does not currently have a token or any other value extraction mechanism. We are building this project as members of the Evmos ecosystem as a public good.

The multisig that controls these funds will be a 4 out of 6 consisting of:
Nic Z, Co-Founder of Tharsis (Evmos Core Team)
Akash Khosla, Co-Founder of Tharsis (Evmos Core Team)
Liam DiGregorio, Head of Business Development at Tharsis (Evmos Core Team)
Mo Dong, Co-Founder of Celer Network
Deborah Simpier, Co-Founder and CEO of Althea (Gravity Bridge)
Georgios Vlachos, Co-Founder of Axelar

The reasoning behind these members is that these would be the key stakeholders for this proposal as they are members of Tharsis, the team building Evmos as well as one representative from each bridge that will be launching. There will be

Additionally, Celer will match this with a minimum of 1m in Celer stablecoin liquidity, which will be deposited into the Celer stablecoin pools.

We’ve been working closely with the Axelar Network, Celer Network, Gravity Bridge teams on this initiative. If there’s any questions, the Kinesis + Axelar + Celer + Gravity bridge teams will be happy to answer down below.

Background on Axelar Network
Summary:
Axelar network is built from the ground up using Cosmos SDK to interoperate between Cosmos chains, EVM chains, Bitcoin, and other PoS and PoW chains. Satellite is the cross-chain asset transfer app built on top of the network and has amassed over $37m TVL and 100k cross-chain transactions since launching in February. It has also recently become the official canonical bridge for Osmosis and plans to further expand its reach into the Cosmos and EVM ecosystem.

Satellite currently supports Avalanche, Polygon, Terra, Osmosis, Moonbeam, Fantom, Ethereum and Cosmos Hub with plans to support all major EVM chains over the next couple months.

Satellite connects all Cosmos chains via IBC and its GCP protocol with multiple ecosystems.
It’s built on top of Axelar’s “decentralized overlay network” that connects different blockchain stacks. Axelar serves as the translation layer that allows Cosmos assets to flow freely to all Axelar-interconnected networks and back. It is permissionless, meaning that anyone can join, submit transactions, relay, and validate.

Axelar has been audited by Ackeeblockchain, Cure53, NCC, Oak Security, Commonprefix labs and independent researchers and is also dedicating at least 5% of AXL supply to an insurance fund/program that will help recuperate any stolen funds to ensure the safety of its users.

Security:

Axelar believes that bridging solutions should be at least as secure as the chains being bridged. Axelar’s solution is having a Cosmos chain, one with a similar validator set to other chains. Additionally, they are heavily invested in securing their smart contracts and in applying cutting-edge cryptography.

The Axelar network isolates functionalities in modules at the Cosmos SDK level, e.g., one module for bitcoin, another for EVM chains, another for IBC transfers, etc.

Each module handles the respective transactions from source and destination chains and are isolated from each other and messages can only be passed through a router module. If there is an issue with one or all chains, a special transfer command that can be supplied by the governance multisig can pause the router module.

There is also minimal reliance on smart contracts and are only used for key-rotations and mint/burn operations.

The mint/burn contracts can also be freezed in case of a compromise and have a rate-limit functionality that allows to minimize how much funds can be stolen in case of an attack. This is not enabled on the mainnet at the moment, but can be considered along with other “access control” policies to minimize attacks.

There will also be an insurance fund/program of at least 5% of the AXL supply to compensate users in case of stolen funds. Axelar has also been audited by Ackeeblockchain, Cure53, NCC, Oak Security, Commonprefix labs and independent researchers and gone through a dozen audits.

Costs:

Axelar currently has a set fee on each transfer that covers gas costs on source + destination chain. Axelar foundation subsidizes any increase in the destination gas cost if need be and is also considering subsiding fees further from its treasury.

Transactions are also batched, reducing the gas costs for the users significantly. Axelar also has active implementation of threshold signatures that can be used to replace individual validator signatures and further save gas costs. While not currently deployed in EVM contracts, this is just one of the many code-level gas optimizations that Axelar will ship in the future.

Users only pay the fee in the asset they transfer on the source chain, and all other fees (relaying / intermediary / destination chains) are covered by services around the network.

App:
https://satellite.axelar.network/

Background on Celer Network
Summary:
Celer Network is a multi-chain operating system built with Cosmos SDK. Celer released the world’s first EVM-supported inter-chain message SDK. and has a track record of no hacks for any product since Celer originated in 2018.

Celer cBridge is a token bridge application built on top of Celer that supports 45 different tokens across 21 different blockchains and layer-2 rollups. cBridge has $400M TVL and has processed more than $4B of cross-chain volume for 105K unique users with a daily volume of $50M-$60M across 21 different blockchains.

cBridge comes with two security models that every user can choose between, the Cosmos-consensus security model and an optimistic-rollup-style delay buffer security model. The consensus model is security that of a validator-based bridge, while the optimistic rollup style also has a delay buffer to allow an independent watchtower service to double check the message.

Users have single-click UX with fast confirmation of the bridge between source and destination chains and with 2,500-3,500 daily active users comfortable with using Celer Network.

Security:

Cosmos-consensus Security Model:
Celer SGN is running as a Cosmos chain with the same level of liveness assumption which is very high like other Cosmos chains. Celer provides an L1-blockchain level security like Cosmos or Polygon as a Proof-of-Stake chain built on Tendermint with CELR as the staking asset. If a validator acts maliciously, their CELR will be slashed.

Optimistic-rollup-style delay buffer Security Model:
Inter-chain dApps can maintain full security with an optimistic-style delay buffer. Instead of instantly processing a message through the consensus model, one can inject a mandatory delay buffer and run an independent watchtower service to double-validate the message on the source chain. If the watchtower service detects any inconsistency, it can prevent the message from being processed before the delay expires.

For transactions larger than a certain amount, two-transactions are needed to make the bridge happen: a “commit” transaction that will trigger the time buffer and then after the time buffer, a “confirm” transaction. The watchtower system also monitors the bridge for small transactions as well to detect and prevent any unexpected smart contract bugs. There is also rate-limiting for token bridging as the last-resort to worst-case scenario.

In addition, Celer cBridge smart contract has been heavily audited and also has a $2M ImmuneFi bug bounty. Celer Network also has a track record since 2018 of no hacking event happening for any product they have built.

Costs:
cBridge charges the gas fee on the destination chain to cover the operation costs of validators. The fee is not related to the size of the transaction, unlike many other bridging solutions.

Celer also has no plans to change the fee structure and remains committed to supporting inter-chain apps where IBC is not available. Unlike other bridges that plan on changing to percentage-based fees later, Celer will only charge a flat fee based on the size of the message passed to provide revenue for validators.

There are also no platform fees charged by the Celer Network for token asset bridging.

App:
https://cbridge.celer.network/#/transfer
Background on Gravity Bridge

Summary:
The Gravity Bridge blockchain is the trustless, neutral bridge between Ethereum and the Cosmos ecosystem. The principal focus of the Gravity community is on providing the most effective and secure bridge possible, instead of a DeFi application on the local chain.

Gravity Bridge protocol focuses on creating a simple bridge design that focuses on providing better security by eliminating design complexity and using non-upgradeable contracts which have been audited by Certick, Least Authority and Cod4rena.

The Gravity Bridge protocol provides strong censorship resistance guarantees and the bridge will remain live as long as the chain remains live, i.e. with more than 2/3 of validators active.

Currently supports Ethereum and has plans to support additional EVM chains before the end of 2022.

While there is not currently a set-aside for an insurance fund, one could easily be set aside from the community pool which contains 50% of $GRAV token distribution which could also contain a collection of assets.

Security:

Gravity Bridge is trustless, decentralized and highly efficient and offers the closest approximation of IBC security possible. It is highly decentralized with all funds directly controlled by validators with slashing to deter validators from acting in bad faith.

The Gravity Bridge design means that the only way to attack the bridge is to go through the validators themselves. This results in a trustless bridge that costs less to operate than a custodial one. It is not possible for a validator to avoid signing a specific transaction or for a relayer to avoid relaying it. Anyone can submit evidence of a signature over a non-protocol message to slash a validator which makes it extremely difficult for validators to steal funds.

The Gravity Bridge, like all bridges, has two components. A Solidity contract on Ethereum and a CosmosSDK module on the Gravity Bridge blockchain.

The solidity contract that holds funds on Ethereum is merely 580 lines of code, which makes it compact and easy to review. It has been audited by three independent teams (Certik, Least Authority, and Code4rena), and it is not upgradable. It requires no trusted parties and can be temporarily paused by governance vote in case of a possible security breach. As a final layer of protection the Gravity Bridge blockchain performs a series of integrity checks that will halt the chain on any unexpected behavior.

While there’s nothing set for an insurance fund at the moment, the $GRAV token holders can choose to use some funds from the large community pool to help recuperate stolen funds.

Costs:

Robust relaying reward systems combined with reduced gas costs creates extremely fast & reliable transfers with a decentralized, self-funded, relayer community

Gravity Bridge batches withdraw transactions. This works like a rollup on Ethereum. Executing many transactions in a single shared context is far more efficient than doing so individually. This process provides huge efficiency gains, up to 96% reduction in gas cost.

Once bridge traffic has matured, tx fees could be enabled by the community to sustain the community pool. These tx fees are payable in a wide variety of tokens, not solely in GRAV. Other mechanisms, like a portion of the inflation, could be used to fund the community pool.

App:
https://spacestation.zone/

Background on Kinesis
About Kinesis:
Kinesis is a decentralized automated market maker (AMM) on the EVMOS blockchain, optimized for trading pegged value crypto assets with minimal slippage such as stablecoins. Through Kinesis, all traders can swap their desired stablecoins between EVM compatible blockchains and Cosmos chains.

Kinesis enables efficient, swift, and low-slippage swaps for traders and high-yield for liquidity providers. We believe in collaboration with other Cosmos projects, in maximizing the interoperability of Kinesis, and in building out the stablecoin ecosystem for Cosmos.

Kinesis has no governance token, no admin fees, and no value extraction mechanism. We are building this project as members of the Evmos ecosystem as a public good.

{
"title": "Liquidity Mining Proposal for Axelar, Celer Network, and Gravity Bridge on Kinesis",
"description": "Link to Proposal: ",
"recipient": "evmos1mx9nqk5agvlsvt2yc8259nwztmxq7zjq50mxkp",
"amount": [
{
"denom": "aevmos",
"amount": "99000000000000000000000"
}
],
"deposit": "64000000000000000000aevmos"
}
