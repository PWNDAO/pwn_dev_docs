# Simple Loan Request

## 1. Summary

PWNSimpleLoanRequest.sol is an abstract contract inherited by Simple Loan Request types.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/simple/factory/request/base/PWNSimpleLoanRequest.sol" %}
GitHub
{% endembed %}

## 3. Contract details

* _PWNSimpleLoanRequest.sol_ is written in Solidity version 0.8.16

### Features

* Implements `_makeRequest` for borrowers to be able to make on-chain loan request&#x20;
* Provides a way to revoke loan requests

### Inherited contracts, implemented Interfaces and ERCs

* [PWNSimpleLoanTermsFactory](../../loan-types/simple-loan/simple-loan-terms-factory.md)
* [PWNHubAccessControl](../../pwn-hub/access-control.md)

### Functions

<details>

<summary><code>_makeRequest</code></summary>

#### Overview

A function to make an on-chain loan request.

This function takes two arguments supplied by the loan request type:

* `bytes32`**`requestStructHash`** - Hash of a proposed loan request
* `address`**`borrower`** - Address of a loan request proposer (borrower)

#### Implementation

```solidity
function _makeRequest(bytes32 requestStructHash, address borrower) internal {
    // Check that caller is a borrower
    if (msg.sender != borrower)
        revert CallerIsNotStatedBorrower(borrower);

    // Mark request as made
    requestsMade[requestStructHash] = true;

    emit RequestMade(requestStructHash, borrower);
}
```

</details>

<details>

<summary><code>revokeRequestNonce</code></summary>

#### Overview

Revokes supplied loan request nonce for `msg.sender`.

This function takes one argument supplied by the caller:

* `uint256`**`offerNonce`** - Loan request nonce to revoke

#### Implementation

```solidity
function revokeRequestNonce(uint256 requestNonce) external {
    revokedRequestNonce.revokeNonce(msg.sender, requestNonce);
}
```

</details>

### Events

The PWN Simple Loan Request contract defines one event and no custom errors.

```solidity
event RequestMade(bytes32 indexed requestHash, address indexed borrower);
```

<details>

<summary><code>RequestMade</code></summary>

RequestMade event is emitted when a borrower creates an on-chain loan request.

This event has two parameters:

* `bytes32 indexed`**`requestHash`** - Hash of a proposed loan request
* `address indexed`**`borrower`** - Address of a loan request proposer (borrower)

</details>
