# Simple Request

## 1. Summary

PWNSimpleLoanSimpleRequest.sol defines Simple Request for the Simple Loan type.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/simple/factory/request/PWNSimpleLoanSimpleRequest.sol" %}
GitHub
{% endembed %}

{% file src="../../../../../.gitbook/assets/PWNSimpleLoanSimpleRequest.json" %}
JSON ABI
{% endfile %}

## 3. Contract details

* _PWNSimpleLoanSimpleRequest.sol_ is written in Solidity version 0.8.16

### Features

* Make Simple Request
* Create Simple Loan terms from Simple Request
* Provides a function to encode Simple Request

### Inherited contracts, implemented Interfaces and ERCs

* [PWNSimpleLoanRequest](./)

### Functions

<details>

<summary><code>makeRequest</code></summary>

#### Overview

Make an on-chain simple loan request.

This function takes one argument supplied by the caller:

* `Request calldata`**`request`** - Request struct with the simple request parameters

#### Implementation

```solidity
function makeRequest(Request calldata request) external {
    _makeRequest(getRequestHash(request), request.borrower);
}
```

</details>

<details>

<summary><code>createLOANTerms</code></summary>

#### Overview

Builds simple loan terms from given data.

This function takes three arguments supplied by the loan type:

* `address`**`caller`** - The caller of the `createLoan` function on the Loan contract
* `bytes calldata`**`factoryData`** - Encoded data for the Loan Terms factory
* `bytes calldata`**`signature`** - Singed loan factory data

#### Implementation

```solidity
function createLOANTerms(
    address caller,
    bytes calldata factoryData,
    bytes calldata signature
) external override onlyActiveLoan returns (PWNLOANTerms.Simple memory loanTerms) {

    Request memory request = abi.decode(factoryData, (Request));
    bytes32 requestHash = getRequestHash(request);

    address lender = caller;
    address borrower = request.borrower;

    // Check that request has been made via on-chain tx, EIP-1271 or signed off-chain
    if (requestsMade[requestHash] == false)
        if (PWNSignatureChecker.isValidSignatureNow(borrower, requestHash, signature) == false)
            revert InvalidSignature();

    // Check valid request
    if (request.expiration != 0 && block.timestamp >= request.expiration)
        revert RequestExpired();

    if (revokedRequestNonce.isNonceRevoked(borrower, request.nonce) == true)
        revert NonceAlreadyRevoked();

    if (request.lender != address(0))
        if (lender != request.lender)
            revert CallerIsNotStatedLender(request.lender);

    if (request.duration < MIN_LOAN_DURATION)
        revert InvalidDuration();

    // Prepare collateral and loan asset
    MultiToken.Asset memory collateral = MultiToken.Asset({
        category: request.collateralCategory,
        assetAddress: request.collateralAddress,
        id: request.collateralId,
        amount: request.collateralAmount
    });
    MultiToken.Asset memory loanAsset = MultiToken.Asset({
        category: MultiToken.Category.ERC20,
        assetAddress: request.loanAssetAddress,
        id: 0,
        amount: request.loanAmount
    });

    // Create loan object
    loanTerms = PWNLOANTerms.Simple({
        lender: lender,
        borrower: borrower,
        expiration: uint40(block.timestamp) + request.duration,
        collateral: collateral,
        asset: loanAsset,
        loanRepayAmount: request.loanAmount + request.loanYield
    });

    revokedRequestNonce.revokeNonce(borrower, request.nonce);
}
```

</details>

<details>

<summary><code>encodeLoanTermsFactoryData</code></summary>

#### Overview

Returns encoded input data for the loan terms factory (inherited by this contract).

This function takes one argument supplied by the caller:

* `Request memory`**`request`** - Simple request struct to encode

#### Implementation

```solidity
function encodeLoanTermsFactoryData(Request memory request) external pure returns (bytes memory) {
    return abi.encode(request);
}
```

</details>

### View Functions

<details>

<summary><code>getRequestHash</code></summary>

#### Overview

Returns a simple request hash according to [EIP-712](https://eips.ethereum.org/EIPS/eip-712).

This function takes one argument supplied by the caller:

* `Request memory`**`request`** - Simple request struct to get hash of

#### Implementation

```solidity
function getRequestHash(Request memory request) public view returns (bytes32) {
    return keccak256(abi.encodePacked(
        hex"1901",
        DOMAIN_SEPARATOR,
        keccak256(abi.encodePacked(
            REQUEST_TYPEHASH,
            abi.encode(request)
        ))
    ));
}
```

</details>

### Events

The PWN Simple Loan Simple Request contract inherits events from the [Simple Loan Request](./) contract and does not define any additional custom events or errors.

### Simple Request Struct

<table><thead><tr><th width="157.09421454876235">Type</th><th width="216.45656287647148">Name</th><th>Comment</th></tr></thead><tbody><tr><td><code>MultiToken.Category</code></td><td><code>collateralCategory</code></td><td>0 -> ERC-20<br>1 -> ERC-721<br>2 -> ERC-1155</td></tr><tr><td><code>address</code></td><td><code>collateralAddress</code></td><td>Address of an asset used as a collateral.</td></tr><tr><td><code>uint256</code></td><td><code>collateralId</code></td><td>Token id of an asset used as a collateral, in case of ERC20 should be 0.</td></tr><tr><td><code>uint256</code></td><td><code>collateralAmount</code></td><td>Amount of tokens used as a collateral, in case of ERC721 should be 0.</td></tr><tr><td><code>address</code></td><td><code>loanAssetAddress</code></td><td>Address of an loaned asset.</td></tr><tr><td><code>uint256</code></td><td><code>loanAmount</code></td><td>Amount of tokens which is requested as a loan.</td></tr><tr><td><code>uint256</code></td><td><code>loanYield</code></td><td>Amount of tokens which act as a lenders loan interest. Borrower has to pay back a borrowed amount + yield.</td></tr><tr><td><code>uint32</code></td><td><code>duration</code></td><td>Loan duration in seconds.</td></tr><tr><td><code>uint40</code></td><td><code>expiration</code></td><td>Request expiration unix timestamp in seconds.</td></tr><tr><td><code>address</code></td><td><code>borrower</code></td><td>Address of a borrower. This address has to sign a request for it to be valid.</td></tr><tr><td><code>address</code></td><td><code>lender</code></td><td>Address of a lender. Only this address can accept a request. If the address is zero address, anybody with a loan asset can accept the request.</td></tr><tr><td><code>uint256</code></td><td><code>nonce</code></td><td>Additional value to enable identical requests in time. Without it, it would be impossible to make a request, which was once revoked. Nonce can be used to create a group of requests, where accepting one request will make other requests in the group invalid.</td></tr></tbody></table>
