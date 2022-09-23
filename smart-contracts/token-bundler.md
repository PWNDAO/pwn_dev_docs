# Token Bundler

## 1. Summary

The contract enables **bundling ERC20, ERC721, and/or ERC1155 tokens into a single ERC1155 token** (the Bundle token).

## 2. Important links

* **Deployment addresses**
  * Mainnet: [0x_19e3293196aee99BB3080f28B9D3b4ea7F232b8d_](https://etherscan.io/address/0x19e3293196aee99BB3080f28B9D3b4ea7F232b8d)__
  * Polygon: [0x_e52405604bF644349f57b36Ca6E85cf095faB8dA_](https://polygonscan.com/address/0xe52405604bf644349f57b36ca6e85cf095fab8da)__
  * Görli: [0x_A0610921062f99D720710d9763EE8cb1fCF7a845_](https://goerli.etherscan.io/address/0xA0610921062f99D720710d9763EE8cb1fCF7a845)__
  * Mumbai: [0x_a5e63d2d2DcA259270b6B8FeD95e0b420d929e58_](https://mumbai.polygonscan.com/address/0xa5e63d2d2DcA259270b6B8FeD95e0b420d929e58)__
* **Source code**
  * [GitHub](https://github.com/PWNFinance/TokenBundler/tree/master)****
* **ABI**
  * [JSON](https://api.etherscan.io/api?module=contract\&action=getabi\&address=0x19e3293196aee99BB3080f28B9D3b4ea7F232b8d)
  * [Text](http://api.etherscan.io/api?module=contract\&action=getabi\&address=0x19e3293196aee99BB3080f28B9D3b4ea7F232b8d\&format=raw)

## 3. Contract Details

* _TokenBundler.sol_ contract is written in Solidity version 0.8.16

### Features

The owner of the bundle NFT has rights to:

* Transferring ownership of the bundle token to another address
* Unwrap the entire bundle resulting in burning the bundle token and gaining ownership over the wrapped tokens

## Functions

This contract inherits from the ERC1155 token standard. This comes with all of the ERC1155 functionalities like transfers etc. Please read the [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) specification for more details.&#x20;

### **`create`**

Mint bundle token and transfers assets to Bundler contract.

{% hint style="warning" %}
Make sure to approve all bundled assets towards the Token Bundler contract before calling this function.
{% endhint %}

This function takes one argument:

* `MultiToken.Asset[] memory`**`_assets`** - List of assets to include in a bundle

{% hint style="info" %}
See [MultiToken](multitoken.md) for more information about the argument type.
{% endhint %}

Returns:

* `uint256` - Bundle id

### `unwrap`

Burns the bundle token and transfers assets to the caller.

{% hint style="warning" %}
The caller of this function has to be the bundle owner.&#x20;
{% endhint %}

This function takes one argument:

* `uint256`**`_bundleId`** - Bundle id to unwrap

### View functions

Functions that don't modify the state of the contract. These functions are used to get information about the bundle token.

#### **`token`**

Each token has its nonce. This function returns an Asset struct (see [MultiToken](multitoken.md#asset-struct)) for a provided token nonce.&#x20;

This function takes one argument:

* `uint265`` `**`_tokenId`** - Token nonce from bundle asset list.

#### **`bundle`**

Returns a list of assets in a bundle as a `uint256[]`.

This function takes one argument:

* `uint256`**`_bundleId`** - Bundle id

#### **`maxSize`**

Returns maximum bundle size.&#x20;

This function does not take any argument.

## Events

Token Bundler contract defines two events and no custom errors.

```
event BundleCreated(uint256 indexed id, address indexed creator);
event BundleUnwrapped(uint256 indexed id);
```

### `BundleCreated`

BundleCreated event is emitted when a new bundle is created.

This event has two parameters:

* `uint256 indexed`**`id`** - Id of the bundle
* `address indexed`**`creator`** - Address of the bundle creator

### `BundleUnwrapped`

BundleUnwrapped event is emitted when a bundle is unwrapped and burned.

This event has one parameter:

* `uint256 indexed`**`id`** - Id of the unwrapped bundle.&#x20;
