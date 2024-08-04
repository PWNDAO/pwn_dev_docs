# Permit

## 1. Summary

Permit.sol defines a Permit struct for off-chain signed ERC-20 permits.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/vault/Permit.sol" %}

## 3. Contract details

* _Permit.sol_ is written in Solidity version 0.8.16

### Errors

```solidity
error InvalidPermitOwner(address current, address expected);
error InvalidPermitAsset(address current, address expected);
```

<details>

<summary><code>InvalidPermitOwner</code></summary>

InvalidPermitOwner error is thrown when the permit owner doesn't match.

This error has two parameters:

* `address`**`current`**
* `address`**`expected`**

</details>

<details>

<summary><code>InvalidPermitAsset</code></summary>

InvalidPermitAsset error is thrown when the permit asset doesn't match.

This error has two parameters:

* `address`**`current`**
* `address`**`expected`**

</details>

### Permit Struct

<table><thead><tr><th width="157.09421454876235">Type</th><th width="216.45656287647148">Name</th><th>Comment</th></tr></thead><tbody><tr><td><code>address</code></td><td><code>asset</code></td><td>Address of the asset the permit is for</td></tr><tr><td><code>address</code></td><td><code>owner</code></td><td>Owner of the asset</td></tr><tr><td><code>uint256</code></td><td><code>amount</code></td><td>Amount of the asset</td></tr><tr><td><code>uint256</code></td><td><code>deadline</code></td><td>Deadline for the permit</td></tr><tr><td><code>uint8</code></td><td><code>v</code></td><td>v value of the signature</td></tr><tr><td><code>bytes32</code></td><td><code>r</code></td><td>r value of the signature</td></tr><tr><td><code>bytes32</code></td><td><code>s</code></td><td>s value of the signature</td></tr></tbody></table>
