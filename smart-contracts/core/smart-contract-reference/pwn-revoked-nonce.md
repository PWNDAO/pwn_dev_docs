# PWN Revoked Nonce

## 1. Summary

PWNRevokedNonce.sol contract is used for revoking offers and loan requests. Each offer or loan request has a unique nonce value which can be revoked. It is also possible to set a minimal nonce value to revoke all nonces up to a set value. One PWNRevokedNonce contract corresponds with one offer or loan type.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/nonce/PWNRevokedNonce.sol" %}
GitHub
{% endembed %}

{% file src="../../../.gitbook/assets/PWNRevokedNonce.json" %}
JSON ABI
{% endfile %}



## 3. Contract details

* _PWNRevokedNonce.sol_ is written in Solidity version 0.8.16

### Features

* Revoke nonce
* Revoke nonce on behalf of the owner
* Set a minimal nonce value

### Functions

<details>

<summary><code>revokeNonce(uint256 nonce)</code></summary>

#### Overview

Revokes supplied nonce for `msg.sender`.

This function takes one argument supplied by the caller:

* `uint256`**`nonce`**

#### Implementation

```solidity
function revokeNonce(uint256 nonce) external {
    _revokeNonce(msg.sender, nonce);
}
```

</details>

<details>

<summary><code>revokeNonce(address owner, uint256 nonce)</code></summary>

#### Overview

Revokes supplied nonce on behalf of the owner. This function can be called only by addresses with the `accessTag` set in the [PWN Hub](pwn-hub/).

This function takes two arguments supplied by the caller:

* `address`**`owner`**
* `uint256`**`nonce`**

#### Implementation

```solidity
function revokeNonce(address owner, uint256 nonce) external onlyWithTag(accessTag) {
    _revokeNonce(owner, nonce);
}
```

</details>

<details>

<summary><code>setMinNonce</code></summary>

#### Overview

Sets a minimal nonce for `msg.sender`, thus revoking all nonces with a smaller nonce.

This function takes one argument supplied by the caller:

* `uint256`**`minNonce`**

#### Implementation

```solidity
function setMinNonce(uint256 minNonce) external {
    // Check that nonce is greater than current min nonce
    uint256 currentMinNonce = minNonces[msg.sender];
    if (currentMinNonce >= minNonce)
        revert InvalidMinNonce();

    // Set new min nonce value
    minNonces[msg.sender] = minNonce;

    // Emit event
    emit MinNonceSet(msg.sender, minNonce);
}
```

</details>

### View Functions

<details>

<summary><code>isNonceRevoked</code></summary>

#### Overview

This function returns a boolean determining if the supplied nonce is valid for a given address.

This function takes two arguments supplied by the caller:

* `address`**`owner`** - The address to check
* `uint256`**`nonce`** - The nonce to check

#### Implementation

```solidity
function isNonceRevoked(address owner, uint256 nonce) external view returns (bool) {
    if (nonce < minNonces[owner])
        return true;

    return revokedNonces[owner][nonce];
}
```

</details>

### Events

The PWN Revoked Nonce contract defines two events and no custom errors.

```solidity
event NonceRevoked(address indexed owner, uint256 indexed nonce);
event MinNonceSet(address indexed owner, uint256 indexed minNonce);
```

<details>

<summary><code>NonceRevoked</code></summary>

A NonceRevoked event is emitted when a nonce is revoked.&#x20;

This event has two parameters:

* `address indexed`**`owner`**
* `uint256 indexed`**`nonce`**

</details>

<details>

<summary><code>MinNonceSet</code></summary>

A MinNonceSet event is emitted when a minimal nonce value is set.&#x20;

This event has two parameters:

* `address indexed`**`owner`**
* `uint256 indexed`**`minNonce`**

</details>
