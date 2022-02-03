---
description: Token accounting method utilized by Crispy Staker
---

# Free balance accounting

In order to create the underlying Hex stakes the _Crispy Staker_ contract needs to receive and manage liquid Hex tokens. Typically to receive and use ERC20 tokens contracts require the caller to have approved the contract in advance so that it may properly account the tokens it is receiving from users and subsequently use them. Normally this pattern is good as it allows EOAs (non-smart contract user wallets) to directly use contracts.

## Basic concept

The _Crispy Staker_ contract uses a different pattern (the "free balance" pattern) to keep track of ERC20 tokens. It is inspired by the accounting pattern that [Uniswap V2 liquidity pools](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol) employ. The pattern has however been expanded so that EOAs can also directly use the contract without having to interact with an additional periphery contract.

If a method (like the create stake methods) need to use some ERC20 tokens it assumes that they have already been transferred to the contract.

This is what is referred to by "free balance": the ERC20 balance the contract already holds.&#x20;

## Advantages

This is particularly useful for other contracts which want to interact with the _Crispy Staker_ contract as they can just directly transfer the tokens they need to deposit to the contract without having to approve the contract first.

This also makes the contract more efficient, particularly when different actions are batched together. Instead of sending tokens back and forth between the staking contract and the user before and after every action it can simply use the free balance it has received from a previous action.

By combining free balance accounting with multicalls we can allow EOAs to directly interact with the contract by providing a `pull` method which executes the `transferFrom` method on the token.

<details>

<summary>Classic vs Free Accounting example</summary>

A user wants to end a stake, partially keeping some of the stake proceeds and restaking the rest:

**Contract operations using the classic pattern:**

1. End stake: stake ended, proceeds transferred to the caller with `transfer`
2. Start stake: new stake principal gets transferred from caller to the contract using `transferFrom`, stake started

Total actions: 1x transfer, 1x transferFrom, 1x end stake, 1x start stake

**Contract operations using the free accounting pattern:**

1. End stake: stake ended
2. Start stake: stake started
3. Push remaining free balance: contract transfers remainder to caller

Total: 1x transfer, 1x end stake, 1x start stake

**Result:**

The free balance accounting approach saves an entire transferFrom call in this example. While there is a small added cost for initiating the push as it is a separate step, overall the savings from the 1 transfer more than make up for it.

</details>

## Accounting fees

To ensure that a contract can also store tokens it needs to keep track of its _held balance_. This is a value that keeps track of the total tokens the contract holds. The contract's free balance is therefore actually: `total_token_balance - held_balance`. To store earned fees the _Crispy Staker_ contract simply increases its held balance. Conversely to withdraw earned fees the contract just decreases its held balance to release tokens into the contract's free balance.&#x20;
