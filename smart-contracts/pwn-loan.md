# PWN LOAN

## 1. Summary

PWN LOAN is a PWN contextual extension of a standard [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) token. **Each LOAN is defined as an ERC1155 NFT**. The PWN LOAN contract allows for reading the contextual information of the loans (like status, expirations, etc.) but **all of its contract features can only be called through the** [**PWN**](pwn/) **contract.**

## 2. Important links

* **Deployment addresses**
  * Mainnet: [0x_cFe385287200F0c10a54100e9b22855A73664156_](https://etherscan.io/address/0xcFe385287200F0c10a54100e9b22855A73664156)__
  * Polygon: [0x_8680AEE63E48AACB51Ddc5Ad15979FC169C1cf2B_](https://polygonscan.com/address/0x8680AEE63E48AACB51Ddc5Ad15979FC169C1cf2B)__
  * Görli: [0x_c9E94453d182c50984A2a4afdD60796D25B027Aa_](https://goerli.etherscan.io/address/0xc9E94453d182c50984A2a4afdD60796D25B027Aa)__
  * Mumbai: [0x_7C995e64a24aCb5806521276697B244D1f65f708_](https://mumbai.polygonscan.com/address/0x7C995e64a24aCb5806521276697B244D1f65f708)__
  * Rinkeby (deprecated): [0x_C33B746Ac85703178D5a796f960b5e855172e7F7_](https://rinkeby.etherscan.io/address/0xC33B746Ac85703178D5a796f960b5e855172e7F7)__
* **Source code**
  * [GitHub](https://github.com/PWNFinance/pwn\_contracts/blob/master/contracts/PWNLOAN.sol)
* **ABI**
  * [JSON](https://api.etherscan.io/api?module=contract\&action=getabi\&address=0xcFe385287200F0c10a54100e9b22855A73664156)
  * [Text](http://api.etherscan.io/api?module=contract\&action=getabi\&address=0xcFe385287200F0c10a54100e9b22855A73664156\&format=raw)

## 3. Contract Details

* _PWNLOAN.sol_ contract is written in Solidity version 0.8.4

### LOAN token lifecycle

The PWN LOAN token is a tokenized representation of a loan that can aquire different states:

* **Dead/None** - Loan is not created or has been claimed and can be burned.
* **Running** - Loan is created by passing offer data and offer signature signed by a lender.
* **Paid back** - Loan has been fully paid back before the expiration date. The LOAN owner is able to claim lent credit + interest.
* **Expired** - Loan had not been fully paid back before expiration date. LOAN owner is able to claim collateral.

#### State diagram

![](<../.gitbook/assets/PWN\_contracts-State diagram.drawio (1).png>)

### Loan token struct

Each LOAN token is defined by the LOAN token struct.&#x20;

**`LOAN`** token struct has the following properties:

| Type                                                                | Name              | Comment                                                                                                            |
| ------------------------------------------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------ |
| `uint8`                                                             | `status`          | see [LOAN token lifecycle](pwn-loan.md#loan-token-lifecycle)                                                       |
| `address`                                                           | `borrower`        | Address of the borrower - stays the same for entire lifespan of the token                                          |
| `uint32`                                                            | `duration`        | Loan duration in seconds                                                                                           |
| `uint40`                                                            | `expiration`      | Unix timestamp (in seconds) setting up the default deadline                                                        |
| `MultiToken.Asset` (see [Asset struct](multitoken.md#asset-struct)) | `collateral`      | Asset used as a loan collateral. Consisting of another `Asset` struct defined in the MultiToken library            |
| `MultiToken.Asset` (see [Asset struct](multitoken.md#asset-struct)) | `asset`           | Asset to be borrowed by lender to borrower. Consisting of another `Asset` struct defined in the MultiToken library |
| `uint256`                                                           | `loanRepayAmount` | Amount of LOAN asset to be repaid                                                                                  |

## Functions

This contract inherits from the ERC1155 token standard. This comes with all of the ERC1155 functionalities like transfers etc. Please read the [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) specification for more details.&#x20;

### View functions

Functions that don't modify the state of the contract. These functions are used to get information about the LOAN token.

#### `getStatus`

Returns LOAN status number. To understand what different status means please refer to [state diagram](pwn-loan.md#state-diagram) above.

This function takes one argument:

* `uint256`**`_loanId`** - Loan id

#### `getExpiration`

Returns the exact expiration time of a particular LOAN as a unix timestamp in seconds.&#x20;

This function takes one argument:

* `uint256`**`_loanId`** - Loan id

#### `getDuration`

Returns loan duration period of particular LOAN in seconds.&#x20;

This function takes one argument:

* `uint256`**`_loanId`** - Loan id

#### `getBorrower`

Returns borrower address of particular LOAN.

This function takes one argument:

* `uint256`**`_loanId`** - Loan id

#### `getCollateral`

Returns collateral asset of a particular LOAN. By asset we mean Asset struct described in [MultiToken](multitoken.md).

This function takes one argument:

* `uint256`**`_loanId`** - Loan id

#### `getLoanAsset`

Returns loan asset of particular LOAN. By asset we mean Asset struct described in [MultiToken](multitoken.md).

This function takes one argument:

* `uint256`**`_loanId`** - Loan id

#### `getLoanRepayAmout`

Returns loan repay amount of a particular LOAN.&#x20;

This function takes one argument:

* `uint256`**`_loanId`** - Loan id

#### `isRevoked`

Utility function to find out if offer is revoked. Returns a boolean.&#x20;

This function takes one argument:

* `bytes32`**`_offerHash`** - Hash of the offer struct

## Events

PWNLOAN contract defines four events and no custom errors.&#x20;

```
event LOANCreated(uint256 indexed loanId, address indexed lender, bytes32 indexed offerHash);
event OfferRevoked(bytes32 indexed offerHash);
event PaidBack(uint256 loanId);
event LOANClaimed(uint256 loanId);
```

### `LOANCreated`

LOANCreated event is emitted when a borrower accepts an offer.&#x20;

This event has three parameters:

* `uint256 indexed`**`loanId`** - The ID of the LOAN token
* `address indexed`**`lender`** - Address of the lender
* `bytes32 indexed`**`offerHash`** - [EIP-712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md) computed [hash](https://docs.ethers.io/v5/api/utils/hashing/#TypedDataEncoder-hash) of the offer struct

### `OfferRevoked`

OfferRevoked event is emitted when a lender decides to revoke an offer.&#x20;

This event has one parameter:

* `bytes32 indexed`**`offerHash`** - [EIP-712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md) computed [hash](https://docs.ethers.io/v5/api/utils/hashing/#TypedDataEncoder-hash) of the offer struct

### `PaidBack`

PaidBack event is emitted when a borrower repays a loan.&#x20;

This event has one parameter:

* `uint256`**`loanId`** - The ID of the LOAN token

{% hint style="info" %}
Lenders can listen to this event to get notified when their loan has been paid back.&#x20;
{% endhint %}

### `LOANClaimed`

LOANClaimed event is emitted when a lender claims the repaid amount or loan underlying collateral in case of a default.&#x20;

This event has one parameter:

* `uint256`**`loanId`** - The ID of the LOAN token
