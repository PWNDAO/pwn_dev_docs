# Access Control

## 1. Summary

Contracts in the PWN Protocol can implement PWNHubAccessControl.sol for managing their access control through the PWN Hub. This is not a standalone contract.&#x20;

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/hub/PWNHubAccessControl.sol" %}
GitHub
{% endembed %}

## 3. Contract details

* _PWNHubAccessControl.sol_ is written in Solidity version 0.8.16

### Features

* Implements modifiers for managing access control through the PWN Hub

### Modifiers

<details>

<summary><code>onlyActiveLoan</code></summary>

#### Overview

This modifier reverts if `msg.sender` doesn't have the `ACTIVE_LOAN` tag set in the PWN Hub.&#x20;

#### Implementation

```solidity
modifier onlyActiveLoan() {
    if (hub.hasTag(msg.sender, PWNHubTags.ACTIVE_LOAN) == false)
        revert CallerMissingHubTag(PWNHubTags.ACTIVE_LOAN);
    _;
}
```

</details>

<details>

<summary><code>onlyWithTag</code></summary>

#### Overview

This modifier reverts if `msg.sender` doesn't have the supplied tag set in the PWN Hub.&#x20;

This modifier defines one parameter:

* `bytes32`**`tag`**

#### Implementation

```solidity
modifier onlyWithTag(bytes32 tag) {
    if (hub.hasTag(msg.sender, tag) == false)
        revert CallerMissingHubTag(tag);
    _;
}
```

</details>
