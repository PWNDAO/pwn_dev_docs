# PWN Config

## 1. Summary

The PWNConfig.sol contract stores the core parameters of the protocol. The parameters are the following:

* Fee size
* Fee collector address
* [Metadata URI](https://docs.opensea.io/docs/metadata-standards)

To prevent any attacks there is a hard cap of 10 % on the fee size.

The config contract is meant to be used behind a proxy contract, which enables the addition and removal of parameters as the protocol evolves. The proxy implementation used is the [TransparentUpgradableProxy](https://docs.openzeppelin.com/contracts/4.x/api/proxy#transparent\_proxy) from Open Zeppelin.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/config/PWNConfig.sol" %}
GitHub
{% endembed %}

{% file src="../../../.gitbook/assets/PWNConfig.json" %}
JSON ABI
{% endfile %}

## 3. Contract details

* _PWNConfig.sol_ is written in Solidity version 0.8.16

### Features

* Stores PWN Protocol parameters

### Inherited contracts, implemented Interfaces and ERCs

* [Ownable2Step](https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable2Step)
* [Initializable](https://docs.openzeppelin.com/contracts/4.x/api/proxy#Initializable)

### Functions

<details>

<summary><code>setFee</code></summary>

#### Overview

&#x20;Updates the fee parameter of the PWN Protocol.&#x20;

This function takes one argument supplied by the owner:

* `uint16`**`_fee`** - New fee in basis points

#### Implementation

```solidity
function setFee(uint16 _fee) external onlyOwner {
    _setFee(_fee);
}
```

</details>

<details>

<summary><code>setFeeCollector</code></summary>

#### Overview

Updates the address that collects the PWN Protocol fees.

This function takes one argument supplied by the owner:

* `address`**`_feeCollector`**

#### Implementation

```solidity
function setFeeCollector(address _feeCollector) external onlyOwner {
    _setFeeCollector(_feeCollector);
}
```

</details>

<details>

<summary><code>setLoanMetadataUri</code></summary>

#### Overview

Updates the metadata URI for a loan contract.

This function takes two arguments supplied by the owner:

* `address`**`loanContract`** - Address of the loan contract for which the URI is updated
* `string memory`**`metadataUri`** - New URI

#### Implementation

```solidity
function setLoanMetadataUri(address loanContract, string memory metadataUri) external onlyOwner {
    loanMetadataUri[loanContract] = metadataUri;
    emit LoanMetadataUriUpdated(loanContract, metadataUri);
}
```

</details>

### Events

The PWN Config contract defines one event and no custom errors.

```solidity
event FeeUpdated(uint16 oldFee, uint16 newFee);
event FeeCollectorUpdated(address oldFeeCollector, address newFeeCollector);
event LoanMetadataUriUpdated(address indexed loanContract, string newUri);
```

<details>

<summary><code>FeeUpdated</code></summary>

FeeUpdated event is emitted when the protocol fee is updated. Fees are represented in basis points.

This event has two parameters:

* `uint16`**`oldFee`**
* `uint16`**`newFee`**

</details>

<details>

<summary><code>FeeCollectorUpdated</code></summary>

FeeCollectorUpdated event is emitted when the protocol fees collector address is updated.

This event has two parameters:

* `address`**`oldFeeCollector`**
* `address`**`newFeeCollector`**

</details>

<details>

<summary><code>LoanMetadataUriUpdated</code></summary>

LoanMetadataUriUpdated event is emitted when a metadata URI for a loan contract is updated.

This event has two parameters:

* `address indexed`**`loanContract`** - Address of the loan contract for which the URI is updated
* `string`**`newUri`**

</details>
