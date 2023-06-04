# Simple Loan Terms Factory

## 1. Summary

PWNSimpleLoanTermsFactory.sol is an abstract contract implemented by offer and loan request types. This contract is used by the Simple Loan type to construct loan terms when creating a loan.&#x20;

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/simple/factory/PWNSimpleLoanTermsFactory.sol" %}
GitHub
{% endembed %}

## 3. Contract details

* _PWNSimpleLoanTermsFactory.sol_ is written in Solidity version 0.8.16

The contract defines a public constant that specifies the minimal duration of a loan for the Simple Loan type.&#x20;

```solidity
uint32 public constant MIN_LOAN_DURATION = 600; // 10 min
```

And also specifies a virtual function `createLOANTerms`. Each offer and loan request type implements this function. `createLOANTerms` returns Simple Loan terms based on the provided data.

```solidity
function createLOANTerms(
    address caller,
    bytes calldata factoryData,
    bytes calldata signature
) external virtual returns (PWNLOANTerms.Simple memory loanTerms);
```

The parameters of the function are:

* `address`**`caller`** - The caller of the `createLoan` function on the Loan contract
* `bytes calldata`**`factoryData`** - Encoded data for the Loan Terms factory
* `bytes calldata`**`signature`** - Singed loan factory data
