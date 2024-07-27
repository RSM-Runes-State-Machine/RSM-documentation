# RSM: Abstract Contract

RSM is a Turing-Complete State Machine for Runes Protocol.

**Time: Jun. 21 2024**

**Creator: Mikael.btc**

**Email:** [**mikael@rsm.network**](mailto:mikael@rsm.network)

**Twitter:** [**https://x.com/MikaelBTC**](https://x.com/MikaelBTC)

In RSM, Contract is a standard to define the computing capabilities of token. Contract runs in RSM and is a collection of pre-defined states and state transition functions. Each Contract needs to be defined separately by Runes developers and then will be online after being developed and supported by RSM Indexers. After the Contract is online, the token can choose to implement it to have the computing capabilities of this Contract. At this time, the token can be called application.

The Contract essentially describes a token with computing capabilities and state. The token deployed based on the contract can be called application. Contract supports token nesting, which refers to creating child tokens for the parent token to achieve modularization of tokens and expand the computing capabilities of the parent token. Contract includes three parts: operation, state, and state transition function. Operation cannot be extended by all the Contracts to ensure that all Runes tokens are compatible with each other, that is, all the Runes tokens can be used in all applications/tokens based on Contract. The state and state transition function can be extended by Contracts.

This chapter will define Operations, State, and State Transition Function that all Contracts support. The Contracts do not need to re-define them in their definition documentation.

# Operation

This chapter defines the operations and operators of token in RSM Contract. One operation could contain several operators to express slightly different semantics. Token operations and operators are not allowed to be extended, which means that all contracts cannot add, delete, or change operations and operators to ensure the compatibility of all Runes tokens, that is, all Runes tokens can be used in any token that implements RSM Contract. RSM adds four attributes: mint2s, mint3s, burn2s, and burn3s to the original Runestone struct of the Runes protocol, the corresponding four operators: mint2/mint3/burn2/burn3 are added, which are used to convert token between UTXO model and State Machine Model. The details will be introduced later.

```
struct Runestone {
  edicts: Vec<Edict>,
  etching: Option<Etching>,
  mint: Option<RuneId>,
  pointer: Option<u32>,
  mint2s: Vec<Mint>,
  mint3s: Vec<Mint>,
  burn2s: Vec<Burn>,
  burn3s: Vec<Burn>,
}
```

## 1. Etching

Etching is used to deploy token in Runes protocol. RSM keeps the compatibility with Runes protocol, that is, RSM supports all the original Etching properties of Runes protocol, and can set the implemented Contract of the token, as well as the required additional properties. Due to the byte limitation of OP_RETURN, all the new properties of the token need to be set in the Witness of the Commitment to ensure compatibility and support enough new properties. All the new structs and properties in Etching are shown in the following table.

| Attribute                    | Description                                                  | Required? | Upgradable? |
| :--------------------------- | :----------------------------------------------------------- | :-------- | :---------- |
| contract                     | The contract token impelments                                | No        | No          |
| mint2Amount                  | Can only be minted by operator "mint2", default: no limitation   | No        | No          |
| burn3ableRuneIds             | Burn3able RuneIds, default: does not support burn3           | No        | No          |
| admins                       | Administrators, can be address or application                | No        | Yes         |
| Trading.blackHolePercentage  | Black hole percentage in AMM DEX, default: 0                 | No        | Yes         |
| Trading.taxPercentage        | Tax percentage in AMM DEX, default: 0                        | No        | Yes         |
| Trading.lpFeePercentage      | Liquidity Provider Fee Percentage for AMM DEX, divisibility: 4, default: 20 | No        | Yes         |
| Trading.serviceFeePercentage | Service Fee Percentage for AMM DEX, divisibility: 4, default: 10           | No        | Yes         |
| Trading.devAddress           | Service Fee and Tax Receiver for AMM DEX, to Owner if not set        | No        | Yes         |
| DAO.started                  | Is DAO started, default: false                               | No        | Yes         |
| DAO.voteLimit                | DAO vote limit                                               | No        | Yes         |
| DAO.timeLock                 | Governance time lock, how many blocks the governance will be executed after the "can be executed" status, default: 144 | No        | Yes         |
| ext                          | Extended attribute for extension protocols                   | No        | Yes         |

The new Etching struct, as well as the newly added Trading struct and DAO struct are as follows. The attributes or structs in bold are newly added by RSM relative to the Runes protocol.

```
struct Etching {
  divisibility: Option<u8>,
  premine: Option<u128>,
  rune: Option<Rune>,
  spacers: Option<u32>,
  symbol: Option<char>,
  terms: Option<Terms>,
  contract: Option<u16>,
  mint2Amount: Option<u128>,
  burn3ableRuneIds: Option<Vec<u128>>,
  admins: Option<Vec<String>>,
  trading: Option<Trading>,
  dao: Option<DAO>,
  ext: Option<Ext>,
}
```

```
struct Terms {
  amount: Option<u128>,
  cap: Option<u128>,
  height: (Option<u64>, Option<u64>),
  offset: (Option<u64>, Option<u64>),
}
```

```
struct Trading {
  blackHolePercentage: Option<u32>,
  taxPercentage: Option<u32>,
  lpFeePercentage: Option<u32>,
  serviceFeePercentage: Option<u32>,
  devAddress: Option<String>,
}
```

```
struct DAO {
  started: Option<u8>,
  voteLimit: Option<u128>,
  timeLock: Option<u32>,
}
```

The following is an Etching example, that set the trading tax percentage and trading black hole percentage, and start DAO to open decentralized governance: all tokens need to be voted to complete governance, and the governance time lock is 288 blocks.

```
Etching {
  divisibility: 8,
  premine: 40000000000000000,
  rune: "RSM•RUNES•STATE•MACHINE",
  spacers: 528,
  symbol: "🟣",
  terms: Terms {
    amount: 200000000000,
    cap: 300000,
  },
  contract: null,
  mint2Amount: 0,
  burn3ableRuneIds: null,
  admins: null,
  trading: Trading {
    blackHolePercentage: 0.01,
    taxPercentage: 0.02,
    devAddress: "bc1pzwv3gfp8yvu0w893ekhte8h63cq0u306k62qnwgvpyy0naw7npjqvf4qhg",
  },
  dao: DAO {
    started: 1,
    voteLimit: 1000000000,
    timeLock: 288,
  },
  ext: null,
}
```

The following is another Etching example, deploying a child LP token for the parent application: AMM•DEX. The child LP token implements the LP contract: Contract-002.

```
Etching {
  divisibility: 18,
  premine: 0,
  rune: "AMM•DEX:LP•RSM•WBTC",
  spacers: 528,
  symbol: "💡",
  terms: null,
  contract: "Contract-002",
  mint2Amount: null,
  burn3ableRuneIds: ["RSM•RUNES•STATE•MACHINE","WBTC"],
  admins: null,
  trading: Trading {
    lpFeePercentage: 0.002,
    serviceFeePercentage: 0.001,
    devAddress: "bc1pzwv3gfp8yvu0w893ekhte8h63cq0u306k62qnwgvpyy0naw7npjqvf4qhg",
  },
  dao: null,
  ext: null,
}
```

## 2. Edict

The semantics of "edict" remain consistent with those in the Runes protocol. So, this article will not describe it in detail.

## 3. Burn

The burn operation is a newly introduced operation by RSM. Its semantics are to burn the token or convert the token from UTXO to the state balance in the application. The specific computing logic is defined in the implemented Contract's state transition function of the application. The operation of burn token is similar to edict, which need to specify the RuneId, amount and output of burn in OP_RETURN. RSM defines two ways to burn tokens: burn2/burn3. Tokens burned by burn2/burn3 operators cannot be re-minted at will, can only be re-minted using mint2/mint3 by the state transition function if the rules are met.

### **3.1 burn2/burn3 operator**

burn2/burn3 are operators with state transition function to compute. After the user burn2/burn3 tokens, the user's token balance will be reduced and the state of the application defined in the "to" attribute will be updated. The specific computing logic is defined in the implemented Contract's state transition function of the "to" application. The difference between the burn2/burn3 is that burn2 requires the "to" application to be the admin of the token or itself, and will reduce the circulating supply. While burn3 do not need any privilege and will not reduce the circulating supply. burn3 just convert the token from UTXO to the state of the "to" application, and the state can be controlled by the application's state transition function. The burn2ed/burn3ed token can be re-minted through the corresponding mint2/mint3 operator following the specific computing logic defined in the state transition function of each Contract.

Burn struct is a new data structure used in RSM to express burn2 and burn3 operations. As defined in the previous Runestone struct, a Runestone can contain multiple burn2 and burn3 operations. In Burn struct, the "to" and "stateTransitionFunction" attributes represent which state transition function of which token/application is triggered, thereby triggering the computing defined in the Contract and completing the state update of the token. The attribute "edicts" represents one or more UTXOs that trigger the state update. For example, when adding liquidity in AMM DEX, two token UTXOs are required. The attribute "proof" is used as a proof to help complete the computing.

```
struct Burn {
  to: RuneId,
  stateTransitionFunction: u32,
  edicts: Vec<Edict>,
  proof: Option<Proof>,
  ext: Option<Ext>,
}
```

```
struct Proof {
  id: String,
  amount: u32,
}
```

For example, using burn2 operator to burn liquidity certificate: LP token to remove liquidity in AMM DEX LP.

```
burn2 { 
  to: "AMM•DEX:LP•RSM•WBTC",
  stateTransitionFunction: "remove_liquidity",
  edicts: [ Edict {
    id: "AMM•DEX:LP•RSM•WBTC",
    amount: 200,
    output: 1,
  }],
}
```

For example, using "burn3" operator to add liquidity of RSM•RUNES•STATE•MACHINE and WBTC token in AMM DEX LP.

```
burn3 { 
  to: "AMM•DEX:LP•RSM•WBTC",
  stateTransitionFunction: "add_liquidity",
  edicts: [ Edict {
    id: "RSM•RUNES•STATE•MACHINE",
    amount: 6000,
    output: 1,
  }, 
  {
    id: "WBTC",
    amount: 1,
    output: 2,
  }],
}
```

## 4. Mint

RSM defines three operators to mint tokens: mint/mint2/mint3, where "mint" is semantically consistent with the "mint" operator of the Runes Protocol to maintain compatibility. "mint2" means to mint token from the application state. "mint3" means to mint token from the token balance held by the application, and does not change the circulating supply. mint/mint2 will increase the circulating supply. mint2/mint3 can mint token through the state transition function defined in the Contract.

### **4.1 mint operator**

The semantics of "mint" operator remain consistent with those in the Runes protocol. So, this article will not describe it in detail.

### **4.2 mint2/mint3 operator**

Like burn2/burn3, mint2/mint3 are the operators that support the state transition function. After the user mint2/mint3 token, the user will receive the token and the state of the application defined in the "from" attribute will be updated. The specific computing logic is defined in the implemented Contract's state transition function of the "from" application. The difference between the mint2 and mint3 is that mint2 requires the "from" application to be the admin of the token or itself, and will increase the circulating supply. While mint3 do not need any privilege and will not increase the circulating supply. mint3 just convert the token from the application's holdings to user's UTXO, then the user can use the UTXO in other applications.

Mint struct is a new data structure used in RSM to express mint2 and mint3 operations. As defined in the previous Runestone struct, a Runestone can contain multiple mint2 and mint3 operations. In Mint struct, the "from" and "stateTransitionFunction" attributes represent which state transition function of which token/application is triggered, thereby triggering the computing defined in the Contract and completing the state update of the "from" application. The attributes "edicts" and "proof" have the same meaning as Burn struct. For example, in the Wrapped BTC application, when wrapping BTC with mint2, the output id of the deposit transaction can be used as the "id" and the value of deposited BTC can be used as the "amount" in the "proof".

```
struct Mint {
  stateTransitionFunction: u32,
  edicts: Vec<Edict>,
  proof: Option<Proof>,
  ext: Option<Ext>,
}
```

For example, in the lending application, after deposited token, the user uses "mint2" operator to mint the certification token: RUNES•Lending:CBTC.

```
mint2 { 
  stateTransitionFunction: "mint_ctoken",
  edicts: [ Edict {
    id: "RUNES•Lending:CBTC",
    amount: 10,
    output: 1,
  }],
}
```

For example, "mint3" the exchanged token in AMM DEX.

```
mint3 { 
  stateTransitionFunction: "withdraw",
  edicts: [ Edict {
    id: "WBTC",
    amount: 3,
    output: 1,
  }],
}
```

# State

This chapter will describe the pre-defined states in the Contract. All the Contracts can use these states to describe the computing logic of the application, also, they can define their own states. The indexers should display these states to users to ensure that the states are open and consistent. All the states should be stored in Merkle Tree and the root of the tree should be displayed to the user. The states are the results computed by the RSM according to users' UTXO input and state transition function defined in the Contract. States can be variables of application, or the state balances of users in application. The states can be in application or child application. The difference between application states and application attributes is that updating attributes need to be completed through governance, but the states are computed by the public algorithms and rules defined in state transition function, and do not require governance. In RSM, there are two kinds of balance: one is UTXO Balance, which is held by addresses as same as Runes protocol; the other one is State Balance introduced by the State Machine model, which can be held by applications or addresses in application. RSM defines the following 5 states to describe the State Balance in application:

- supply, represents the total supply of the current token, equals to "token amount minted" + " token amount mint2ed" - " token amount burn2ed".
- sb2, State Balance for mint2, used to express the token amount that an address can mint2 from the current application, can be minted by the State Transition Function: w2.
- sb3, State Balance for mint3, used to express the token amount that an address can mint3 from the current application, can be minted by the State Transition Function: w3.
- sba2, State Balance of Application for mint2, used to express the total token amount that can be "mint2"ed from the current application, equals to the sum of token amount in state sb2.
- sba3, State Balance of Application for mint3, used to express the total token amount that can be "mint3"ed from the current application, the sum of token amount in state sb3 cannot be greater than the value in sba3.

# State Transition Function (stf)

State Transition Function (stf) represents the computing capabilities of the Contact, which is expressed by the attribute "stateTransitionFunction" of Mint and Burn struct, and need to be used in conjunction with mint2/mint3/burn2/burn3 operators. State Transition Function defines how the application should compute to update the states when the user performs mint2/mint3/burn2/burn3 operations. All the Contracts can define any State Transition Function that are not repeated within the Contract. The following will describe the 2 pre-defined State Transition Functions: w2 and w3 for all the Contracts in RSM. These 2 State Transition Functions cannot be changed or deleted by any Contract.

## 1. Withdraw: w2/w3

The state transition function: w2 and w3 are used to withdraw the token in state sb2 and sb3. After the user calls the application's state transition function to compute, RSM will update the value in the state sb2/sb3 of the application, to assign the tokens that can be mint2ed/mint3ed to user. w2/w3 can be used to mint these tokens to the user as UTXO with the mint2/mint3 operator. w2/w3 will eventually update the value in the state sb2/sb3.

For example, in the decentralized stablecoin application, user can withdraw the USD-pegged stablecoin: "Runes•Stablecoin:DUSD" after staking their collateral.

```
mint2 { 
  stateTransitionFunction: "w2",
  edicts: [ Edict {
    id: "Runes•Stablecoin:DUSD",
    amount: 10000,
    output: 1,
  }],
}
```

## 2. Computing Failure

The state transition function may be failed. After failure, the indexer needs to add the token amount in the UTXOs corresponding to the state transition function to the state: sb2 or sb3 of the user.
