# MultiToken

## 1. Summary

MultiToken is a solidity library unifying ERC20, ERC721 & ERC1155 transfers by wrapping them into a MultiToken Asset struct.

## 2. Important links

* **Source code**
  * [GitHub](https://github.com/PWNFinance/MultiToken)
* ****[**NPM package**](https://www.npmjs.com/package/@pwnfinance/multitoken)****
  * If you want to use the MultiToken library in your solidity project you can install our NPM package.
  * Run: `npm install @pwnfinance/multitoken`

## 3. Contract Details

* _MultiToken.sol_ contract is written in Solidity version 0.8.0, but can be compiled with any solidity compiler version 0.8.\*

### Features

The library defines a token asset as a struct of token identifiers. **It wraps transfer, allowance and balance check calls of the following token standards:**

* ERC20
* ERC721
* ERC1155

Unifying the function calls used within the PWN context, so we don't have to worry about handling these token standards individually.

## Functions

### `transferAsset`

Function for transfer calls on various token interfaces.&#x20;

This function takes two arguments:

* `Asset memory`**`_asset`** - [Asset](multitoken.md#asset-struct) struct defining all necessary context of a token
* `address`**`_dest`** - Destination address

### `transferAssetFrom`

Function for transfer from calls on various token interfaces.&#x20;

This function takes three arguments:

* `Asset memory`**`_asset`** - [Asset](multitoken.md#asset-struct) struct defining all necessary context of a token
* `address`**`_source`** - Account/address that provided the allowance
* `address`**`_dest`** - Destination address

### `balanceOf`

Function for checking balances on various token interfaces.&#x20;

This function takes two arguments:

* `Asset memory`**`_asset`** - [Asset](multitoken.md#asset-struct) struct defining all necessary context of a token
* `address`**`_target`** - Target address to be checked

### `approveAsset`

Function for approving calls on various token interfaces.&#x20;

This function takes two arguments:

* `Asset memory`**`_asset`** - [Asset](multitoken.md#asset-struct) struct defining all necessary context of a token
* `address`**`_target`** - Target address to be checked

### Asset struct

Each asset is defined by the **`Asset`** struct and has the following properties:

| Type                                                | Name           | Comment                                          |
| --------------------------------------------------- | -------------- | ------------------------------------------------ |
| `address`                                           | `assetAddress` | Address of the token contract defining the asset |
| `MultiToken.Category` (category is described below) | `category`     | Corresponding asset category                     |
| `uint256`                                           | `amount`       | Amount of fungible tokens or 0 -> 1              |
| `uint256`                                           | `id`           | TokenID of an NFT or 0                           |

A **category** is defined as an [enum](https://docs.soliditylang.org/en/v0.8.12/structure-of-a-contract.html?highlight=enum#enum-types) and can have values `ERC20`, `ERC721` or `ERC1155`.

## Events

MultiToken library does not define any events or custom errors.&#x20;
