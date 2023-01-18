---
title: Cosmos
date: '2021-05-02'
tags: ['crypto', 'infra']
draft: false
summary: 'A short dive into cosmos'
---

# Cosmos

Most blockchains have:

1. Shared execution layer
2. Smart contracts

## What do I mean when I say Cosmos?

- Not Cosmos Hub
- Not $ATOM
- Cosmos SDK: Tendermint Consensus + IBC Modules + other stuff

### Philosophy

- Instead of shared execution layer, you want application specific chains.
- Benefit 1: You have control
  - Precompiles - code that denotes complicated computations cheaper on chain. If you don't have a precompile, you need to use the evm. E.g. cryptography. Defined in clients Cacn technically use .com domain as ENS, but this is extremely expensive to do DNSSEC onchain since cryptography isn't on chain. Might cost $1000 some money to verify
  - Novel consensus/Protocol Design. Osmosis wants to prevent MEV from happening; they want to use threshold encryption to do so. Thus, Osmosis can embed this in their chain
- Benefit 2: Blockspace Monopoly
  - Vertical Scaling brings problems like state bloat, computation power. Solana drops Merkle trees from client design to improve L1 scaling.
  - Cosmos Approach: Instead of scaling vertically, you can create your own blockchain and scale horizontally. If each application has their own separate throughput, Polygon doesn't get rekt bc people are playing sunflower games.

## IBC

- relies on instant finality
- TCP/IP - not a bridge
- easy to add Cosmos based blockchains and other blockchains with easy way to create light client proofs, Relayer
- peg zone
- **Relayer**: trustless way to send data from one blockchain to another. Think of it as a cable that can transmit data from one chain to another. Security comes from the light client proof, relayers have no verification.
  End Goal: Each wallet is its own relayer. Relayers also get transaction fees in the future. Currently, relayers are subsidized via the Interchain foundation(Ethereum foundation equivalent)
- No flash loans, flash mints, flash anything via IBC bc that requires atomicity.
- Can do token transfers, cross chain swaps

## IBC Research

Quick notes

- Bc of the blockchain trilemma{decentralization, scalability, security}, applications will use multiple blockchains to achieve the trilemma
- To communicate between blockchains

1. Deploy contract on Ethereum: meant to communicate with validators
2. Send package to validators
3. Validators communicate with solana blockchain
   Eg If transferring USDT, you lock USDT on ethereum contract and mint synthetic USDT on solana blockchain
   **Validator:** Who runs the validators between chains?

## IBC

- Handle Authentication, ordering data packets between blockchains, can use
- Recall above, the package needs to be validated
- Do you need to use IBC + a bridge?
  Mental model: IBC is a validator. (Who runs the validator?)
  Goal: Manage communication between two modules. This involves sending/receiving packets with establish connection

## How IBC works

- Properties of IBC: Data confidentiality + Legibility: Send as little data to validator as possible + validator needs to understand the data
- Reliability: Relayer + blockchain works independently, there is a chance of dropping packets. This is managed by a sequence number per packet.
- Flow Control: Depends on ledger, IBC does not have overriding rule
- Authentication: Whenever IBC is to be used, data must be first on the originating blockchain. - IBC handler(validator?) verifies the data
- Statefulness: Three parts - clients, connection, channel
- Clients: info about state of both blockchains
  connection: identifiers/data structures agreed by both blockchains
- Channel: info about encoding info, state, sequence numbers
- Multiplexing: smart contracts can use single IBC connection. Generally one to one but IBC supports one to many and many to one
- Relayer - keeps watch of state chain from origin chain(ie Ethereum in example above)

## Use Case

Why switch networks when purchasing insurance on other chain?

- If vault and pool are connected with IBC, user can buy insurance from any chain while spending from mainnet wallet.

## Questions?

Confirmation time is 15 seconds on ETH, Bitcoin is 10 minutes, can we still use IBC?
Need to wait 10 minutes?

What about reorgs? There's never true finality on proof of work chains right?
Solved via validators?

Learning - Learn more about cosmos
Document - Publish technical articles, explainer articles, based on learning
Buidling - play with cosmos ibc, a bit tough scotty doesn't believe in us, Spinning up dapp chain, running a relayer

# Questions

## Subnets vs Cosmos

- There is an expectation that you share security with Avalanche. Problem is that shared security isn't free, you need to pay avalanche validators for this.
- Tendermint is novel: instant finality + 2/3 network validators needs to be aligned. Instant finality is huge because it simplifies multichain consensus. In a proof of work blockchain, by the virtue of a 51% attack, blocks are not final since you can end up with uncle blocks[reorgs](https://medium.com/dragonfly-research/dr-reorg-or-how-i-learned-to-stop-worrying-and-love-mev-2ee72b428d1d)
  - This is bad for bridges because a transaction where you burn a token to mint it via a bridge to a different chain can be reverted due to a reorg.
    - bridge UX gets killed
  - Multihop dexes can take a very long time because you need to wait for probabilistic finality.
- Are there Oracles natively in Cosmos?
  - No current oracles, but can built oracle chain
