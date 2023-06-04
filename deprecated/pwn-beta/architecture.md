# Architecture

Before we dive deep into the contracts let's have a look at the high-level architecture.

<figure><img src="../../.gitbook/assets/Architecture.png" alt=""><figcaption><p>PWN smart contract architecture</p></figcaption></figure>

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
