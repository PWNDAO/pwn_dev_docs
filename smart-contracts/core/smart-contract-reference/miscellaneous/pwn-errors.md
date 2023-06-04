# PWN Errors

## 1. Summary

PWNErrors.sol defines errors that are specific to the PWN Protocol.&#x20;

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/PWNErrors.sol" %}
GitHub
{% endembed %}

## 3. Implementation

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.16;


// Access control
error CallerMissingHubTag(bytes32);

// Loan contract
error LoanDefaulted(uint40);
error InvalidLoanStatus(uint256);
error NonExistingLoan();
error CallerNotLOANTokenHolder();
error InvalidExtendedExpirationDate();

// Invalid asset
error InvalidLoanAsset();
error InvalidCollateralAsset();

// LOAN token
error InvalidLoanContractCaller();

// Vault
error UnsupportedTransferFunction();
error IncompleteTransfer();

// Nonce
error NonceAlreadyRevoked();
error InvalidMinNonce();

// Signature checks
error InvalidSignatureLength(uint256);
error InvalidSignature();

// Offer
error CallerIsNotStatedBorrower(address);
error OfferExpired();
error CollateralIdIsNotWhitelisted();

// Request
error CallerIsNotStatedLender(address);
error RequestExpired();

// Request & Offer
error InvalidDuration();

// Input data
error InvalidInputData();

// Config
error InvalidFeeValue();
error InvalidFeeCollector();
```
