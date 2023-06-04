# Simple Loan

## 1. Summary

PWNSimpleLoan.sol contract manages the simple loan type in the PWN protocol. This contract also acts as a Vault for all assets used in simple loans. The minimum duration of a simple loan is 10 minutes.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/simple/loan/PWNSimpleLoan.sol" %}
GitHub
{% endembed %}

{% file src="../../../../../.gitbook/assets/PWNSimpleLoan.json" %}
JSON ABI
{% endfile %}

## 3. Contract details

* _PWNSimpleLoan.sol_ is written in Solidity version 0.8.16

### Features

* Manages simple loan flow
  * Creation
  * Repayment
  * Claim
* Implements an option for the lender to extend the expiration (maturity) date of a loan

{% hint style="info" %}
The expiration of a loan can be extended by a maximum of 30 days into the future. This is a measure to protect lenders from accidentally extending a loan maturity date too far. Lenders can extend a loan expiration date an unlimited amount of times meaning a loan expiration date can be extended indefinitely.&#x20;
{% endhint %}

### Inherited contracts, implemented Interfaces and ERCs

* [PWNVault](../../pwn-vault.md)
* [IERC5646](https://eips.ethereum.org/EIPS/eip-5646)
* [IPWNLoanMetadataProvider](https://github.com/PWNFinance/pwn\_contracts/blob/master/src/loan/token/IPWNLoanMetadataProvider.sol)

### Functions

<details>

<summary><code>createLOAN</code></summary>

#### Overview

Users use this function to start a simple loan in the PWN protocol.

This function takes five arguments supplied by the caller:

* `address`**`loanTermsFactoryContract`** - Address of a loan terms factory contract. Needs to have `SIMPLE_LOAN_TERMS_FACTORY` tag in the [PWN Hub](../../pwn-hub/).
* `bytes calldata`**`loanTermsFactoryData`** - Encoded data for a loan terms factory
* `bytes calldata`**`signature`** - Signed loan factory data. Can be empty if a offer/request was created via an on-chain transaction
* `bytes calldata`**`loanAssetPermit`** - Permit data for a loan asset signed by a lender
* `bytes calldata`**`collateralPermit`** - Permit data for collateral signed by a borrower

#### Implementation

```solidity
function createLOAN(
    address loanTermsFactoryContract,
    bytes calldata loanTermsFactoryData,
    bytes calldata signature,
    bytes calldata loanAssetPermit,
    bytes calldata collateralPermit
) external returns (uint256 loanId) {
    // Check that loan terms factory contract is tagged in PWNHub
    if (hub.hasTag(loanTermsFactoryContract, PWNHubTags.SIMPLE_LOAN_TERMS_FACTORY) == false)
        revert CallerMissingHubTag(PWNHubTags.SIMPLE_LOAN_TERMS_FACTORY);

    // Build PWNLOANTerms.Simple by loan factory
    PWNLOANTerms.Simple memory loanTerms = PWNSimpleLoanTermsFactory(loanTermsFactoryContract).createLOANTerms({
        caller: msg.sender,
        factoryData: loanTermsFactoryData,
        signature: signature
    });

    // Check loan asset validity
    if (MultiToken.isValid(loanTerms.asset) == false)
        revert InvalidLoanAsset();

    // Check collateral validity
    if (MultiToken.isValid(loanTerms.collateral) == false)
        revert InvalidCollateralAsset();

    // Mint LOAN token for lender
    loanId = loanToken.mint(loanTerms.lender);

    // Store loan data under loan id
    LOAN storage loan = LOANs[loanId];
    loan.status = 2;
    loan.borrower = loanTerms.borrower;
    loan.expiration = loanTerms.expiration;
    loan.loanAssetAddress = loanTerms.asset.assetAddress;
    loan.loanRepayAmount = loanTerms.loanRepayAmount;
    loan.collateral = loanTerms.collateral;

    emit LOANCreated(loanId, loanTerms);

    // Transfer collateral to Vault
    _permit(loanTerms.collateral, loanTerms.borrower, collateralPermit);
    _pull(loanTerms.collateral, loanTerms.borrower);

    // Permit spending if permit data provided
    _permit(loanTerms.asset, loanTerms.lender, loanAssetPermit);

    uint16 fee = config.fee();
    if (fee > 0) {
        // Compute fee size
        (uint256 feeAmount, uint256 newLoanAmount) = PWNFeeCalculator.calculateFeeAmount(fee, loanTerms.asset.amount);

        if (feeAmount > 0) {
            // Transfer fee amount to fee collector
            loanTerms.asset.amount = feeAmount;
            _pushFrom(loanTerms.asset, loanTerms.lender, config.feeCollector());

            // Set new loan amount value
            loanTerms.asset.amount = newLoanAmount;
        }
    }

    // Transfer loan asset to borrower
    _pushFrom(loanTerms.asset, loanTerms.lender, loanTerms.borrower);
}
```

</details>

<details>

<summary><code>repayLOAN</code></summary>

#### Overview

Borrowers use this function to repay simple loans in the PWN Protocol. &#x20;

This function takes two arguments supplied by the caller:

* `uint256`**`loanId`** - ID of the loan that is being repaid
* `bytes calldata`**`loanAssetPermit`** - Permit data for a loan asset signed by a borrower

#### Implementation

```solidity
function repayLOAN(
    uint256 loanId,
    bytes calldata loanAssetPermit
) external {
    LOAN storage loan = LOANs[loanId];
    uint8 status = loan.status;

    // Check that loan is not from a different loan contract
    if (status == 0)
        revert NonExistingLoan();
    // Check that loan is running
    else if (status != 2)
        revert InvalidLoanStatus(status);

    // Check that loan is not expired
    if (loan.expiration <= block.timestamp)
        revert LoanDefaulted(loan.expiration);

    // Move loan to repaid state
    loan.status = 3;

    // Transfer repaid amount of loan asset to Vault
    MultiToken.Asset memory repayLoanAsset = MultiToken.Asset({
        category: MultiToken.Category.ERC20,
        assetAddress: loan.loanAssetAddress,
        id: 0,
        amount: loan.loanRepayAmount
    });

    _permit(repayLoanAsset, msg.sender, loanAssetPermit);
    _pull(repayLoanAsset, msg.sender);

    // Transfer collateral back to borrower
    _push(loan.collateral, loan.borrower);

    emit LOANPaidBack(loanId);
}
```

</details>

<details>

<summary><code>claimLOAN</code></summary>

#### Overview

Holders of LOAN tokens (lenders) use this function to claim a repaid loan or defaulted collateral. The claimed asset is transferred to the LOAN token holder and the LOAN token is burned.&#x20;

This function takes one argument supplied by the caller:

* `uint256`**`loanId`** - ID of the loan that is being claimed

#### Implementation

```solidity
function claimLOAN(uint256 loanId) external {
    LOAN storage loan = LOANs[loanId];

    // Check that caller is LOAN token holder
    if (loanToken.ownerOf(loanId) != msg.sender)
        revert CallerNotLOANTokenHolder();

    if (loan.status == 0) {
        revert NonExistingLoan();
    }
    // Loan has been paid back
    else if (loan.status == 3) {
        MultiToken.Asset memory loanAsset = MultiToken.Asset({
            category: MultiToken.Category.ERC20,
            assetAddress: loan.loanAssetAddress,
            id: 0,
            amount: loan.loanRepayAmount
        });

        // Delete loan data & burn LOAN token before calling safe transfer
        _deleteLoan(loanId);

        // Transfer repaid loan to lender
        _push(loanAsset, msg.sender);

        emit LOANClaimed(loanId, false);
    }
    // Loan is running but expired
    else if (loan.status == 2 && loan.expiration <= block.timestamp) {
            MultiToken.Asset memory collateral = loan.collateral;

        // Delete loan data & burn LOAN token before calling safe transfer
        _deleteLoan(loanId);

        // Transfer collateral to lender
        _push(collateral, msg.sender);

        emit LOANClaimed(loanId, true);
    }
    // Loan is in wrong state or from a different loan contract
    else {
        revert InvalidLoanStatus(loan.status);
    }
}
```

</details>

<details>

<summary><code>extendLOANExpirationDate</code></summary>

#### Overview

Holders of LOAN tokens (lenders) can decide to extend an expiration date of a loan by up to 30 days from the time the transaction is included in a block.&#x20;

This function takes two arguments supplied by the caller:

* `uint256`**`loanId`** - ID of the loan that is being extended
* `uint40`**`extendedExpirationDate`** - New expiration (maturity) date of the loan. Has to be in the future and maximally 30 days from the previous expiration

#### Implementation

```solidity
function extendLOANExpirationDate(uint256 loanId, uint40 extendedExpirationDate) external {
    // Check that caller is LOAN token holder
    // This prevents from extending non-existing loans
    if (loanToken.ownerOf(loanId) != msg.sender)
        revert CallerNotLOANTokenHolder();

    LOAN storage loan = LOANs[loanId];

    // Check extended expiration date
    if (extendedExpirationDate > uint40(block.timestamp + MAX_EXPIRATION_EXTENSION)) // to protect lender
        revert InvalidExtendedExpirationDate();
    if (extendedExpirationDate <= uint40(block.timestamp)) // have to extend expiration futher in time
        revert InvalidExtendedExpirationDate();
    if (extendedExpirationDate <= loan.expiration) // have to be later than current expiration date
        revert InvalidExtendedExpirationDate();

    // Extend expiration date
    loan.expiration = extendedExpirationDate;

    emit LOANExpirationDateExtended(loanId, extendedExpirationDate);
}
```

</details>

### View Functions

<details>

<summary><code>getLOAN</code></summary>

#### Overview

Returns a Loan struct with information about a supplied loan ID.&#x20;

This function takes one argument supplied by the caller:

* `uint256`**`loanId`** - ID of the loan to get parameters for

#### Implementation

```solidity
function getLOAN(uint256 loanId) external view returns (LOAN memory loan) {
    loan = LOANs[loanId];
    loan.status = _getLOANStatus(loanId);
}
```

</details>

<details>

<summary><code>loanMetadataUri</code></summary>

#### Overview

Returns a [metadata URI](https://docs.opensea.io/docs/metadata-standards) for LOAN tokens. This URI is defined in [PWN Config](../../pwn-config.md).&#x20;

This function doesn't take any arguments.&#x20;

#### Implementation

```solidity
function loanMetadataUri() override external view returns (string memory) {
    return config.loanMetadataUri(address(this));
}
```

</details>

<details>

<summary><code>getStateFingerprint</code></summary>

#### Overview

This function returns the current token state fingerprint for a supplied token ID. See [ERC-5646](https://eips.ethereum.org/EIPS/eip-5646) standard specification for more detailed information. &#x20;

This function takes one argument supplied by the caller:

* `uint256`**`tokenId`** - ID of the LOAN token to get a fingerprint for

#### Implementation

```solidity
function getStateFingerprint(uint256 tokenId) external view virtual override returns (bytes32) {
    LOAN storage loan = LOANs[tokenId];

    if (loan.status == 0)
        return bytes32(0);

    // The only mutable state properties are:
    // - status, expiration
    // Status is updated for expired loans based on block.timestamp.
    // Others don't have to be part of the state fingerprint as it does not act as a token identification.
    return keccak256(abi.encode(
        _getLOANStatus(tokenId),
        loan.expiration
    ));
}
```

</details>

### Events

The PWN Simple Loan contract defines one event and no custom errors.

```solidity
event LOANCreated(uint256 indexed loanId, PWNLOANTerms.Simple terms);
event LOANPaidBack(uint256 indexed loanId);
event LOANClaimed(uint256 indexed loanId, bool indexed defaulted);
event LOANExpirationDateExtended(uint256 indexed loanId, uint40 extendedExpirationDate);
```

<details>

<summary><code>LOANCreated</code></summary>

LOANCreated is emitted when a new simple loan is started.&#x20;

This event has two parameters:

* `uint256 indexed`**`loanId`** - ID of the LOAN token that is associated with the created loan
* `PWNLOANTerms.Simple`**`terms`** - Struct with the parameters of the created loan. See [PWN Loan Terms](../../miscellaneous/pwn-loan-terms.md) for more information

</details>

<details>

<summary><code>LOANPaidBack</code></summary>

LOANPaidBack event is emitted when a borrower repays a simple loan.

This event has one parameter:

* `uint256 indexed`**`loanId`** - ID of the LOAN token that is associated with the repaid loan

</details>

<details>

<summary><code>LOANClaimed</code></summary>

LOANClaimed event is emitted when a lender claims repaid asset or defaulted collateral.&#x20;

This event has two parameters:

* `uint256 indexed`**`loanId`** - ID of the LOAN token that is associated with the claimed loan
* `bool indexed`**`defaulted`** - Boolean determining if the claimed loan was defaulted or properly repaid

</details>

<details>

<summary><code>LOANExpirationDateExtended</code></summary>

LOANExpirationDateExtended event is emitted when a lender extends the loan maturity date.&#x20;

This event has two parameters:

* `uint256 indexed`**`loanId`** - ID of the LOAN token that is associated with the loan being extended
* `uint40`**`extendedExpirationDate`** - New expiration (maturity) date of the loan

</details>

### Simple Loan Struct

<table><thead><tr><th width="152.09421454876235">Type</th><th width="167.45656287647148">Name</th><th>Comment</th></tr></thead><tbody><tr><td><code>uint8</code></td><td><code>status</code></td><td>0 -> None/Dead<br>2 -> Running/accepted offer/accepted request<br>3 -> Repaid<br>4 -> Expired</td></tr><tr><td><code>address</code></td><td><code>borrower</code></td><td>Address of a borrower</td></tr><tr><td><code>uint40</code></td><td><code>expiration</code></td><td>Unix timestamp (in seconds) setting up a maturity date</td></tr><tr><td><code>address</code></td><td><code>loanAssetAddress</code></td><td>Asset used as a loan credit. See <a href="../../../../libraries/multitoken.md">MultiToken</a> for more infomation about the Asset struct.</td></tr><tr><td><code>uint256</code></td><td><code>loanRepayAmount</code></td><td>Amount of <code>asset</code> to be repaid. </td></tr><tr><td><code>MultiToken.Asset</code></td><td><code>collateral</code></td><td>Asset used as a loan collateral. See <a href="../../../../libraries/multitoken.md">MultiToken</a> for more infomation about the Asset struct.</td></tr></tbody></table>
