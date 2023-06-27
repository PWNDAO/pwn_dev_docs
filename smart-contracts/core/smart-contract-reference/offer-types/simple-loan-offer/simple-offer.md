# Simple Offer

## 1. Summary

PWNSimpleLoanSimpleOffer.sol defines Simple Offer for the Simple Loan type.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/simple/factory/offer/PWNSimpleLoanSimpleOffer.sol" %}
GitHub
{% endembed %}

{% file src="../../../../../.gitbook/assets/PWNSimpleLoanSimpleOffer.json" %}
JSON ABI
{% endfile %}

## 3. Contract details

* _PWNSimpleLoanSimpleOffer.sol_ is written in Solidity version 0.8.16

### Features

* Make Simple Offer
* Create Simple Loan terms from Simple Offer
* Provides a function to encode Simple Offer

### Inherited contracts, implemented Interfaces and ERCs

* [PWNSimpleLoanOffer](./)

### Functions

<details>

<summary><code>makeOffer</code></summary>

#### Overview

Make an on-chain simple offer.

This function takes one argument supplied by the caller:

* `Offer calldata`**`offer`** - Offer struct with the simple offer parameters

#### Implementation

```solidity
function makeOffer(Offer calldata offer) external {
    _makeOffer(getOfferHash(offer), offer.lender);
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

    Offer memory offer = abi.decode(factoryData, (Offer));
    bytes32 offerHash = getOfferHash(offer);

    address lender = offer.lender;
    address borrower = caller;

    // Check that offer has been made via on-chain tx, EIP-1271 or signed off-chain
    if (offersMade[offerHash] == false)
        if (PWNSignatureChecker.isValidSignatureNow(lender, offerHash, signature) == false)
            revert InvalidSignature();

    // Check valid offer
    if (offer.expiration != 0 && block.timestamp >= offer.expiration)
        revert OfferExpired();

    if (revokedOfferNonce.isNonceRevoked(lender, offer.nonce) == true)
        revert NonceAlreadyRevoked();

    if (offer.borrower != address(0))
        if (borrower != offer.borrower)
            revert CallerIsNotStatedBorrower(offer.borrower);

    if (offer.duration < MIN_LOAN_DURATION)
        revert InvalidDuration();

    // Prepare collateral and loan asset
    MultiToken.Asset memory collateral = MultiToken.Asset({
        category: offer.collateralCategory,
        assetAddress: offer.collateralAddress,
        id: offer.collateralId,
        amount: offer.collateralAmount
    });
    MultiToken.Asset memory loanAsset = MultiToken.Asset({
        category: MultiToken.Category.ERC20,
        assetAddress: offer.loanAssetAddress,
        id: 0,
        amount: offer.loanAmount
    });

    // Create loan object
    loanTerms = PWNLOANTerms.Simple({
        lender: lender,
        borrower: borrower,
        expiration: uint40(block.timestamp) + offer.duration,
        collateral: collateral,
        asset: loanAsset,
        loanRepayAmount: offer.loanAmount + offer.loanYield
    });

    // Revoke offer if not persistent
    if (!offer.isPersistent)
        revokedOfferNonce.revokeNonce(lender, offer.nonce);
}
```

</details>

<details>

<summary><code>encodeLoanTermsFactoryData</code></summary>

#### Overview

Returns encoded input data for the loan terms factory (inherited by this contract).

This function takes one argument supplied by the caller:

* `Offer memory`**`offer`** - Simple offer struct to encode

#### Implementation

```solidity
function encodeLoanTermsFactoryData(Offer memory offer) external pure returns (bytes memory) {
    return abi.encode(offer);
}
```

</details>

### View Functions

<details>

<summary><code>getRequestHash</code></summary>

#### Overview

Returns a simple offer hash according to [EIP-712](https://eips.ethereum.org/EIPS/eip-712).

This function takes one argument supplied by the caller:

* `Offer memory`**`offer`** - Simple offer struct to get hash of

#### Implementation

```solidity
function getOfferHash(Offer memory offer) public view returns (bytes32) {
    return keccak256(abi.encodePacked(
        hex"1901",
        DOMAIN_SEPARATOR,
        keccak256(abi.encodePacked(
            OFFER_TYPEHASH,
            abi.encode(offer)
        ))
    ));
}
```

</details>

### Events

The PWN Simple Loan Simple Offer contract inherits events from the [Simple Loan Offer](./) contract and does not define any additional custom events or errors.

### Simple Offer Struct

<table><thead><tr><th width="157.09421454876235">Type</th><th width="146.45656287647148">Name</th><th>Comment</th></tr></thead><tbody><tr><td><code>MultiToken.Category</code></td><td><code>collateralCategory</code></td><td>0 -> ERC-20<br>1 -> ERC-721<br>2 -> ERC-1155</td></tr><tr><td><code>address</code></td><td><code>collateralAddress</code></td><td>Address of an asset used as a collateral.</td></tr><tr><td><code>uint256</code></td><td><code>collateralId</code></td><td>Token id of an asset used as a collateral, in case of ERC20 should be 0.</td></tr><tr><td><code>uint256</code></td><td><code>collateralAmount</code></td><td>Amount of tokens used as a collateral, in case of ERC721 should be 0.</td></tr><tr><td><code>address</code></td><td><code>loanAssetAddress</code></td><td>Address of an loaned asset.</td></tr><tr><td><code>uint256</code></td><td><code>loanAmount</code></td><td>Amount of tokens which is requested as a loan.</td></tr><tr><td><code>uint256</code></td><td><code>loanYield</code></td><td>Amount of tokens which act as a lenders loan interest. Borrower has to pay back a borrowed amount + yield.</td></tr><tr><td><code>uint32</code></td><td><code>duration</code></td><td>Loan duration in seconds.</td></tr><tr><td><code>uint40</code></td><td><code>expiration</code></td><td>Request expiration unix timestamp in seconds.</td></tr><tr><td><code>address</code></td><td><code>borrower</code></td><td>Address of a borrower. This address has to sign a request for it to be valid. If the address is zero address, anybody with valid collateral can accept the offer.</td></tr><tr><td><code>address</code></td><td><code>lender</code></td><td>Address of a lender. Only this address can accept a request. If the address is zero address, anybody with a loan asset can accept the request.</td></tr><tr><td><code>bool</code></td><td><code>isPersistent</code></td><td>If true, offer will not be revoked on acceptance. Persistent offer can be revoked manually.</td></tr><tr><td><code>uint256</code></td><td><code>nonce</code></td><td>Additional value to enable identical requests in time. Without it, it would be impossible to make a request, which was once revoked. Nonce can be used to create a group of requests, where accepting one request will make other requests in the group invalid.</td></tr></tbody></table>
