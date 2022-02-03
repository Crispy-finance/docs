---
description: Introduction to the Crispy Staker system
---

# Introduction

{% hint style="info" %}
The _Crispy Staker_ documentation assumes that the reader is at least somewhat familiar with the Hex token and how its stakes typically function. Learn more at [hex.com](https://hex.com)
{% endhint %}

_Crispy Staker_ is a system of smart contracts which enables hex stakers to create so called "wrapped" or "tokenized" stakes. These are stakes that are trustlessly backed 1:1 by real hex stakes, generating the same yield while adding capabilities to stakes.

Stakes are tokenized as NFTs, this is done according to the [ERC721 standard](https://eips.ethereum.org/EIPS/eip-721) so that stakes can automatically be compatible with a wide array of existing NFT tools and dApps. Unlike art NFTs, stake tokens should derive their value mainly from the underlying hex stake that is being controlled by the asset.

Beyond simply being able to transfer and sell stakes, _Crispy Staker_ enables the batching of any stake related operation (transferring, creating, ending, rolling over) into a single transaction when necessary, not only improving the user experience but also saving its users a lot in gas fees (depending on the operations being batched).

_Crispy Staker_ also lends itself very well to other developers who wish to build hex based applications. This is because _Crispy Staker_ was built __ with efficiency, security and composability in mind.

To learn more about _Crispy Staker_ feel free to visit our [landing page](https://crispy-staker.com) or proceed to the technical documentation on the following pages if you want to learn more about the inner workings and how to integrate with our contract.
