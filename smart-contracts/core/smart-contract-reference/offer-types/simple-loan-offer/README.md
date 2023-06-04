# Simple Loan Offer

## 1. Summary

PWNSimpleLoanOffer.sol is an abstract contract inherited by Simple Offer types.&#x20;

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/simple/factory/offer/base/PWNSimpleLoanOffer.sol" %}
GitHub
{% endembed %}

## 3. Contract details

* _PWNSimpleLoanOffer.sol_ is written in Solidity version 0.8.16

### Features

* Implements `_makeOffer` for lenders to be able to make on-chain offer
* Provides a way to revoke offers

### Inherited contracts, implemented Interfaces and ERCs

* [PWNSimpleLoanTermsFactory](../../loan-types/simple-loan/simple-loan-terms-factory.md)
* [PWNHubAccessControl](../../pwn-hub/access-control.md)

### Functions

<details>

<summary><code>_makeOffer</code></summary>

#### Overview

A function to make an on-chain offer.

This function takes two arguments supplied by the offer type:

* `bytes32`**`offerStructHash`** - Hash of a proposed offer
* `address`**`lender`** - Address of an offer proposer (lender)

#### Implementation

```solidity
function _makeOffer(bytes32 offerStructHash, address lender) internal {
    // Check that caller is a lender
    if (msg.sender != lender)
        revert CallerIsNotStatedLender(lender);

    // Mark offer as made
    offersMade[offerStructHash] = true;

    emit OfferMade(offerStructHash, lender);
}
```

</details>

<details>

<summary><code>revokeOfferNonce</code></summary>

#### Overview

Revokes supplied offer nonce for `msg.sender`.

This function takes one argument supplied by the caller:

* `uint256`**`offerNonce`** - Offer nonce to revoke

#### Implementation

```solidity
function revokeOfferNonce(uint256 offerNonce) external {
    revokedOfferNonce.revokeNonce(msg.sender, offerNonce);
}
```

</details>

### Events

The PWN Simple Loan Offer contract defines one event and no custom errors.

```solidity
event OfferMade(bytes32 indexed offerHash, address indexed lender);
```

<details>

<summary><code>OfferMade</code></summary>

OfferMade event is emitted when a lender creates an on-chain offer.

This event has two parameters:

* `bytes32 indexed`**`offerStructHash`** - Hash of a proposed offer
* `address indexed`**`lender`** - Address of an offer proposer (lender)

</details>
