# Introduction

The PWN Protocol can be challenging to understand at first. This article is a good starting point if you’re new to PWN and want to learn more about the technical aspects. We will look at **the most important smart contracts, how they interact with each other and how users interact with the protocol.** Lastly, we will talk about immutability and trustless properties.&#x20;

{% hint style="info" %}
Note, this article does not provide a complete overview of the PWN V1 Protocol. Please read the [Deep Dive](deep-dive.md) and the [Smart Contract Reference](smart-contract-reference/) if you’re looking for a comprehensive overview of the PWN V1 Protocol.
{% endhint %}

The following topics are covered in this article:

* [Extendable Modular Design](introduction.md#extendable-modular-design)
* [How do users interact with the protocol?](introduction.md#how-do-users-interact-with-the-protocol)
* [Security guarantees](introduction.md#security-guarantees)

{% hint style="success" %}
This article assumes you’re familiar with the fundamentals of Ethereum, DeFi, and the high-level basics of PWN. If you're not, we suggest visiting the [ethereum.org](http://ethereum.org) website and our [What is PWN](https://docs.pwn.xyz/) documentation page to deepen your understanding before diving into this article.
{% endhint %}

## Extendable Modular Design

The PWN Protocol has been designed to be expanded on with new features, while keeping the immutability properties, as demonstrated in the diagram below. In this part, we will dive into the technical aspects of this design.

<figure><img src="../../.gitbook/assets/Diagram V1 simple.png" alt=""><figcaption><p>Simplified Architecture of the PWN Protocol</p></figcaption></figure>

### The PWN Config and Hub Contracts

The PWN Hub contract defines which contracts are valid parts of the PWN Protocol by storing a tag for each valid contract of the protocol. All contracts in the PWN Protocol use the Hub to manage access control and verify genuine PWN Protocol contracts.&#x20;

The PWN Config contract defines all the configuration details of the PWN Protocol (e.g. fee size).

### Extendability

The Hub makes it possible to add and remove contracts in the protocol thus enabling the addition of new loan types, which makes the PWN Protocol extendable by design.&#x20;

And there is one more thing - not only is it possible to add new loan types, but it’s also possible to add new offer and loan request types. The loan request and offer contracts can be shared (e.g. two offer types can share one loan type and vice versa). Thanks to these properties we can limit the number of smart contracts that must be deployed.

{% hint style="info" %}
Note, already-running loans are unaffected by any changes in the PWN Hub.
{% endhint %}

### LOAN Token

Each time a new loan is started on the PWN Protocol a new LOAN token is minted and transferred to the lender. The LOAN token represents the loan and is burned when claiming repayment or defaulted collateral. Since the LOAN token implements the [ERC-721](https://ethereum.org/en/developers/docs/standards/tokens/erc-721/) specification it can be sold or even used as collateral.

### Loan, Offer, and Request Types

As you can see in the diagram below, the main Loan contract in each loan type acts as a Vault for the collateral of loans of that type. Each Loan contract can accept multiple loan requests and offer types, allowing for greater reusability of already deployed contracts.

{% hint style="info" %}
One type of loan is called a Simple Loan. In this type of loan, a borrower provides collateral and receives tokens from a lender. The borrower has two options: they can repay the agreed amount and retrieve their collateral, or they can choose to default on the loan. Once the repayment is made or the default occurs, the lender has the right to claim the repaid tokens or the collateral.
{% endhint %}

<figure><img src="../../.gitbook/assets/Loan type.png" alt=""><figcaption></figcaption></figure>

## How do users interact with the protocol?

As a user, you will primarily be interacting with the Loan contracts. Since the Loan contracts act as Vaults, you will be making approvals towards the Loan contracts.&#x20;

Usual interactions with the Loan contract are:

* Start a loan
* Repay a loan
* Claim repayment or collateral
* Loan-type-specific actions
* Revoke signed offer or loan request

## Security guarantees

Let’s look at the security properties of the PWN Protocol and see what are the potential risks.

To understand the security profile of the PWN Protocol we need to know what entities own the smart contracts. Look at the following diagram:

<figure><img src="../../.gitbook/assets/Ownership diagram.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
We haven’t covered the PWN Deployer contract in this article for simplicity's sake. PWN Deployer’s only purpose is to deploy contracts on different chains with the same address.
{% endhint %}

As you can see there are two entities. The **PWNDAO** and the **Protocol Team**. The contracts enforce that these two are separate entities (addresses). Both of these entities also have a time lock for their operations. The protocol team and the product team (PWN DAO) have a delay of 4 days. At the moment both of these entities are 2-of-4 multi-signature wallets.

{% hint style="info" %}
Both the time lock delay and the minimal signatures threshold on the multisigs are expected to increase as the protocol matures.&#x20;
{% endhint %}

### PWN Config

The PWNDAO can change the core protocol parameters through the PWN Config. The parameters are the following:

* Fee size
* Fee collector address
* Metadata URI

To help prevent any attacks there is a hard cap of 10 % on the fee size.

The Protocol Team can upgrade the PWN Config (since technically it’s a [proxy](https://docs.openzeppelin.com/contracts/4.x/api/proxy#transparent\_proxy)) but cannot change the parameters. The reason for the PWN Config being upgradable is to make sure new parameters can be added in the future.

### PWN Hub

The Protocol Team owns this contract and is therefore responsible for adding new contracts to the protocol and deprecating old contracts.

Even in the case that the Protocol Team was a malicious entity, it could only pause the creation of new loans. Already running loans would be unaffected and all assets would still be completely safe.

## Conclusion

We discussed the main components of the PWN V1 Protocol, how users interact with the protocol, the security properties of the protocol, and how the protocol is designed to be extendable by adding new features. If you’re looking for a more comprehensive overview of the PWN Protocol, we suggest reading the [Deep Dive](deep-dive.md) and the [Smart Contract Reference](smart-contract-reference/).
