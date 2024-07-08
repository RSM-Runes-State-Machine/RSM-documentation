# Contract-002: An Automated Liquidity Contract for Runes Assets

**Time: Jul. 8 2024**

**Creator: Mikael.btc**

**Email:** [**mikael@rsm.network**](mailto:mikael@rsm.network)

**Twitter:** [**https://x.com/MikaelBTC**](https://x.com/MikaelBTC)

This article defines the RSM Contract: Contract-002.

# 1. Summary

Contract-002 is an automated liquidity contract for Runes assets, that defines an automated market making method based on "constant product formula" (x*y=k) for a pair of Runes tokens.

# 2. Abstract

The Contract-002 is the first contract for application in RSM. The Contract-002 defines an automated market making method based on "constant product formula" (xy=k) for a pair of Runes tokens. There are three roles in the contract: liquidity provider, trader, and liquidity pool. The liquidity provider is responsible for providing the two tokens of the pair to the liquidity pool. These two tokens can be tokens based on any RSM contract such as Contract-002, Contract-004, Contract-006, etc., or normal tokens that do not implement contract. Traders can exchange one token of the pair for another in the liquidity pool. The liquidity pool is based on the AMM algorithm of "constant product formula" (xy=k), which will automatically provide liquidity to traders and is implemented by the RSM indexers. Through the cooperation of the three roles, Contract-002 can realize a completely decentralized, trustless, self-custody, censorship-resistant, and permissionless token exchange on Bitcoin Layer 1.

# 3. Concepts

There are seven important concepts in the Contract-002: UTXO, Swap, Liquidity Pool, Refund, Fees & Rewards, Black Hole Address and Divisibility. These seven concepts cover the core algorithm of this contract. The application deployed based on Contract-002 is called: Liquidity Pool. Swapping tokens, adding/removing liquidity are all interacting with the Liquidity Pool.

## 3.1 UTXO

As same as Runes, OP_RETURN and UTXO are the basis of the RSM. Users inscribe Runestone to the OP_RETURN with State Transition Function (stf), parameters, and the corresponding Rune/Token UTXO, to notify the indexer to compute the states.

The burn2/burn3/mint2/mint3 computing logic has two steps:

- Inscribe the Runestone to the OP_RETURN.
- Indexers compute UTXO and states changes.

The entire process is completed on Bitcoin Layer 1. The transfer of burn2/burn3 UTXO does not actually transfer the assets to the receiver, but only completes the conversion from UTXO to application state. In this contract, the transfer of user assets must meet the preset rules, and no custody of user assets is required.

## 3.2 Swap

Swap refers to using one Runes token to exchange for another in the Liquidity Pool. In Contract-002, there is no need to wait for the matching of buy orders and sell orders like the orderbook marketplace does, the trading can be completed immediately. The trading process is completed in the liquidity pool, which will be introduced in the next section. The liquidity pool is based on the Automated Market Maker (AMM) algorithm: "constant product formula" (x*y=k), which automatically provides liquidity to trading users and completes trades immediately. Traders can use the burn3 Runestone with state transition function (stf): sw to exchange one token for another.

1. The trader inscribes burn3 Runestone with stf: sw. The Runestone contains the tokens to be paid, as well as the restrictions: the received token minimum amount: ext.amountOutMin and the deadline timestamp: ext.deadline.
2. If the current block timestamp is greater than the set deadline timestamp: ext.deadline, the computing fails.
3. Compute the token amount that the trader can receive based on the "constant product formula" (x*y=k) algorithm. Taking into account the black hole percentage, tax percentage and fees, the constant formula should be: $`  {x}_{0\cdot }{y}_{0}=({x}_{0}+{x}_{in}-{x}_{in}\cdot {p}_{tbhp}-{x}_{in}\cdot {p}_{ttp}-{x}_{in}\cdot {p}_{lpfp}-{x}_{in}\cdot {p}_{sfp})\cdot ({y}_{0}-{y}_{out}) `$, then the received token amount should be: $`  {y}_{out}={y}_{0}-\frac {{x}_{0}\cdot {y}_{0}} {{x}_{0}+{x}_{in}-{x}_{in}\cdot {p}_{tbhp}-{x}_{in}\cdot {p}_{ttp}-{x}_{in}\cdot {p}_{lpfp}-{x}_{in}\cdot {p}_{sfp}} `$. If the amount is less than the minimum amount set by the ext.amountOutMin, the computing fails, otherwise the computing is successful, and the received token enters state sb3.  
4. If the computing is successful, the black hole and tax parts will enter state sb3. The fees have two parts: one part is allocated to the liquidity provider, and the other part is allocated to the preset dev address, which are determined by the parent application attributes trading.lpFeePencetage and trading.serviceFeePencetage respectively. The trading.lpFeePencetage part will change the corresponding state reserve0 or reserve1, which means to be allocated to all liquidity providers, and trading.serviceFeePercentage will enter the state sb3 of the dev address.
5. If the computing is successful, the state reserve0 or reserve1 corresponding to $` {x}_{in} `$ need to increase $`  {x}_{in}-{x}_{in}\cdot {p}_{tbhp}-{x}_{in}\cdot {p}_{ttp}-{x}_{in}\cdot {p}_{sfp} `$, the state reserve0 or reserve1 corresponding to $` {y}_{out} `$ need to decrease $` {y}_{out} `$ .
6. If the computing fails, the burn3ed tokens enters state sb3.
7. Finally, if the computing is successful, the trader can use mint3 Runestone with state transition function (stf): w3 defined in the RSM Abstract Contract to mint the received token to himself from the state sb3, to complete the conversion from application state to UTXO.

## 3.3 Liquidity Pool (LP)

The application deployed by Contract-002 is called Liquidity Pool, which is a trading venue for the pair of Runes tokens of the Pool. The trading is based on the Automated Market Maker (AMM) algorithm: "constant product formula" (x*y=k). The contract has two state transition functions (stf) to manage liquidity: add liquidity and remove liquidity. After adding liquidity, the token is converted from UTXO to state sba3 of the application, and can only be transferred to user's states through the contract's preset rules when removing liquidity or swapping tokens. Then the user can mint3 the states to himself to complete the conversion from state to UTXO. No one can manipulate the token at will. The whole process is completely decentralized and there is no third-party custody of user assets.

### 3.3.1 Add Liquidity

The user adds two tokens to the Pool by Runestone with two burn3s and stf: al, then receives LP token as the liquidity certification.

1. The liquidity provider inscribes the Runestone with two burn3s and stf: al. The Runestone contains the two tokens to be added to the Pool, as well as the restrictions: the deposited minimum amount of the two Tokens: ext.amount0Min (Amount0 Min) and ext.amount1Min (Amount1 Min), and the deadline timestamp: ext.deadline (deadline).
2. If the current block timestamp is greater than the set deadline timestamp: ext.deadline , the computing fails.
3. Compute the actual tokens amounts that should be added to the Pool. If there are no tokens in the Pool, both sent tokens will be added to the Pool. Otherwise, compute $` {y}_{deposited\_ optimal}=\frac {{Reserve}_{y}} {{Reserve}_{x}}\cdot {x}_{deposited\_ desired} `$, if $` {y}_{deposited\_ optimal}\leq {y}_{deposited\_ desired} `$, the actual deposited tokens amounts ($` {x}_{deposited} `$ and $` {y}_{deposited} `$) should be $` {x}_{deposited\_ desired} `$ and $` {y}_{deposited\_ optimal} `$. Otherwise, they should be $` {x}_{deposited\_ optimal}=\frac {{Reserve}_{x}} {{Reserve}_{y}}\cdot {y}_{deposited\_ desired} `$ and $` {y}_{deposited\_ desired} `$. If any of the actual deposited tokens amounts is less than the set minimum amount or greater than the amount of UTXO, the computing fails, otherwise the computing succeeds. Among them, $` {x}_{deposited\_ desired} `$ and $` {y}_{deposited\_ desired} `$ are the tokens amounts in the UTXO, $` {x}_{deposited} `$ and $` {y}_{deposited} `$ represent the actual tokens amounts that are added to the pool.
4. If the computing fails, the burn3ed tokens enter state sb3.
5. If the computing is successful, the user can receive LP Token as the certification for adding liquidity, that is, updating the state sb2. The amount computing rules are divided into two types. If the Pool does not have any token: $` {s}_{minted}=\sqrt {{x}_{deposited}\cdot {y}_{deposited}}-{1}^{-15} `$, otherwise, $` {s}_{minted}=min(\frac {{x}_{deposited}} {{x}_{starting}}\cdot {s}_{starting},\frac {{y}_{deposited}} {{y}_{starting}}\cdot {s}_{starting}) `$. When adding initial liquidity, 1e-15 LP Token will be sent to the black hole address, that is, the state: sb2 of the black hole address will be updated, to prevent extreme situations from occurring.
6. If the computing is successful, the state reserve0 and reserve1 need to increase $` {x}_{deposited} `$ and $` {y}_{deposited} `$.
7. If the computing is successful, the user can use mint2 Runestone with stf: w2 defined in the RSM Abstract Contract to mint the received LP token to himself from the state sb2, to complete the conversion from state to UTXO.
8. Finally, if any of the two tokens is left after the computing, it will enter the state sb3 to refund to the user. The user can re-mint through mint3 Runestone with stf: w3 defined in the RSM Abstract Contract, to himself to complete the conversion from state to UTXO.

### 3.3.2 Remove Liquidity

The user uses the burn2 Runestone with stf: rl to burn the liquidity certification: LP token to the Pool to remove the liquidity, and receive the two tokens by the corresponding proportions.

1. The liquidity provider inscribes the burn2 Runestone with stf: rl. The Runestone contains the liquidity certification: LP token, as well as restrictions: the withdrawn minimum amount of two Tokens: ext.amount0Min (Amount0 Min) and ext.amount1Min (Amount1 Min), and the deadline timestamp: ext.deadline (deadline).
2. If the current block timestamp is greater than the set deadline timestamp: ext.deadline , the computing fails.
3. Compute the two withdrawn tokens amounts after removing liquidity according to the formula: $` {x}_{withdrawn}=\frac {{s}_{burned}} {{s}_{starting}}\cdot {x}_{starting} `$, $` {y}_{withdrawn}=\frac {{s}_{burned}} {{s}_{starting}}\cdot {y}_{starting} `$, if any of computed amounts is less than the set withdrawn minimum amount, the computing fails, otherwise the computing is successful and the state sb3 will be updated.
4. If the computing fails, the burn2ed tokens enter state sb2.
5. If the computing is successful, the state reserve0 and reserve1 need to decrease $` {x}_{withdrawn} `$ and $` {y}_{withdrawn} `$.
6. If the computing is successful, the user can use mint3 Runestone with stf: w3 defined in the RSM Abstract Contract to mint the two tokens after removing liquidity to himself from the state sb3, to complete the conversion from state to UTXO.

## 3.4 Refund

Since the Contract-002 is based on UTXO, the tokens amount in the UTXO is fixed, there will be some cases where tokens are left, and the left tokens will be refunded to the user by updating the state sb3, such as adding liquidity, there is a high probability that one token will be refunded to the user.

## 3.5 Fees & Rewards

In Contract-002, two fees will be charged when the users swap tokens: Liquidity Provider Fee and Service Fee. These two fees are set through the properties of the parent application: trading.lpFeePercentage and trading.serviceFeePercentage respectively. trading.lpFeePercentage is added directly to reserve0 or reserve1, which serves as rewards for all LP providers. The other trading.serviceFeePercentage is added to the state sb3 of the receiving address: trading.devAddress, to reward the application developer.

## 3.6 Black Hole Address

The attribute: trading.blackHolePercentage (trading black hole percentage) is defined in the RSM Abstract Contract, which represents the percentage allocated to the black hole when the tokens are traded in the AMM DEX. The black hole is represented in this contract as an address that no one can control, mainnet: 1BitcoinEaterAddressDontSendf59kuE, testnet/signet: mxxD8soCEyzP3ZJReqKUPsTYC7ZCfeLsvo. And the allocation to the black hole means allocating to the sb3 state of the address.

## 3.7 Divisibility

This contract may have accuracy issues when computing the states.  In order to ensure that all RSM indexers can have completely consistent results after computing, the computing needs to be carried out according to the following rules.

- Every operator without special meaning are processed as divisibility is 30.
- The operators of tokens, even in the middle of the formula, needs to be consistent with the token divisibility, such as the computing of trading black hole, trading tax, liquidity provider fee, service fee.
- The divisibility of the state needs to be consistent with the divisibility of the corresponding token, for example reserve0 divisibility should equal to the token0 divisibility.
- The divisibility of 4 attributes: trading.blackHolePercentage, trading.taxPercentage, trading.lpFeePercentage, and trading.serviceFeePercentage are 4. If exceeded, the deployment will fail.
- The states without corresponding tokens, need to be processed as divisibility is 18.
- If the computing result exceeds divisibility, the excess part needs to be discarded.

# 4. Parameters

The parameters are defined in the contract and do not need to be set when deploying the application.

- openAsChild: Yes
- onlyChild: Yes

# 5. Operation

This chapter defines the operations and operators of Contract-002. All operations and operators in RSM Abstract Contract will be inherited, and new operations and operators are not allowed to be added. However, in order to express more semantics, the reserved extension attribute: ext can be used.

## 5.1 Etching

For the Contract-002, all attributes in RSM Abstract Contract will be inherited. Since Contract-002 can be deployed as a child application by anyone using etching, some special attributes require permission restrictions and can only be set by the application owner. Therefore, these attributes can only be set in the trading attribute of the parent application, not in the child application. These attributes include trading.lpFeePercentage (Liquidity Provider Fee Percentage), trading.serviceFeePercentage (Service Fee Percentage), trading.devAddress (Service Fee and Tax Receiver). At the same time, these attributes in the parent application can be upgraded. If the first two attributes are not set, the corresponding default values will be used. If trading.devAddress is not set, the service fee receiver will be the owner of the current application. If the owner is changed, the service fee receiver will also be changed.

| Attribute            | Description                                                  | Required? | Upgradable? |
| :------------------- | :----------------------------------------------------------- | :-------- | :---------- |
| lpFeePercentage      | Liquidity Provider Fee Percentage, should reference the value in parent application | No        | Yes         |
| serviceFeePercentage | Service Fee Percentage, should reference the value in parent application, default | No        | Yes         |
| devAddress           | Service Fee and Tax Receiver, should reference the value in parent application, default: owner address | No        | Yes         |

For example, deploy the parent application: AMM•DEX, with LP fee: 0.2% and service fee: 0.1%.

```
Etching {
  divisibility: 8,
  premine: 400000000,
  rune: "AMM•DEX",
  spacers: 528,
  symbol: "🔥",
  terms: Terms {
    amount: 1000,
    cap: 600000,
  },
  contract: null,
  mint2Amount: 0,
  burn3ableRuneIds: null,
  trading: Trading {
    lpFeePercentage: 20,
    serviceFeePercentage: 10,
    devAddress: "bc1pzwv3gfp8yvu0w893ekhte8h63cq0u306k62qnwgvpyy0naw7npjqvf4qhg",
  },
  dao: DAO {
    started: 1,
    voteLimit: 1000000001,
    timeLock: 10000001,
  },
  ext: null,
}
```

For example, deploy the child application: AMM•DEX:LP•RSM•WBTC based on the Contract-002 for the parent application: AMM•DEX.

```
Etching {
  divisibility: 18,
  premine: 0,
  rune: "AMM•DEX:LP•RSM•WBTC",
  spacers: 528,
  symbol: "🔥",
  terms: null,
  contract: "Contract-002",
  mint2Amount: null,
  burn3ableRuneIds: ["RSM•RUNES•STATE•MACHINE","RUNES•WRAPPING•BTC"],
  trading: null,
  dao: DAO {
    started: 1,
    voteLimit: 1000000001,
    timeLock: 10000001,
  },
  ext: null,
}
```

### 5.1.1 Deployment Constraints

This chapter is used to describe the constraints when the contract is deployed to the Bitcoin network through Runestones. Applications can only be deployed successfully if they meet these constraints. Besides the basic format constraints, the Contract-002 also needs to meet some other logic constraints for successful deployment, as shown in the following table. The special one is "burn3ableRuneIds" attribute, which represents the two RuneId of the two tokens in the current LP, and cannot be duplicated in all child applications based on Contract-002 of the parent application. For example, if "burn3ableRuneIds": ["RSM•RUNES•STATE•MACHINE","RUNES•WRAPPING•BTC"] already exists, the child Contract-002 application deployment with "burn3ableRuneIds": ["RSM•RUNES•STATE•MACHINE","RUNES•WRAPPING•BTC"] or ["RUNES•WRAPPING•BTC","RSM•RUNES•STATE•MACHINE"] will fail.

| Attribute        | Constraint                                                   |
| :--------------- | :----------------------------------------------------------- |
| divisibility     | should be 18                                                 |
| premine          | should be 0                                                  |
| terms            | should not be set                                            |
| mint2Amount      | should not be set                                            |
| burn3ableRuneIds | should be 2, not exist among all the Contract-002 child applications of the parent application |
| admins           | should not be set                                            |
| trading          | should not be set                                            |
| dao.started      | should be 1                                                  |
| dao.voteLimit    | should be greater than 1,000,000,000                         |
| dao.timeLock     | should be greater than 10,000,000                            |
| ext              | should not be set                                            |

# 6. State

This chapter is used to describe the states in the Contract-002. Similarly, Contract-002 inherits all the states from the RSM Abstract Contract, and the following 5 new states will be added:

- token0, represents the token with index 0 in the "burn3ableRuneIds" array of the current application/pool.
- token1, represents the token with index 1 in the "burn3ableRuneIds" array of the current application/pool.
- reserve0, represents the token0 amount in the current application/pool.
- reserve1, represents the token1 amount in the current application/pool.
- supply, represents the LP token supply of the current application/pool, equals to "the LP token amount increased by adding liquidity" - "the LP token amount burned by removing liquidity".

# 7. State Transition Function (stf)

This chapter defines the extension state transition functions (stf) of the Contract-002. Of course, since the Contract-002 inherits from RSM Abstract Contract, all the stf defined in RSM Abstract Contract are still valid in the Contract-002. Contract-002 adds 3 new stf: cop: sw (Swap), al (Add Liquidity), rl (Remove Liquidity), and in the ext attribute corresponding to the stf, adds the deadline attribute to represents the deadline timestamp of the computing (in seconds), which will be introduced in detail below.

## 7.1 Swap: sw

As described in the Swap of Concepts chapter, the burn3 Runestone with stf: sw is used to complete the exchange of tokens. ext.amountOutMin (Amount Out Min) represents the minimum received token amount, and ext.deadline represents the deadline timestamp of this computing (in seconds). If the received token amount is less than the minimum amount: ext.amountOutMin, or the confirmation time on the chain is greater than ext.deadline, the computing will fail. After the token swapping is successfully, the states: sb3, reserve0, reserve1 will be updated.

```
burn3 { 
  to: "AMM•DEX:LP•RSM•WBTC",
  stateTransitionFunction: "sw",
  edicts: [ Edict {
    id: "RUNES•WRAPPING•BTC",
    amount: 10000000,
    output: 1,
  }],
  ext: {
    amountOutMin: 1000000000000,
    deadline: 1722383999
  },
}
```

After the swapping is successful, the user can use mint3 Runestone with stf: w3 defined in the RSM Abstract Contract to mint the received token to himself from the state sb3.

```
mint3 { 
  stateTransitionFunction: "w3",
  edicts: [ Edict {
    id: "RSM•RUNES•STATE•MACHINE",
    amount: 1200000000000,
    output: 1,
  }],
}
```

## 7.2 Add Liquidity: al

As described in the Add Liquidity of Concepts chapter, the burn3 Runestone with stf: al is used to add liquidity. ext.amount0Min and ext.amount1Min respectively represent the deposited token minimum amount with index 0 (state token0) and index 1 (state token1) in the "burn3ableRuneIds" array, and cannot be greater than the corresponding token amount in the UTXO. ext.deadline represents the deadline timestamp of this computing (in seconds). If any of the deposited amount of two tokens is less than the minimum amount, or the confirmation time on the chain is greater than ext.deadline, the computing will fail. After adding liquidity successfully, the states: sb3, reserve0, reserve1, sb2 will be updated.

```
burn3 { 
  to: "AMM•DEX:LP•RSM•WBTC",
  stateTransitionFunction: "al",
  edicts: [ 
    Edict {
      id: "RSM•RUNES•STATE•MACHINE",
      amount: 600000000000,
      output: 1,
    }, 
    Edict {
      id: "RUNES•WRAPPING•BTC",
      amount: 100000000,
      output: 2,
    }
  ],
  ext:{
    amount0Min: 500000000000,
    amount1Min: 90000000,
    deadline: 1722383999
  },
}
```

After adding liquidity successfully, the user can use mint2 Runestone with stf: w2 defined in the RSM Abstract Contract to mint the application token (LP Token) as the liquidity certification from state sb2.

```
mint2 { 
  stateTransitionFunction: "w2",
  edicts: [ Edict {
    id: "AMM•DEX:LP•RSM•WBTC",
    amount: 260000000,
    output: 1,
  }],
}
```

## 7.3 Remove Liquidity: rl

As described in the Remove Liquidity of Concepts chapter, the burn2 Runestone with stf: rl is used to remove liquidity. ext.amount0Min and ext.amount1Min respectively represent the withdrawn token minimum amount with index 0 (state token0) and index 1 (state token1) in the "burn3ableRuneIds" array. ext.deadline represents the deadline timestamp of this computing (in seconds). If any of the withdrawn amount of two tokens is less than the minimum amount, or the confirmation time on the chain is greater than ext.deadline, the computing will fail. After removing liquidity successfully, the states: sb3, reserve0, reserve1 will be updated.

```
burn2 { 
  to: "AMM•DEX:LP•RSM•WBTC",
  stateTransitionFunction: "rl",
  edicts: [ Edict {
    id: "AMM•DEX:LP•RSM•WBTC",
    amount: 60000000,
    output: 1,
  }],
  ext:{
    amount0Min: 100000000000,
    amount1Min: 20000000,
    deadline: 1722383999
  },
}
```

After the liquidity is successfully removed, the user can use mint3 Runestone with stf: w3 defined in the RSM Abstract Contract to mint the two withdrawn tokens from state sb3 to himself.

```
mint3 { 
  stateTransitionFunction: "w3",
  edicts: [ 
    Edict {
      id: "RSM•RUNES•STATE•MACHINE",
      amount: 150000000000,
      output: 1,
    }, 
    Edict {
      id: "RUNES•WRAPPING•BTC",
      amount: 30000000,
      output: 2,
    }
  ],
}
```

# 8. Reference

[1] Uniswap v2 Core, https://docs.uniswap.org/whitepaper.pdf

[2] Runes, https://docs.ordinals.com/runes.html

[3] RSM: Runes State Machine, https://docs.rsm.network/

[4] BRC-102 Protocol: An Automated Liquidity Protocol for BRC-100 Assets, https://docs.brc100.org/brc-100-extension-protocols/brc-102-protocol

[5] BRC-100 Protocol: An Extensible Decentralized Computing Protocol based on Ordinals Theory, https://docs.brc100.org/
