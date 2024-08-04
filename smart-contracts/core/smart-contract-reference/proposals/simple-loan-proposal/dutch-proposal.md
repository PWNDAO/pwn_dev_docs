# Dutch Proposal

## 1. Summary

PWNSimpleLoanDutchAuctionProposal.sol defines the Dutch Auction Proposal type for Simple Loan and implements functions to make an on-chain proposal and accept proposals.&#x20;

The Dutch Auction Proposal creates a dutch auction with user pre-defined collateral, loan duration and interest. Credit amount drops (or goes up, if proposal is an offer) from user defined minimum and maximum for the auction duration from auction start. Interest can be either accruing or fixed.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/simple/proposal/PWNSimpleLoanDutchAuctionProposal.sol" %}

{% file src="../../../../../.gitbook/assets/PWNSimpleLoanDutchAuctionProposal.json" %}

## 3. Contract details

* _PWNSimpleLoanDutchAuctionProposal.sol_ is written in Solidity version 0.8.16

### Features

* Provides `acceptProposal` function and `makeProposal` for on-chain proposals
* Defines the `Proposal` struct

### Inherited contracts, implemented Interfaces and ERCs

* [PWNSimpleLoanProposal](./)

### Functions

<details>

<summary><code>acceptProposal</code></summary>

#### Overview

A function to accept a proposal.

This function takes five arguments supplied by the caller:

* `address`**`acceptor`** - Address of a proposal acceptor
* `uint256`**`refinancingLoanId`** - Refinancing loan ID
* `bytes32 calldata`**`proposalData`** - Encoded proposal struct
* `bytes32[] calldata`**`proposalInclusionProof`** - Multiproposal inclusion proof. Empty if single proposal
* `bytes calldata`**`signature`** - Signature of a proposal

#### Implementation

```solidity
function acceptProposal(
    address acceptor,
    uint256 refinancingLoanId,
    bytes calldata proposalData,
    bytes32[] calldata proposalInclusionProof,
    bytes calldata signature
) override external returns (bytes32 proposalHash, PWNSimpleLoan.Terms memory loanTerms) {
    // Decode proposal data
    (Proposal memory proposal, ProposalValues memory proposalValues) = decodeProposalData(proposalData);

    // Make proposal hash
    proposalHash = _getProposalHash(PROPOSAL_TYPEHASH, abi.encode(proposal));

    // Calculate current credit amount
    uint256 creditAmount = getCreditAmount(proposal, block.timestamp);

    // Check acceptor values
    if (proposal.isOffer) {
        if (
            creditAmount < proposalValues.intendedCreditAmount ||
            proposalValues.intendedCreditAmount + proposalValues.slippage < creditAmount
        ) {
            revert InvalidCreditAmount({
                auctionCreditAmount: creditAmount,
                intendedCreditAmount: proposalValues.intendedCreditAmount,
                slippage: proposalValues.slippage
            });
        }
    } else {
        if (
            creditAmount > proposalValues.intendedCreditAmount ||
            proposalValues.intendedCreditAmount - proposalValues.slippage > creditAmount
        ) {
            revert InvalidCreditAmount({
                auctionCreditAmount: creditAmount,
                intendedCreditAmount: proposalValues.intendedCreditAmount,
                slippage: proposalValues.slippage
            });
        }
    }

    // Try to accept proposal
    _acceptProposal(
        acceptor,
        refinancingLoanId,
        proposalHash,
        proposalInclusionProof,
        signature,
        ProposalBase({
            collateralAddress: proposal.collateralAddress,
            collateralId: proposal.collateralId,
            checkCollateralStateFingerprint: proposal.checkCollateralStateFingerprint,
            collateralStateFingerprint: proposal.collateralStateFingerprint,
            creditAmount: creditAmount,
            availableCreditLimit: proposal.availableCreditLimit,
            expiration: proposal.auctionStart + proposal.auctionDuration + 1 minutes,
            allowedAcceptor: proposal.allowedAcceptor,
            proposer: proposal.proposer,
            isOffer: proposal.isOffer,
            refinancingLoanId: proposal.refinancingLoanId,
            nonceSpace: proposal.nonceSpace,
            nonce: proposal.nonce,
            loanContract: proposal.loanContract
        })
    );

    // Create loan terms object
    loanTerms = PWNSimpleLoan.Terms({
        lender: proposal.isOffer ? proposal.proposer : acceptor,
        borrower: proposal.isOffer ? acceptor : proposal.proposer,
        duration: proposal.duration,
        collateral: MultiToken.Asset({
            category: proposal.collateralCategory,
            assetAddress: proposal.collateralAddress,
            id: proposal.collateralId,
            amount: proposal.collateralAmount
        }),
        credit: MultiToken.ERC20({
            assetAddress: proposal.creditAddress,
            amount: creditAmount
        }),
        fixedInterestAmount: proposal.fixedInterestAmount,
        accruingInterestAPR: proposal.accruingInterestAPR,
        lenderSpecHash: proposal.isOffer ? proposal.proposerSpecHash : bytes32(0),
        borrowerSpecHash: proposal.isOffer ? bytes32(0) : proposal.proposerSpecHash
    });
}
```

