# Offer types

## 1. Summary

A lender can choose between two types while making an offer. **Basic and flexible**.

## 2. Offer Types Detailed

### Basic

The Basic offer is where the lender sets all loan parameters up-front and the borrower has an option to accept or not. Nothing else.

**`Offer`** struct has these properties:

| Type                                                                    | Name                 | Comment                                                                                                                                |
| ----------------------------------------------------------------------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `address`                                                               | `collateralAddress`  | Address of an asset used as a collateral                                                                                               |
| `MultiToken.Category` (see [MultiToken](../../libraries/multitoken.md)) | `collateralCategory` | Category of an asset used as a collateral (0 == ERC20, 1 == ERC721, 2 == ERC1155)                                                      |
| `uint256`                                                               | `collateralAmount`   | The Amount of tokens used as collateral, in the case of ERC721 should be 1                                                             |
| `uint256`                                                               | `collateralId`       | Token id of an asset used as collateral, in the case of ERC20 should be 0                                                              |
| `address`                                                               | `loanAssetAddress`   | Address of an asset which is lent to a borrower                                                                                        |
| `uint256`                                                               | `loanAmount`         | Amount of tokens which is offered as a loan to a borrower                                                                              |
| `uint256`                                                               | `loanYield`          | Amount of tokens that acts as a lender's loan interest. Borrower has to pay back borrowed amount + yield                               |
| `uint32`                                                                | `duration`           | Loan duration in seconds                                                                                                               |
| `uint40`                                                                | `expiration`         | Offer expiration timestamp in seconds                                                                                                  |
| `address`                                                               | `lender`             | Address of a lender. This address has to sign an offer to be valid                                                                     |
| `bytes32`                                                               | `nonce`              | Additional value to enable identical offers in time. Without it, it would be impossible to make again an offer, which was once revoked |

{% hint style="info" %}
With the flexible offer struct, we change the `collateralId`, `loanAmount`, `loanYield` and `duration` parameters to a range of these parameters.&#x20;
{% endhint %}

### Flexible

With flexible offers, lenders can give borrowers additional flexibility by not providing concrete values but rather giving borrower ranges for several parameters. When accepting an offer, a borrower has to provide concrete values to proceed. This increases a lender's chance of accepting their offer as it could be accepted by more borrowers.

**`FlexibleOffer`** struct has these properties:

| Type                                                                    | Name                                   | Comment                                                                                                                                |
| ----------------------------------------------------------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `address`                                                               | `collateralAddress`                    | Address of an asset used as a collateral                                                                                               |
| `MultiToken.Category` (see [MultiToken](../../libraries/multitoken.md)) | `collateralCategory`                   | Category of an asset used as a collateral (0 == ERC20, 1 == ERC721, 2 == ERC1155)                                                      |
| `uint256`                                                               | `collateralAmount`                     | The Amount of tokens used as collateral, in the case of ERC721 should be 0                                                             |
| `bytes32`                                                               | **`collateralIdsWhitelistMerkleRoot`** | Root of a merkle tree constructed on an array of whitelisted collateral ids                                                            |
| `address`                                                               | `loanAssetAddress`                     | Address of an asset which is lent to a borrower                                                                                        |
| `uint256`                                                               | **`loanAmountMax`**                    | Max amount of tokens which is offered as a loan to borrower                                                                            |
| `uint256`                                                               | **`loanAmountMin`**                    | Min amount of tokens which is offered as a loan to borrower                                                                            |
| `uint256`                                                               | **`loanYieldMax`**                     | Amount of tokens which acts as a lender's loan interest for max duration.                                                              |
| `uint32`                                                                | **`durationMax`**                      | Maximal loan duration in seconds                                                                                                       |
| `uint32`                                                                | **`durationMin`**                      | Minimal loan duration in seconds                                                                                                       |
| `uint40`                                                                | `expiration`                           | Offer expiration timestamp in seconds                                                                                                  |
| `address`                                                               | `lender`                               | Address of a lender. This address has to sign an offer to be valid.                                                                    |
| `bytes32`                                                               | `nonce`                                | Additional value to enable identical offers in time. Without it, it would be impossible to make again an offer, which was once revoked |

{% hint style="success" %}
Flexible offers enable lenders to create an offer for a whole NFT collection.
{% endhint %}

{% hint style="info" %}
Don't know what a merkle tree is? Read this [article](https://brilliant.org/wiki/merkle-tree/) for more information.
{% endhint %}

#### Flexible offer values

When a borrower decides to accept a flexible offer they have to provide a specific representation of the offer. They do this by providing `FlexibleOfferValues` struct as the second argument for the [`createLoanFlexible`](./#createloanflexible) function.

**`FlexibleOfferValues`** struct has these properties:

| Type        | Name                   | Comment                                                                                          |
| ----------- | ---------------------- | ------------------------------------------------------------------------------------------------ |
| `uint256`   | `collateralId`         | Collateral token id. Ignored if it’s not a collection offer                                      |
| `uint256`   | `loanAmount`           | Loan asset amount (always ERC20) in range \<loanAmountMin; loanAmountMax>                        |
| `uint32`    | `duration`             | Offered loan duration in range \<durationMin; durationMax>                                       |
| `bytes32[]` | `merkleInclusionProof` | Proof that selected collateralId is indeed in the Merkle tree with root in signed flexible offer |

#### How is the loan repay amount calculated?

$$
loanRepayAmount = loanAmount + \frac{loanYieldMax*duration}{durationMax}
$$

* `loanAmount` - Loan asset amount (always ERC20) in range \<loanAmountMin; loanAmountMax>
* `loanYieldMax` - Amount of tokens which acts as a lender's loan interest for max duration
* `duration` - Offered loan duration in range \<durationMin; durationMax>
* `durationMax` - Maximum loan duration in seconds
