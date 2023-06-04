# PWN Vault

## 1. Summary

Loan contracts in the PWN Protocol inherit the PWNVault.sol contract for transferring and managing assets. This is not a standalone contract.&#x20;

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/PWNVault.sol" %}
GitHub
{% endembed %}

## 3. Contract details

* _PWNVault.sol_ is written in Solidity version 0.8.16

### Features

* Transferring assets
* Set Vault allowance for an asset

### Inherited contracts, implemented Interfaces and ERCs

* [IERC721Receiver](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/IERC721Receiver.sol)
* [IERC1155Receiver](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/IERC1155Receiver.sol)
* [ERC-165](https://eips.ethereum.org/EIPS/eip-165#how-interfaces-are-identified%5BEIP%20section%5D)

### Functions

<details>

<summary><code>_pull</code></summary>

#### Overview

Takes a supplied asset and pulls it into the vault from the origin.

**This function assumes a prior token approval was made to the vault address.**

This function takes two arguments supplied by the caller:

* `MultiToken.Asset`**`asset`** - The transferred asset (see [MultiToken](../../libraries/multitoken.md))
* `address indexed`**`origin`**

#### Implementation

```solidity
function _pull(MultiToken.Asset memory asset, address origin) internal {
    uint256 originalBalance = asset.balanceOf(address(this));

    asset.transferAssetFrom(origin, address(this));
    _checkTransfer(asset, originalBalance, address(this));

    emit VaultPull(asset, origin);
}
```

</details>

<details>

<summary><code>_push</code></summary>

#### Overview

Pushes a supplied asset from the vault to the beneficiary.

This function takes two arguments supplied by the caller:

* `MultiToken.Asset`**`asset`** - The transferred asset (see [MultiToken](../../libraries/multitoken.md))
* `address indexed`**`beneficiary`**

#### Implementation

```solidity
function _push(MultiToken.Asset memory asset, address beneficiary) internal {
    uint256 originalBalance = asset.balanceOf(beneficiary);

    asset.safeTransferAssetFrom(address(this), beneficiary);
    _checkTransfer(asset, originalBalance, beneficiary);

    emit VaultPush(asset, beneficiary);
}
```

</details>

<details>

<summary><code>_pushFrom</code></summary>

#### Overview

Pushes a supplied asset from the origin to the beneficiary.

**This function assumes a prior token approval was made to the vault address.**

This function takes three arguments supplied by the caller:

* `MultiToken.Asset`**`asset`** - The transferred asset (see [MultiToken](../../libraries/multitoken.md))
* `address indexed`**`origin`**
* `address indexed`**`beneficiary`**

#### Implementation

```solidity
function _pushFrom(MultiToken.Asset memory asset, address origin, address beneficiary) internal {
    uint256 originalBalance = asset.balanceOf(beneficiary);

    asset.safeTransferAssetFrom(origin, beneficiary);
    _checkTransfer(asset, originalBalance, beneficiary);

    emit VaultPushFrom(asset, origin, beneficiary);
}
```

</details>

<details>

<summary><code>_permit</code></summary>

#### Overview

Uses signed permit data to set the vault allowance for an asset.

This function takes three arguments supplied by the caller:

* `MultiToken.Asset`**`asset`** - The transferred asset (see [MultiToken](../../libraries/multitoken.md))
* `address indexed`**`origin`**
* `bytes memory`**`beneficiary`**

#### Implementation

```solidity
function _permit(MultiToken.Asset memory asset, address origin, bytes memory permit) internal {
    if (permit.length > 0)
        asset.permit(origin, address(this), permit);
}
```

</details>

### Events

The PWN Vault contract defines one event and no custom errors.

```solidity
event VaultPull(MultiToken.Asset asset, address indexed origin);
event VaultPush(MultiToken.Asset asset, address indexed beneficiary);
event VaultPushFrom(MultiToken.Asset asset, address indexed origin, address indexed beneficiary);
```

<details>

<summary><code>VaultPull</code></summary>

VaultPull event is emitted when a transfer happens from the `origin` to the vault.&#x20;

This event has two parameters:

* `MultiToken.Asset`**`asset`** - The transferred asset (see [MultiToken](../../libraries/multitoken.md))
* `address indexed`**`origin`**

</details>

<details>

<summary><code>VaultPush</code></summary>

VaultPush event is emitted when a transfer happens from the vault to the `beneficiary`.

This event has two parameters:

* `MultiToken.Asset`**`asset`** - The transferred asset (see [MultiToken](../../libraries/multitoken.md))
* `address indexed`**`beneficiary`**

</details>

<details>

<summary><code>VaultPushFrom</code></summary>

VaultPushFrom event is emitted when a transfer happens from the `origin` to the `beneficiary`.&#x20;

This event has three parameters:

* `MultiToken.Asset`**`asset`** - The transferred asset (see [MultiToken](../../libraries/multitoken.md))
* `address indexed`**`origin`**
* `address indexed`**`beneficiary`**

</details>
