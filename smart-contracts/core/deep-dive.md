# Deep Dive

Welcome to our deep-dive article on PWN Protocol. This article provides a comprehensive understanding of the PWN Protocol architecture and properties. With each section, we will build up the PWN Protocol architecture diagram and finally end up with the full picture.

{% hint style="success" %}
This article assumes you’re familiar with EVM, Solidity, and PWN. If you're not we suggest visiting the [ethereum.org](http://ethereum.org) website and reading our [Introduction to the PWN Protocol](introduction.md).
{% endhint %}

Here’s an overview of the topics covered in this article:

* [Hub](deep-dive.md#hub) - What contracts are part of the protocol?
* [Config](deep-dive.md#config) - Definition of protocol parameters
* [The LOAN (Vault)](deep-dive.md#the-loan-vault) - Business logic
* [Offers and loan request types](deep-dive.md#offers-and-loan-request-types)
* [LOAN token](deep-dive.md#loan-token) - Tokenised debt
* [Nonces](deep-dive.md#nonces) - Identify unique offers and loan requests
* [Miscellaneous](deep-dive.md#miscellaneous)
* [What now?](deep-dive.md#what-now)

## Hub

### Tags

The [PWNHub.sol](smart-contract-reference/pwn-hub/) contract stores tags for each contract in the protocol and therefore **defines what contracts are a valid part of the protocol**. Let’s look at the technical implementation:

```solidity
mapping (address => mapping (bytes32 => bool)) private tags;
```

The outer mapping is indexing the address of a contract, and the inner mapping is indexing a `bytes32` tag. The value of the inner mapping is a boolean indicating whether or not the contract address is part of the protocol. The tag value is determined by the [PWNHubTags.sol](smart-contract-reference/pwn-hub/tags.md) library. Here’s an example of a tag:

```solidity
bytes32 internal constant ACTIVE_LOAN = keccak256("PWN_ACTIVE_LOAN");
```

Tags can be changed by the owner (we will talk about ownership aspects later) through the `setTag` function. There’s also the `setTags` function that changes multiple tags in one call. The `hasTag` view function returns a boolean value given a contract `address` and a `bytes32` tag.

Other contracts in the protocol inherit the [PWNHubAccessControl.sol](smart-contract-reference/pwn-hub/access-control.md). This contract defines two modifiers:

1. `onlyActiveLoan` - checks if the caller has an "ACTIVE\_LOAN" tag in the hub contract, otherwise, it reverts with the [CallerMissingHubTag](smart-contract-reference/miscellaneous/pwn-errors.md) error.
2. `onlyWithTag` - checks if the caller has a specific tag in the PWN Hub contract, otherwise it reverts. The tag is passed as an argument to the modifier.

### Ownership

The Protocol Team owns this contract and is therefore responsible for adding new contracts to the protocol and deprecating old contracts.

{% hint style="info" %}
Even if the Protocol Team was a malicious entity it could only pause the creation of new loans. Already running loans would be unaffected and all assets would still be safe.
{% endhint %}

## Config

### Parameters

The [PWNConfig.sol](smart-contract-reference/pwn-config.md) contract stores the core parameters of the protocol. The parameters are the following:

* Fee size
* Fee collector address
* Metadata URI

To prevent any attacks there is a hard cap of 10 % on the fee size.

### Proxy

The PWN Config contract is meant to be used behind a proxy contract. This enables the addition and removal of parameters as the protocol evolves. The proxy implementation used is the [TransparentUpgradableProxy](https://docs.openzeppelin.com/contracts/4.x/api/proxy#transparent\_proxy) from OpenZeppelin.

### Ownership

There are two entities affecting this contract. One owner is the owner of PWNConfig. The other is the owner of the proxy and for the sake of clarity, we will call this entity admin. The admin (Protocol Team) is able to add and remove parameters of the protocol by upgrading the PWNConfig through the proxy. The owner (PWNDAO) is able to change the parameters of the protocol. These two cannot be the same entities to prevent otherwise possible attacks.

<figure><img src="../../.gitbook/assets/Ownership diagram - 1.png" alt=""><figcaption></figcaption></figure>

## The LOAN (Vault)

The LOAN contracts are the primary contracts doing business logic. Given a loan request and an offer (we will talk about these in more detail later), the contract creates a loan. There can be an unlimited number of these contracts, we call them loan types. The LOAN contracts can implement any logic, for example, simple loans or mortgage-type loans. Each loan type has to be added to the PWN Hub by the Protocol Team.

### PWNVault

The LOAN contracts inherit the [PWNVault.sol](smart-contract-reference/pwn-vault.md) contract. The Vault is used for transferring and managing collateral and loan assets. The Vault contains three transfer functions, `_pull`, `_push`, and `_pushFrom`. The `_pull` function pulls an asset into the Vault from the borrower address, assuming a prior token approval was made to the LOAN (Vault) address. The `_pull` function is typically used to transfer the collateral from a borrower to the Vault. The `_push` function pushes an asset from the Vault to a defined recipient, such as a borrower or a lender. The `_push` function is typically used to transfer the collateral back to a borrower when a loan is repaid. The `_pushFrom` function pushes an asset from one address to another, assuming a prior token approval was made to the Vault address. The `_pushFrom` function is typically used to transfer borrowed tokens from a lender to a borrower.

### SimpleLoan

The first loan type in the PWN Protocol is the Simple Loan. In this loan, a borrower provides collateral and the lender lends ERC-20 tokens to the borrower. The borrower must repay an agreed amount of the borrowed tokens before the loan matures. If the borrower does not repay the loan the lender can claim the collateral. There is also an option for the lender to extend the maturity date of a running loan by up to 30 days.

{% hint style="info" %}
Simple Loans can be extended by the lender by more than 30 days but not in one transaction and only by 30 days from the transaction block inclusion. This is a security measure to help protect lenders.&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/Ownership diagram - 2.png" alt=""><figcaption></figcaption></figure>

## Offers and loan request types

A LOAN contract we’ve just covered has one feature we’ve not mentioned yet. It can accept more offer and loan request types!&#x20;

{% hint style="info" %}
For example, the Simple Loan type can accept offers made on entire collections. That means the user can make an offer on the entire BAYC collection and the borrowers don’t have to wait for someone to make an offer on their specific Ape and can instead accept the so-called Collection Offer.
{% endhint %}

### LoanTermsFactory

Each offer and loan request type implements the [LoanTermsFactory](smart-contract-reference/loan-types/simple-loan/simple-loan-terms-factory.md) contract for a given loan type (e.g. Simple Loan). This contract defines only one function called `createLOANTerms` and as the name suggests it creates loan terms for a given offer and loan request. All offers and loan requests in the PWN Protocol are signed typed structs according to the [EIP-712](https://eips.ethereum.org/EIPS/eip-712).&#x20;

{% hint style="info" %}
Keep in mind that although offer and loan requests can be created on-chain users will typically create and sign offers and loan requests off-chain to save unnecessary gas fees.
{% endhint %}

<figure><img src="../../.gitbook/assets/Ownership diagram - 3.png" alt=""><figcaption></figcaption></figure>

## LOAN token

### Functionality

The [PWNLOAN.sol](smart-contract-reference/pwn-loan.md) is an [ERC-721](https://eips.ethereum.org/EIPS/eip-721) token contract. Each token represents a unique loan in the PWN Protocol. Only the LOAN (Vault) contracts are allowed to mint or burn these tokens. There’s also a `tokenURI` function that returns the metadata URI for a given LOAN token ID and a mapping of LOAN token IDs to contract addresses that minted them. Furthermore, this contract implements the [ERC-165](https://eips.ethereum.org/EIPS/eip-165) and [ERC-5646](https://eips.ethereum.org/EIPS/eip-5646) standards.

### ERC-5646

EIP-5646 provides a standardized interface that allows for the unambiguous identification of the state of a mutable token without requiring any knowledge of the token's implementation details. The EIP specification defines the `getStateFingerprint` function, which returns a unique value that must change when the token's state changes, and includes all state properties that may change during its lifecycle, excluding immutable properties. By providing this minimum interface, protocols can support mutable tokens without the need for specific implementation knowledge, enabling greater interoperability and reducing the bottleneck effect that arises from requiring explicit support for every new token.

<figure><img src="../../.gitbook/assets/Ownership diagram - 4.png" alt=""><figcaption></figcaption></figure>

## Nonces

### Usage

Each offer (or loan request) struct has a nonce value, represented as a `uint256`. Once an offer (or loan request) is used to create a loan, its nonce is considered revoked, and any other offers (or loan requests) with the same nonce will be invalid, This allows a lender to make multiple offers, but only one of them can be accepted while the rest are automatically revoked.

{% hint style="info" %}
An exception to this rule is so-called persistent offers that stay valid even after being used to start a loan.
{% endhint %}

### Revoking a nonce

If an account wants to manually revoke an offer (loan request) it can do so with the `revokeNonce` function passing the nonce as an argument. This function is implemented by the [PWNRevokedNonce.sol](smart-contract-reference/pwn-revoked-nonce.md), Loan Request and Offer contracts.&#x20;

{% hint style="info" %}
There’re two `revokeNonce` functions with a different function signature. One takes only the nonce as an argument and the other also takes an address. The latter enables to revoke nonces for other accounts, but it’s only callable by account with a tag in the Hub.&#x20;
{% endhint %}

## Miscellaneous

### Errors

[PWNErrors.sol](smart-contract-reference/miscellaneous/pwn-errors.md) defines all custom errors in the PWN Protocol.

### FeeCalculator

[PWNFeeCalculator.sol](smart-contract-reference/miscellaneous/pwn-fee-calculator.md) library implements the `calculateFeeAmount` function. This function calculates the token amount that will be paid to the protocol as a fee based on the borrowed amount and the protocol fee defined in [PWNConfig.sol](smart-contract-reference/pwn-config.md).&#x20;

### SignatureChecker

[PWNSignatureChecker.sol](smart-contract-reference/miscellaneous/pwn-signature-checker.md) library implements the `isValidSignatureNow` function. This function checks the validity of a given signature of a given hash and signer address. The check supports both EOA and contract accounts. The library is a modification of the [SignatureChecker](https://docs.openzeppelin.com/contracts/4.x/api/utils#SignatureChecker) library from Open Zeppelin extended by support for [EIP-2098](https://eips.ethereum.org/EIPS/eip-2098) compact signatures.

### Deployer

[PWNDeployer.sol](smart-contract-reference/pwn-deployer.md) deploys other PWN protocol contracts with the CREATE2 opcode. This enables having the same contract addresses on all EVM-compatible blockchains.

### Owners

Throughout this article, we’ve mentioned two owners that manage some contracts in the PWN Protocol. Let’s look at these entities in detail. Keep in mind that although these accounts have a lot of power they still cannot alter already existing loans. Even if both of these entities are malicious, your already existing loan is safe!&#x20;

Both of these entities also have a time lock for their operations. The Protocol Team has a delay of 14 days and the Product Team (PWN DAO) has 7 days.&#x20;

#### Protocol Team

The Protocol Team is responsible for managing and upgrading the protocol smart contracts. At the time of writing, there is no plan to hand over this role to the community as there’re no serious security risks associated with this role being centralized.

#### Product team / PWNDAO

The product team is responsible for updating the parameters of the protocol (e.g. updating the fee). At the time of the launch of the PWN Protocol V1, this is a Safe Multisig account owned by the PWN team. As we progress to become more decentralized this role will be taken over by the PWN DAO.

<figure><img src="../../.gitbook/assets/Ownership diagram - 5.png" alt=""><figcaption></figcaption></figure>

## What now?

This deep dive article has provided a comprehensive analysis of the architecture and properties of the PWN Protocol. If you want to learn more see our [Smart Contract Reference](smart-contract-reference/) for all contracts and the [`pwn_contracts`](https://github.com/PWNFinance/pwn\_contracts) GitHub repository. Check out the tests to see how other contracts can interact with the protocol. If you have any questions feel free to reach out to us on our [Discord](https://discord.gg/aWghBQSdHv).
