---
description: Welcome to the PWN developer documentation.
---

# Welcome!

PWN protocol is a decentralized and permissionless set of smart contracts. With PWN protocol you can **unlock the full potential of your assets** such as in-game collectibles, art or ERC-20 tokens - all of that without the risk of liquidation and with fixed interest. In this developer documentation you can learn about the contracts and the protocol architecture.

### Our architecture

![](.gitbook/assets/PWN\_contracts-Architecture.drawio.png)

{% tabs %}
{% tab title="PWN" %}
PWN contract serves as the only interface between a user and the protocol.&#x20;
{% endtab %}

{% tab title="PWN Vault" %}
PWN Vault is used for storing assets.
{% endtab %}

{% tab title="PWN LOAN" %}
PWN LOAN is used to store information about LOANs. LOAN is an ERC-1155 token representing a loan in the PWN protocol.
{% endtab %}

{% tab title="MultiToken" %}
MultiToken is a solidity library that wraps transfer, allowance and balance check calls for ERC20, ERC721 & ERC1155 tokens. Unifying the function calls used within the PWN context, so we don't have to worry about handling these token standards individually.
{% endtab %}
{% endtabs %}

### Smart contracts

Here you can learn about the core functionalities of the protocol.&#x20;

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

### Open-source

All contracts used in the PWN protocol are under _GNU General Public_ License _v3.0_. Please refer to [LICENSE.md](https://github.com/PWNFinance/pwn\_contracts/blob/master/LICENSE.md) on our GitHub for more information about the license.&#x20;
