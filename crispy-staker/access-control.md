---
description: Overview of how different parts of an application are access restricted
---

# Access control

To ensure that only the right parties can end, transfer and rollover stakes the _Crispy Staker_ contract implements some straight forward access control.

Certain, special methods like setting fee percentages and updating metadata URIs are restricted to the `owner` address.

## General Stake Control

The minted _Crispy Stakes_ are completely under the control of their respective owner. The owner of a token can transfer or end their stake at any time. The owner may also rollover the stake but only once it reaches maturity i.e. when all days have been served.

{% hint style="info" %}
The ability to rollover stakes early was blocked due the fact that rolling over a stake retains the token's ID while changing the underlying principal and stake duration. This could've lead to people maliciously rolling over their stakes just before being sold on platforms like OpenSea.

Early rollover would've also been subject early end stake penalties.
{% endhint %}

{% hint style="danger" %}
Do not offer to buy **mature** stakes on platforms like OpenSea or Rarible as their owner may rollover the stake just before accepting your order, leaving you with a different underlying stake.
{% endhint %}

Beyond the direct owner, approved operators will also have access to the tokens they are approved to transfer under the [ERC721](https://eips.ethereum.org/EIPS/eip-721) standard. For example, if a certain account is approved as an operator by calling `setApprovalForAll(account, true)` that account will be able to end, transfer and rollover any _Crispy Stake_ that the approving account owns.

{% hint style="success" %}
Approved operators are ultimately under the control of the stake owner, approval to any account is always opt-in and owners can also always revoke previously given approvals.
{% endhint %}

Under the [ERC721](https://eips.ethereum.org/EIPS/eip-721) standard any approved operator would already be able to transfer the stakes to an address of their choosing anyway. To allow integrating contracts to be more efficient the _Crispy Staker_ contract gives approved operators direct access to stakes to save them the extra step of having to first transfer the stake to themselves.

## The Owner Role

The main _Crispy Staker_ staking contract can only have a single address set as its `owner`. To inspect which address is the current owner the contract has an `owner()` method which can be called by anyone.

{% hint style="success" %}
The `owner` address is **not an admin key**.

The `owner` address **cannot:** Modify the contract's code, steal underlying stakes, steal _Crispy Stakes_ or generally steal liquid Hex tokens from accounts which have approved the contract.
{% endhint %}

The main ability of the contract "owner" consists of changing the create and rollover fee percentage as well as keeping the stake metadata URIs up-to-date. In theory the owner can set an arbitrary percentage for the two fees from 0% - 100%.

To limit the potential abuse of the fee setting ability the contract includes a maximum fee parameter which can be queried via the `getMaxFee()` method. Once set, no fee will ever be allowed to exceed the set `maxFee`. Furthermore the `maxFee` cannot be increased to allow higher fee percentages to be set, once the `maxFee` value is lowered it can never be raised above the newly set value again.

Beyond the `maxFee` cap the contract also allows users to specify a maximum acceptable fee for their transactions in case the fee is changed while their transaction is being confirmed. The ability to arbitrary change fees is restricted even further by the accessory `CrispyController` contract while will be set as the `owner` address of the _Crispy Staker_ contract. The _Crispy Controller_ ensures that any changes to the fee percentages or `owner` address must be announced publicly, on the blockchain at least 14 days before going into effect.
