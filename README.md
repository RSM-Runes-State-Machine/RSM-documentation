# RSM: Runes State Machine
RSM is a Turing-Complete State Machine for Runes Protocol.

**Time: Jun. 21 2024**

**Creator: Mikael.btc**

**Email:** [**mikael@rsm.network**](mailto:mikael@rsm.network)

**Twitter:** [**https://x.com/MikaelBTC**](https://x.com/MikaelBTC)

# RSM Introduction

This document is intended to manage Runes State Machine (RSM) and all contracts. The following table shows the current status of them.

| RSM Contract | Creator    | Description                                                  | Status  | Time         |
| :----------- | :--------- | :----------------------------------------------------------- | :------ | :----------- |
| Contract-XXX |            | More RSM Contracts                                           |         |              |
| ...          |            | ...                                                          |         |              |
| Contract-016 |            | An Asset Vesting Contract                                    |         |              |
| Contract-015 |            | An Aggregator Contract for Lending                           |         |              |
| Contract-014 |            | An Aggregator Contract for Decentralized Exchange            |         |              |
| Contract-013 |            | A Launchpad Contract for BRC-100 Assets                      |         |              |
| Contract-012 | Mikael.btc | An Asset Issuance Contract for Administrator                 |         |              |
| Contract-011 |            | A Verification Contract for Bitcoin Layer 2                  |         |              |
| Contract-010 |            | A Relay Contract among Runes and EVM Compatible Blockchains  |         |              |
| Contract-009 |            | A Decentralized Exchange Contract for Perpetual Futures      |         |              |
| Contract-008 |            | An Automated Liquidity Contract for Stablecoin               |         |              |
| Contract-007 | Mikael.btc | A Decentralized Lending Contract                             |         |              |
| Contract-006 | Mikael.btc | A Decentralized Collateral-backed Stablecoin Contract        |         |              |
| Contract-005 | Mikael.btc | A Decentralized Airdrop Contract                             |         |              |
| Contract-004 | Mikael.btc | A Liquid Staking/Restaking Contract for Runes, BTC, BRC-20 and BRC-100 |         |              |
| Contract-003 | Mikael.btc | A Relay Contract among Bitcoin, Runes, BRC-100 and BRC-20    |         |              |
| [Contract-002](https://github.com/RSM-Runes-State-Machine/RSM-documentation/blob/main/Contracts/Contract-002.md) | Mikael.btc | An Automated Liquidity Contract for Runes Assets             | Relesed |  Jul. 8 2024 |
| Contract-001 | Mikael.btc | A Decentralized On-Chain Governance Contract                 |         |              |
| RSM          | Mikael.btc | RSM: Runes State Machine                                     | Relesed | Jun. 21 2024 |

As we all know, Ordinals Theory, Runes, BRC-20 and other meta-protocols based on Bitcoin have brought a lot of imagination to the development of Bitcoin ecosystem through "on-chain declaration, off-chain computing" mechanism. And a large number of non-fungible tokens (NFTs) and fungible tokens have been issued, but the development of decentralized applications such as DeFi (Decentralized Finance) on Bitcoin is still lagging behind. This document explores a new paradigm for the computing capabilities of Runes Tokens: Runes State Machine (RSM). Besides asset issuance, RSM enables Runes protocol with the complex computing capabilities required for DeFi and other decentralized applications, to create the potential for decentralized applications based on Bitcoin Layer 1.

https://docs.ordinals.com/

https://docs.ordinals.com/runes/specification.html

Runes State Machine (RSM) is a Turing-Complete State Machine for Runes Protocol, serves as the runtime environment for the pre-defined Contracts, which can be deployed and run in RSM. RSM is a Runes Improvement Protocol. Contract is a collection of states and state transition functions. When deploying a Runes token, developer can choose which contract to implement. RSM introduces decentralized computing capabilities to the Runes protocol, endowing Runes tokens with programmability, enabling complex decentralized applications like DeFi for Runes Protocol besides token issuance. Each token/application can define its computing logic and internal state through implementing a Contract. Additionally, RSM proposes an application/token modular approach: nesting, which provides a theoretical foundation for the infinite extension of application computing capabilities. Essentially, RSM describes a token with computing capabilities and state, where the token's behaviors are extended from Runes protocol. The computing capabilities, states, and state transition rules are defined in the Contract. Tokens compliant with the Runes Protocol standard can be used in any Contract, and the process is entirely decentralized, without custody of user assets.

RSM is based on two models: UTXO model and state machine model, which greatly extends the computing capabilities of Bitcoin. The operation of token satisfies the UTXO model, and the computation conforms to the state machine model. And RSM provides new operations: burn2/burn3 and mint2/mint3, so that token can be converted between UTXO model and state machine model, decentralized and safely.

RSM introduces many concepts, please read the explanation below firstly.

| Term                      | Description                                                  |
| :------------------------ | :----------------------------------------------------------- |
| Runes State Machine (RSM) | A Turing-Complete state machine to run Contracts for Runes Protocol |
| Contract                  | A standard to define the computing capabilities of a token, running on RSM, is a collection of pre-defined states and state transition functions |
| Token/Rune                | Created by etchings, implements different Contract, with different computing capabilities |
| Application               | The Rune/Token that implements a Contract can be called Application |
| State                     | A situation of a token depending on previous inputs and causes a reaction on following inputs, defined in the Contract |
| State Transition Function | Computes the next state of the token given the current state and inputs, defined in the Contract |
| Token Nesting             | Multiple runes/tokens can be created under one rune/token, to complete multiple independent computing logics, called child rune/token |
| Governance of Rune/Token  | The process of updating rune/token attributes, creating child runes/tokens |
| Oracle                    | Indexers can serve as the Oracle Verifier to verify the off-protocol proof data submitted by the user |

![rsm_arc](https://github.com/RSM-Runes-State-Machine/RSM-documentation/assets/143789621/0d3ce463-ec93-4221-8b89-4c63439b0336)

RSM is open, and developers are welcome to participate in building contracts, developing applications, and jointly building the Bitcoin ecosystem. You can join the RSM Community: https://t.me/RSMCommunity.

Also, if you like RSM, you can donate to: **bc1pzwv3gfp8yvu0w893ekhte8h63cq0u306k62qnwgvpyy0naw7npjqvf4qhg**.

# Design Principles

The RSM is designed based on three principles: security, extension, and consistency. The RSM Contracts must also conform to these three principles.

## 1. Security

Security is the foundation of all RSM decentralized applications. RSM design needs to consider two aspects: user asset security and application security, including:

- No custody of user assets
- Trustless
- Censorship-resistant
- Permissionless
- Token/Application's computing logic is open and transparent
- Token/Application supports decentralized governance

## 2. Extension

RSM should have great extension, and can support the logic of currently known decentralized applications and possible future innovations. RSM achieves extension and modularity through the support for token nesting. Applications deployed on RSM can create child applications to utilize the computing capabilities of the child applications, to implement the modularity of building application.

## 3. Consistency

RSM must meet the consistency requirements, ensuring any indexer can compute a completely consistent state based on the on-chain data in OP_RETURN and the computing logic of the Contract. The RSM Contract consists of two parts: state and state transition function. The state transition function must align with token UTXO operations, which are based on the UTXO model and cannot be extended, to ensure the compatibility and consistency of all Runes tokens. The state is expressed using the state machine model, that requires a rigorous and unambiguous description of the computing logic in the Contract to guarantee the consistency of all the indexers' computing.

# Concepts

This chapter will define and explain some concepts in RSM to help developers to understand better.

## 1. Runes State Machine (RSM)

Runes State Machine (RSM) is a Turing-Complete State Machine for the Runes Protocol, is a Runes Improvement Protocol. RSM aims at endowing Runes Tokens with programmability to enable complex decentralized applications such as DeFi (Decentralized Finance) for Runes Protocol. RSM is a State Machine and serves as the runtime environment for Contracts. RSM can update the internal states of token based on the pre-defined states and state transition functions in the Contract, to complete the necessary computing for complex decentralized applications (dApps). RSM is not an independent Virtual Machine for the tokens; rather, RSM is the runtime environment for each Runes token. Tokens do not need to be transferred to RSM for computing, and the state updates rules are pre-defined in the Contract and transparent. The computing process based on RSM is entirely decentralized and does not require custody of user assets.

## 2. Contract and Token

In RSM, the Contract is a standard that describes the states and state transition functions of the token. The token is an instance created after a Contract is deployed to the Bitcoin network, with the complex computing capabilities and states defined in the Contract. The computing capabilities of a token are described by the state transition functions in the Contract. Without adding child tokens, a token cannot have computing capabilities not described in the Contract. The added child tokens can only have the computing capabilities of the implemented Contract too. Otherwise, the public indexers cannot verify the states of the token, resulting in the states inconsistency. Token is called Rune in Runes protocol. In RSM, the token that implements the Contract can also be called application. Application is essentially a token with computing capabilities and states.

## 3. Token Nesting

Runes tokens based on RSM can be nested, that is, one token can be created under another token, which is called child token/application. The name of the child token starts with "parent token name:", and multiple tokens can be created under one token to introduce multiple independent computing logics. For example, in the classic AMM DEX scenario, multiple LP child tokens need to be created in one AMM DEX token, like "AMM•DEX:LP•RSM•WBTC".

## 4. State

Besides the UTXO model, RSM also introduces the state machine model to expand the computing capabilities of token. Token can have all the states defined in the implemented Contract. For example, a token can have states: reserve0 and reserve1 to represent the assets amount held by the AMM DEX Liquidity Pool; a token can have a state: "Total Staked Amount" to represent the staked asset amount by the users in the Yield Farming Pool.

The conversion of UTXO and state is completed through the newly introduced operators: burn2/burn3/mint2/mint3 and the state transition function. The state transition function is used to represent the specific computing rules to update the state and UTXO. For example, user A sends 10 token1 assets to Application2 through the burn3 operator to trigger the computing of the state transition function in Application2 and complete the conversion from UTXO to state. Currently, Application2 owns the UTXO, and holds these 10 token1 in the state; then, RSM can distribute these 10 token1 by changing the internal state of Application2 according to the pre-defined logic in the implemented Contract. User B who receives token1 can mint the internal state balance in the form of UTXO to his own wallet through the operator mint3, to complete the conversion from state to UTXO.

## 5. State Transition Function

State Transition Function is defined in the Contract, that computes the next state of the token given the current state and UTXO inputs, and is the smallest unit of RSM computing capability. State Transition Function is used in conjunction with operators: burn2/burn3/mint2/mint3 to complete the conversion between UTXO and state. Among them, burn2/burn3 defines how the internal states of the token update after the corresponding UTXOs are transferred, and mint2/mint3 defines that after the internal states update, the corresponding UTXOs will be transferred to the user.

## 6. Decentralized Governance of Token

RSM introduces a governance contract: Contract-001, which can govern Runes tokens that implement other contracts. And after the token starts DAO (Decentralized Autonomous Organization), the governance needs to be completed through decentralized voting. The governance of token includes: update the attributes of token and child tokens, and deploy child tokens. Token governance is on-chain governance. After the on-chain voting is passed, the token should be notified through the state transition function, then RSM will automatically execute the governance after the time lock.

## 7. Privileges

RSM introduces two kinds of roles: owner and admin. The address to etch Runes token is called owner. The owner of all child tokens is the owner of the parent token. The admin is managed by the owner, and the admin cannot manage other admins. The scope of the owner's authority includes the admin's. And the privileges of the owner and the admin are strictly limited. They cannot censor users and can only do: govern tokens that have not started DAO. Admin can be an address or an application or a child application. A token and its child tokens are each other's admin by default, also the token is its own admin by default, no additional settings are required, but child tokens are not admins of each other.

The part of token needed to be mint by "mint2" can only be allocated by the token/child token logic, and the token/child token need to be the admin of the token, also "burn2" operator has similar logic.

## 8. Contract Parameters

When defining Contract, 2 parameters need to be defined: openAsChild and onlyChild, please find the explanation in the following. These parameters are defined when defining the Contract and do not need to be set when deploying the token. For example, in the AMM DEX LP Contract: Contract-002, openAsChild: true, onlyChild: No.

- openAsChild, Can the contract be deployed by anyone as child token
- onlyChild, Can the contract only be deployed as child token

## 9. Deploy Token

In RSM, there are two ways to deploy token: one is to deploy directly using the Etching of the Runes Protocol, and the other one is to deploy through the governance Contract: Contract-001. The first one is used to deploy parent tokens and child tokens configured not to require governance, and the other one is used to deploy child tokens that require governance.

## 10. Burn Token

The burn operation is a newly introduced operation by RSM, which will burn the assets or transfer the assets corresponding UTXOs to the state of the token. Similar to the definition of mint operation, there are 2 burn operators: burn2 and burn3, which correspond logically to mint2 and mint3 respectively. No extra configuration is required, all tokens support these 2 burn operators.

- burn2: Burn to Whitelist, according to the Contract's preset rules, after burn2 assets to token, the user's balance will be reduced, the state of the token will also be updated accordingly, the circulating supply will be reduced. In practice, the logic such as remove liquidity in AMM DEX can be realized by burn2.
- burn3: Burn to State, burn3 will reduce the user's token balance and increase the balance of the "to" token. In practice, burn3 can be used with mint3 to complete swapping token and adding liquidity in AMM DEX.

## 11. Mint Token

Besides "mint" operator in Runes Protocol, RSM provides 2 new mint operators: mint2 and mint3, which are used to mint tokens in different scenarios, to convert an address's state in the token to UTXO. When deploying a token, developers need to set the token amount that can be minted by the "mint2" operator.

- mint: Mint from Public, public mint, as same as the mint in Runes Protocol, anyone can mint. After minting the circulating supply of token will be increased.
- mint2: Mint from Whitelist, the token state records the token amount that users can mint, anyone can mint2 assets under the rules. After mint2 the circulating supply of token will also be increased.
- mint3: Mint from State, mint3 the state balance of users in token, anyone can mint3 assets under the rules. After mint3 the circulating supply of token will not be increased.

## 12. Trading Tax and Deflation

RSM introduces a new token trading mechanism: trading tax and deflation. Token can set the trading tax percentage, trading tax receiver and trading black hole percentage. These settings only take effect when trading in decentralized exchanges based on AMM. Normal edict and mint operations will not trigger trading tax and deflation.

## 13. Anti-MEV

In order to prevent MEV attacks, RSM has proposed an effective way. When the state machine model is converted to the UTXO model, it needs to be confirmed after at least 1 block. For example, a user uses token1 to swap token2 in AMM DEX on block X. At this time, token2 will be allocated to the user's state balance. If the user wants to use this state balance, he must first use mint3 to convert the state balance to UTXO. This mint3 operation can only be successful on the block >= X+1. This method can effectively prevent MEV attacks such as sandwich attacks.

## 14. Oracle

Oracle is a common requirement for blockchain to interact with off-chain parties, and have been well implemented and applied on blockchains such as Ethereum. Without an oracle, the smart contract on the blockchain would be limited entirely to the on-chain data. But compared with blockchain, RSM has very special characteristics and advantages. RSM not only has the computing capabilities as the blockchain do, but also relies on the off-chain indexers to complete the computing. At the same time, the off-chain indexers are able to communicate with the other blockchains or meta-protocols directly, but the blockchain cannot do this, which means that the indexers can verify any data off-chain or on-chain by sufficient proof data to meet the Oracle requirements of RSM. For example: verify a transfer of BTC assets, verify the ETH price on a certain block of Ethereum, etc. In other words, in the RSM, the Oracle has a new paradigm: proof and verification, where the user submits proof data, and the indexers serve as the Oracle Verifier to verify the off-protocol proof data submitted by the user, and the independent Oracle service is not needed.

In RSM, the operator: burn2/burn3/mint2/mint3 natively supports the proof attribute, which is used to submit the off-protocol proof data. RSM Indexers can verify the proof data to ensure the consistency and correctness of the state, the proof can be Transfer Proof, Merkle Tree Proof, Zero-Knowledge Proof, Price Proof, etc., and can be used in many scenarios such as Stablecoin, Lending, Bitcoin Layer 2, Relay, Airdrop, etc.

# Developers

Developers are very important role in RSM. The purpose of RSM is to make more developers to develop decentralized applications based on the Bitcoin Layer 1. If a developer wants to develop a decentralized application based on RSM, he should first design the corresponding Contract, then make the Contract public and obtain indexer support, and finally develop the application to implement the new Contract.

Welcome developers to develop decentralized applications based on RSM. The development of the Bitcoin is driven by you. If you have any questions, you can ask in the RSM Community: https://t.me/RSMCommunity.

# Donate

Of course, if you think that RSM has inspired or helped you, or you have seen and recognized our efforts to develop Bitcoin ecosystem, you can donate to: bc1pzwv3gfp8yvu0w893ekhte8h63cq0u306k62qnwgvpyy0naw7npjqvf4qhg.

# Reference

[1] Runes, https://docs.ordinals.com/runes.html

[2] Finite-State Machine, https://en.wikipedia.org/wiki/Finite-state_machine

[3] brc-20 Protocol, https://domo-2.gitbook.io/brc-20-experiment/

[4] BRC-100 Protocol: An Extensible Decentralized Computing Protocol based on Ordinals Theory, https://docs.brc100.org/

[5] BRC-101 Protocol: A Decentralized On-Chain Governance Protocol for BRC-100 Protocol Stack, https://docs.brc100.org/brc-100-extension-protocols/brc-101-protocol

[6] BRC-102 Protocol: An Automated Liquidity Protocol for BRC-100 Assets, https://docs.brc100.org/brc-100-extension-protocols/brc-102-protocol

[7] BRC-104 Protocol: A Liquid Staking/Restaking Pool Protocol for BRC-100, BRC-20, Runes and BTC, https://docs.brc100.org/brc-100-extension-protocols/brc-104-protocol
