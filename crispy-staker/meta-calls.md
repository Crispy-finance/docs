# Meta calls

This page documents the different meta call methods available on the _Crispy Staker_ contract. A meta call is a method who's primary purpose is to trigger sub calls within the contract.

## Multicalls

Often people need to execute multiple actions on a single smart contract like when creating stake ladders for example, this is often impractical and expensive as users are required to issue multiple transactions, one at a time.

Luckily the `Multicall` pattern is gaining in popularity among smart contract developers and is seeing more and more use. We too have decided to add multicall functionality to the main _Crispy Staker_ contract so that users can batch many calls to the contract into a single transaction, once a multicall transaction confirms, all the contained calls are also confirmed.

{% hint style="info" %}
If a call contained within a multicall reverts, following calls will not be attempted and the entire multicall will revert.
{% endhint %}

<details>

<summary><strong>Basic multicall using the <code>ethers</code> library</strong></summary>

```javascript
// `staker` is assumed to be an ethers.Contract instance of the
// Crispy Staker contract
const stakeAmount = '1000000000000' // 10k HEX (8 decimals)
const stakeDays = 365 * 3
const stakerEncode = staker.interface.encodeFunctionData

// encoding the sub calls
const depositCall = stakerEncode('pull', [stakeAmount])
const createStake = stakerEncode('createStake', [stakeAmount, stakeDays])

// submit multicall 
await staker.multicall([depositCall, createStake])
```

</details>

### Basic multicall

**Method definition:** `multicall(bytes[] calldata _data) returns (bytes[] memory)`

The `multicall` method is a standard multicall method. It accepts a list of abi encoded sub-calls as the `_data` parameter and returns the return data for the individual calls as a list. Unlike some other multicall implementations the `multicall` method does not accept any ETH.

### Expiring multicall

**Method definition:** `expiringMulticall(uint256 _expireAfter, bytes[] calldata _data) returns (bytes[] memory)`

The `expiringMulticall` method is very similar to the basic `multicall` method in that it also accepts a list of sub-calls as `_data` and returns the list of return data. However it has an extra parameter `_expireAfter` which allows the caller to specify a block timestamp after which the call will no longer be valid and revert if submitted. Useful for submitting a batch of time sensitive calls which should not be executed if confirmed too late.

### Fee protected multicall

**Method definition:**  `feeCappedMulticall(uint256 _maxCreateFee, uint256 _maxRolloverFee, bytes[] calldata _data) returns (bytes[] memory)`

Similar to the basic `multicall` and the `expiringMulticall` methods. However this multicall method is useful for submitting batches of fee sensitive calls as it will revert if the stake creation fee is above `_maxCreateFee` or if the stake rollover fee is above `_maxRolloverFee` if one of the two fees is irrelevant to the calls being batched an arbitrarily high maximum can be set for the fee that is irrelevant. If both fees are irrelevant use the basic `multicall` method

### Fee protected and expiring multicall

**Method definition:** `feeCappedExpiringMulticall(uint256 _expireAfter, uint256 _maxCreateFee, uint256 _maxRolloverFee, bytes[] calldata _data) returns (bytes[] memory)`

This method combines both the `feeCappedMulticall` and `expiringMulticall` methods into one and offers both protections in a single call. Useful for time sensitive calls which should also only be run below certain fee values. If either the fee maximums or the `_expireAfter` timestamp the call will revert before running any additional calls.

## No fee call

**Method definition:** `noFeeCall(bytes calldata _data) returns (bytes memory)`

Allows the use of the _Crispy Staker_ contract with fees (create and rollover) temporarily set to 0% for the context of that transaction. After the `noFeeCall` is complete the fees are reverted to their initial value.

{% hint style="info" %}
The `noFeeCall` method is only available to the `owner` address. Learn more about the capabilities of the `owner` address on the [access control](access-control.md) page
{% endhint %}