</details>

<details>

<summary><code>makeProposal</code></summary>

#### Overview

Function to create an on-chain proposal. Marks the hash of the supplied proposal as proposed.

This function takes one argument supplied by the caller:

* `Proposal calldata`**`proposal`** - Proposal struct containing all needed proposal data

#### Implementation

```solidity
function makeProposal(Proposal calldata proposal) external returns (bytes32 proposalHash) {
    proposalHash = getProposalHash(proposal);
    _makeProposal(proposalHash, proposal.proposer);
    emit ProposalMade(proposalHash, proposal.proposer, proposal);
}
```

</details>

### View Functions

<details>

<summary><code>getProposalHash</code></summary>

#### Overview

This function returns supplied proposals hash according to [EIP-712](https://eips.ethereum.org/EIPS/eip-712).

This function takes one argument supplied by the caller:

* `Proposal calldata`**`proposal`** - Proposal struct to be hashed

#### Implementation

```solidity
function getProposalHash(Proposal calldata proposal) public view returns (bytes32) {
    return _getProposalHash(PROPOSAL_TYPEHASH, abi.encode(proposal));
}
```

</details>

<details>

<summary><code>encodeProposalData</code></summary>

#### Overview

Function to encode a proposal struct and proposal values.

This function takes two arguments supplied by the caller:

* `Proposal memory`**`proposal`** - Proposal struct to be encoded
* `ProposalValues memory`**`proposalValues`** - ProposalValues struct to be encoded

#### Implementation

```solidity
function encodeProposalData(
    Proposal memory proposal,
    ProposalValues memory proposalValues
) external pure returns (bytes memory) {
    return abi.encode(proposal, proposalValues);
}
```

</details>

<details>

<summary><code>decodeProposalData</code></summary>

#### Overview

Function to decode an encoded proposal struct and proposal values.

This function takes one argument supplied by the caller:

* `bytes memory`**`proposalData`** - Encoded proposal and proposal values structs

#### Implementation

```solidity
function decodeProposalData(bytes memory proposalData) public pure returns (Proposal memory, ProposalValues memory) {
    return abi.decode(proposalData, (Proposal, ProposalValues));
}
```

</details>

<details>

<summary><code>getCreditAmount</code></summary>

#### Overview

Function to compute credit amount for an auction in a specific timestamp.

This function takes two arguments supplied by the caller:

* `Proposal memory`**`proposal`** - Proposal struct containing all proposal data
* `uint256`**`timestamp`** - Timestamp to calculate auction credit amount for

#### Implementation

```solidity
function getCreditAmount(Proposal memory proposal, uint256 timestamp) public pure returns (uint256) {
    // Check proposal
    if (proposal.auctionDuration < 1 minutes) {
        revert InvalidAuctionDuration({
            current: proposal.auctionDuration,
            limit: 1 minutes
        });
    }
    if (proposal.auctionDuration % 1 minutes > 0) {
        revert AuctionDurationNotInFullMinutes({
            current: proposal.auctionDuration
        });
    }
    if (proposal.maxCreditAmount <= proposal.minCreditAmount) {
        revert InvalidCreditAmountRange({
            minCreditAmount: proposal.minCreditAmount,
            maxCreditAmount: proposal.maxCreditAmount
        });
    }

    // Check auction is in progress
    if (timestamp < proposal.auctionStart) {
        revert AuctionNotInProgress({
            currentTimestamp: timestamp,
            auctionStart: proposal.auctionStart
        });
    }
    if (proposal.auctionStart + proposal.auctionDuration + 1 minutes <= timestamp) {
        revert Expired({
            current: timestamp,
            expiration: proposal.auctionStart + proposal.auctionDuration + 1 minutes
        });
    }

    // Note: Auction duration is increased by 1 minute to have
    // `maxCreditAmount` value in the last minutes of the auction.

    uint256 creditAmountDelta = Math.mulDiv(
        proposal.maxCreditAmount - proposal.minCreditAmount, // Max credit amount difference
        (timestamp - proposal.auctionStart) / 1 minutes, // Time passed since auction start
        proposal.auctionDuration / 1 minutes // Auction duration
    );

    // Note: Request auction is decreasing credit amount (dutch auction).
    // Offer auction is increasing credit amount (reverse dutch auction).

    // Return credit amount
    return proposal.isOffer
        ? proposal.minCreditAmount + creditAmountDelta
        : proposal.maxCreditAmount - creditAmountDelta;
}
```

</details>

### Events

The PWN Simple Loan Dutch Auction Proposal contract defines one event and five errors.

```solidity
event ProposalMade(bytes32 indexed proposalHash, address indexed proposer, Proposal proposal);
```

<details>

<summary><code>ProposalMade</code></summary>

ProposalMade event is emitted when an on-chain proposal is made.

This event has three parameters:

* `bytes32 indexed`**`proposalHash`** - Hash of the proposed proposal
* `address indexed`**`proposer`** - Address of the proposer
* `Proposal`**`proposal`** - The proposal made

</details>

### `Errors`

```solidity
error InvalidAuctionDuration(uint256 current, uint256 limit);
error AuctionDurationNotInFullMinutes(uint256 current);
error InvalidCreditAmountRange(uint256 minCreditAmount, uint256 maxCreditAmount);
error InvalidCreditAmount(uint256 auctionCreditAmount, uint256 intendedCreditAmount, uint256 slippage);
error AuctionNotInProgress(uint256 currentTimestamp, uint256 auctionStart);
```

<details>

<summary><code>InvalidAuctionDuration</code></summary>

InvalidAuctionDuration error is thrown when auction duration is less than min auction duration.

This error has two parameters:

* `uint256`**`current`** - Provided duration
* `uint256`**`limit`** - Minimal duration (1 minute)

</details>

<details>

<summary><code>AuctionDurationNotInFullMinutes</code></summary>

AuctionDurationNotInFullMinutes error is thrown when auction duration is not in full minutes.

This error has two parameters:

* `uint256`**`current`** - Supplied duration

</details>

<details>

<summary><code>InvalidCreditAmountRange</code></summary>

InvalidCreditAmountRange error is thrown when min credit amount is greater than max credit amount.

This error has two parameters:

* `uint256`**`minCreditAmount`**
* `uint256`**`maxCreditAmount`**

</details>

<details>

<summary><code>InvalidCreditAmount</code></summary>

InvalidCreditAmount error is thrown when current auction credit amount is not in the range of intended credit amount and slippage.

This error has three parameters:

* `uint256`**`auctionCreditAmount`**
* `uint256`**`intendedCreditAmount`**
* `uint256`**`slippage`**

</details>

<details>

<summary><code>AuctionNotInProgress</code></summary>

AuctionNotInProgress error is thrown when auction has not started yet or has already ended.

This error has two parameters:

* `uint256`**`currentTimestamp`**
* `uint256`**`auctionStart`**

</details>

### `Proposal` struct

<table><thead><tr><th width="156.09421454876235">Type</th><th width="265.4565628764715">Name</th><th>Comment</th></tr></thead><tbody><tr><td><a data-footnote-ref href="#user-content-fn-1"><code>MultiToken.Category</code></a></td><td><code>collateralCategory</code></td><td>Corresponding collateral category</td></tr><tr><td><code>address</code></td><td><code>collateralAddress</code></td><td>Address of a loan collateral</td></tr><tr><td><code>uint256</code></td><td><code>collateralId</code></td><td>ID of a collateral. Zero if ERC-20</td></tr><tr><td><code>uint256</code></td><td><code>minCollateralAmount</code></td><td>Minimal amount of tokens used as a collateral</td></tr><tr><td><code>bool</code></td><td><code>checkCollateralStateFingerprint</code></td><td>Flag to enable check of collaterals state fingerprint (see <a href="https://eips.ethereum.org/EIPS/eip-5646">ERC-5</a><a href="https://eips.ethereum.org/EIPS/eip-5646">646</a>)</td></tr><tr><td><code>bytes32</code></td><td><code>collateralStateFingerprint</code></td><td>A collateral state fingerprint (see <a href="https://eips.ethereum.org/EIPS/eip-5646">ERC-5</a><a href="https://eips.ethereum.org/EIPS/eip-5646">646</a>)</td></tr><tr><td><code>address</code></td><td><code>creditAddress</code></td><td>Address of credit asset</td></tr><tr><td><code>uint256</code></td><td><code>minCreditAmount</code></td><td>Minimum amount of tokens which are proposed as a loan to a borrower. If <code>isOffer</code> is true, auction will start with this amount, otherwise it will end with this amount.</td></tr><tr><td><code>uint256</code></td><td><code>maxCreditAmount</code></td><td>Maximum amount of tokens which is proposed as a loan to a borrower. If <code>isOffer</code> is true, auction will end with this amount, otherwise it will start with this amount.</td></tr><tr><td><code>uint256</code></td><td><code>availableCreditLimit</code></td><td>Maximum credit limit of credit asset</td></tr><tr><td><code>uint256</code></td><td><code>fixedInterestAmount</code></td><td>Fixed interest amount in credit tokens. It is the minimum amount of interest which has to be paid by a borrower</td></tr><tr><td><code>uint24</code></td><td><code>accruingInterestAPR</code></td><td>Accruing interest APR with 2 decimals</td></tr><tr><td><code>uint32</code></td><td><code>duration</code></td><td>Loan duration in seconds</td></tr><tr><td><code>uint40</code></td><td><code>auctionStart</code></td><td>Auction start timestamp in seconds</td></tr><tr><td><code>uint40</code></td><td><code>auctionDuration</code></td><td>Auction duration in seconds</td></tr><tr><td><code>address</code></td><td><code>allowedAcceptor</code></td><td>Allowed acceptor address. Zero address if propsal can be accepted by any account</td></tr><tr><td><code>address</code></td><td><code>proposer</code></td><td>Proposer address</td></tr><tr><td><code>bytes32</code></td><td><code>proposerSpecHash</code></td><td>Hash of a proposer specific data, which must be provided during a loan creation</td></tr><tr><td><code>bool</code></td><td><code>isOffer</code></td><td>Flag to determine if a proposal is an offer or loan request</td></tr><tr><td><code>uint256</code></td><td><code>refinancingLoanId</code></td><td>ID of a loan to be refinanced. Zero if creating a new loan.</td></tr><tr><td><code>uint256</code></td><td><code>nonceSpace</code></td><td>Nonce space of the proposal</td></tr><tr><td><code>uint256</code></td><td><code>nonce</code></td><td>Nonce of the proposal</td></tr><tr><td><code>address</code></td><td><code>loanContract</code></td><td>Loan type contract</td></tr></tbody></table>

[^1]: A **category** is defined as an [enum](https://docs.soliditylang.org/en/v0.8.12/structure-of-a-contract.html?highlight=enum#enum-types) and can have values `ERC20`, `ERC721` or `ERC1155`.
