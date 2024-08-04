# Simple Loan Proposal

## 1. Summary

PWNSimpleLoanProposal.sol is an abstract contract inherited by Simple Loan Proposal types.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/simple/proposal/PWNSimpleLoanProposal.sol" %}

## 3. Contract details

* _PWNSimpleLoanProposal.sol_ is written in Solidity version 0.8.16

### Features

* Defines base interface for Simple Loan Proposals
* Defines [`ProposalBase`](./#proposalbase-struct) struct
* Implements `_makeProposal` and `_acceptProposal` functions

### Functions

<details>

<summary><code>revokeNonce</code></summary>

#### Overview

A helper function for revoking a proposal nonce on behalf of a caller.

This function takes two arguments supplied by the caller:

* `uint256`**`nonceSpace`** - Nonce space of a proposal nonce to be revoked
* `uint256`**`nonce`** - Proposal nonce to be revoked

#### Implementation

```solidity
function revokeNonce(uint256 nonceSpace, uint256 nonce) external {
    revokedNonce.revokeNonce(msg.sender, nonceSpace, nonce);
}
```

</details>

### Internal Functions

<details>

<summary><code>_makeProposal</code></summary>

#### Overview

Function to make an on-chain proposal.

This function takes two arguments:

* `bytes32`**`proposalHash`** - hash of the respective proposal type struct
* `address`**`proposer`** - Address of a proposal proposer

#### Implementation

```solidity
function _makeProposal(bytes32 proposalHash, address proposer) internal {
    if (msg.sender != proposer) {
        revert CallerIsNotStatedProposer({ addr: proposer });
    }

    proposalsMade[proposalHash] = true;
}
```

</details>

<details>

<summary><code>_acceptProposal</code></summary>

#### Overview

Makes necessary checks for accepting a proposal and reverts if any loan parameters are not valid.&#x20;

This function takes six arguments:

* `address`**`acceptor`** - Address of a proposal acceptor
* `uint256`**`refinancingLoanId`** - Refinancing loan ID
* `bytes32`**`proposalHash`** - Proposal hash
* `bytes32[] calldata`**`proposalInclusionProof`** - Multiproposal inclusion proof. Empty if single proposal
* `bytes calldata`**`signature`** - Signature of a proposal
* `ProposalBase memory`**`proposal`** - [`ProposalBase`](./#proposalbase-struct) struct

#### Implementation

```solidity
function _acceptProposal(
    address acceptor,
    uint256 refinancingLoanId,
    bytes32 proposalHash,
    bytes32[] calldata proposalInclusionProof,
    bytes calldata signature,
    ProposalBase memory proposal
) internal {
    // Check loan contract
    if (msg.sender != proposal.loanContract) {
        revert CallerNotLoanContract({ caller: msg.sender, loanContract: proposal.loanContract });
    }
    if (!hub.hasTag(proposal.loanContract, PWNHubTags.ACTIVE_LOAN)) {
        revert AddressMissingHubTag({ addr: proposal.loanContract, tag: PWNHubTags.ACTIVE_LOAN });
    }

    // Check proposal signature or that it was made on-chain
    if (proposalInclusionProof.length == 0) {
        // Single proposal signature
        if (!proposalsMade[proposalHash]) {
            if (!PWNSignatureChecker.isValidSignatureNow(proposal.proposer, proposalHash, signature)) {
                revert PWNSignatureChecker.InvalidSignature({ signer: proposal.proposer, digest: proposalHash });
            }
        }
    } else {
        // Multiproposal signature
        bytes32 multiproposalHash = getMultiproposalHash(
            Multiproposal({
                multiproposalMerkleRoot: MerkleProof.processProofCalldata({
                    proof: proposalInclusionProof,
                    leaf: proposalHash
                })
            })
        );
        if (!PWNSignatureChecker.isValidSignatureNow(proposal.proposer, multiproposalHash, signature)) {
            revert PWNSignatureChecker.InvalidSignature({ signer: proposal.proposer, digest: multiproposalHash });
        }
    }

    // Check proposer is not acceptor
    if (proposal.proposer == acceptor) {
        revert AcceptorIsProposer({ addr: acceptor});
    }

    // Check refinancing proposal
    if (refinancingLoanId == 0) {
        if (proposal.refinancingLoanId != 0) {
            revert InvalidRefinancingLoanId({ refinancingLoanId: proposal.refinancingLoanId });
        }
    } else {
        if (refinancingLoanId != proposal.refinancingLoanId) {
            if (proposal.refinancingLoanId != 0 || !proposal.isOffer) {
                revert InvalidRefinancingLoanId({ refinancingLoanId: proposal.refinancingLoanId });
            }
        }
    }

    // Check proposal is not expired
    if (block.timestamp >= proposal.expiration) {
        revert Expired({ current: block.timestamp, expiration: proposal.expiration });
    }

    // Check proposal is not revoked
    if (!revokedNonce.isNonceUsable(proposal.proposer, proposal.nonceSpace, proposal.nonce)) {
        revert PWNRevokedNonce.NonceNotUsable({
            addr: proposal.proposer,
            nonceSpace: proposal.nonceSpace,
            nonce: proposal.nonce
        });
    }

    // Check propsal is accepted by an allowed address
    if (proposal.allowedAcceptor != address(0) && acceptor != proposal.allowedAcceptor) {
        revert CallerNotAllowedAcceptor({ current: acceptor, allowed: proposal.allowedAcceptor });
    }

    if (proposal.availableCreditLimit == 0) {
        // Revoke nonce if credit limit is 0, proposal can be accepted only once
        revokedNonce.revokeNonce(proposal.proposer, proposal.nonceSpace, proposal.nonce);
    } else if (creditUsed[proposalHash] + proposal.creditAmount <= proposal.availableCreditLimit) {
        // Increase used credit if credit limit is not exceeded
        creditUsed[proposalHash] += proposal.creditAmount;
    } else {
        // Revert if credit limit is exceeded
        revert AvailableCreditLimitExceeded({
            used: creditUsed[proposalHash] + proposal.creditAmount,
            limit: proposal.availableCreditLimit
        });
    }

    // Check collateral state fingerprint if needed
    if (proposal.checkCollateralStateFingerprint) {
        bytes32 currentFingerprint;
        IStateFingerpringComputer computer = config.getStateFingerprintComputer(proposal.collateralAddress);
        if (address(computer) != address(0)) {
            // Asset has registered computer
            currentFingerprint = computer.computeStateFingerprint({
                token: proposal.collateralAddress, tokenId: proposal.collateralId
            });
        } else if (ERC165Checker.supportsInterface(proposal.collateralAddress, type(IERC5646).interfaceId)) {
            // Asset implements ERC5646
            currentFingerprint = IERC5646(proposal.collateralAddress).getStateFingerprint(proposal.collateralId);
        } else {
            // Asset is not implementing ERC5646 and no computer is registered
            revert MissingStateFingerprintComputer();
        }

        if (proposal.collateralStateFingerprint != currentFingerprint) {
            // Fingerprint mismatch
            revert InvalidCollateralStateFingerprint({
                current: currentFingerprint,
                proposed: proposal.collateralStateFingerprint
            });
        }
    }
}

}

```

</details>

### View Functions

<details>

<summary><code>getMultiproposalHash</code></summary>

#### Overview

This function returns a multiproposal hash according to [EIP-712](https://eips.ethereum.org/EIPS/eip-712).

This function takes one argument supplied by the caller:

* `Multiproposal memory`**`multiproposal`** - [`Multiproposal`](./#multiproposal-struct) struct

#### Implementation

```solidity
function getMultiproposalHash(Multiproposal memory multiproposal) public view returns (bytes32) {
    return keccak256(abi.encodePacked(
        hex"1901", MULTIPROPOSAL_DOMAIN_SEPARATOR, keccak256(abi.encodePacked(
            MULTIPROPOSAL_TYPEHASH, abi.encode(multiproposal)
        ))
    ));
}
```

</details>

<details>

<summary><code>_getProposalHash</code></summary>

#### Overview

This function returns a proposal hash according to [EIP-712](https://eips.ethereum.org/EIPS/eip-712).

This function takes two arguments supplied by the caller:

* `bytes32`**`proposalTypehash`** - Hash of the respective proposal type
* `bytes memory`**`encodedProposal`** - Encoded respective proposal type struct

#### Implementation

```solidity
function _getProposalHash(
    bytes32 proposalTypehash,
    bytes memory encodedProposal
) internal view returns (bytes32) {
    return keccak256(abi.encodePacked(
        hex"1901", DOMAIN_SEPARATOR, keccak256(abi.encodePacked(
            proposalTypehash, encodedProposal
        ))
    ));
}
```

</details>

### Errors

The PWN Simple Loan Offer contract defines eight errors and no events.

```solidity
error CallerNotLoanContract(address caller, address loanContract);
error MissingStateFingerprintComputer();
error InvalidCollateralStateFingerprint(bytes32 current, bytes32 proposed);
error CallerIsNotStatedProposer(address addr);
error AcceptorIsProposer(address addr);
error InvalidRefinancingLoanId(uint256 refinancingLoanId);
error AvailableCreditLimitExceeded(uint256 used, uint256 limit);
error CallerNotAllowedAcceptor(address current, address allowed);
```

<details>

<summary><code>CallerNotLoanContract</code></summary>

A CallerNotLoanContract error is thrown when a caller is missing a required hub tag.

This error has two parameters:

* `address`**`caller`**
* `address`**`loanContract`**

</details>

<details>

<summary><code>MissingStateFingerprintComputer</code></summary>

A MissingStateFingerprintComputer error is thrown when a state fingerprint computer is not registered.

This error doesn't define any parameters.

</details>

<details>

<summary><code>InvalidCollateralStateFingerprint</code></summary>

A InvalidCollateralStateFingerprint error is thrown when a proposed collateral state fingerprint doesn't match the current state.

This error has two parameters:

* `bytes32`**`current`**
* `bytes32`**`proposed`**

</details>

<details>

<summary><code>CallerIsNotStatedProposer</code></summary>

A CallerIsNotStatedProposer error is thrown when a caller is not a stated proposer.

This error has one parameter:

* `address`**`addr`**

</details>

<details>

<summary><code>AcceptorIsProposer</code></summary>

An AcceptorIsProposer error is thrown when proposal acceptor and proposer are the same.

This error has one parameter:

* `address`**`addr`**

</details>

<details>

<summary><code>InvalidRefinancingLoanId</code></summary>

An InvalidRefinancingLoanId error is thrown when provided refinance loan id cannot be used.

This error has one parameter:

* `uint256`**`refinancingLoanId`**

</details>

<details>

<summary><code>AvailableCreditLimitExceeded</code></summary>

An AvailableCreditLimitExceeded error is thrown when a proposal would exceed the available credit limit.

This error has two parameters:

* `uint256`**`used`**
* `uint256`**`limit`**

</details>

<details>

<summary><code>CallerNotAllowedAcceptor</code></summary>

A CallerNotAllowedAcceptor error is thrown when caller is not allowed to accept a proposal.

This error has two parameters:

* `address`**`current`**
* `address`**`allowed`**

</details>

### `ProposalBase` Struct

<table><thead><tr><th width="124.09421454876235">Type</th><th width="211.45656287647148">Name</th><th>Comment</th></tr></thead><tbody><tr><td><code>address</code></td><td><code>collateralAddress</code></td><td>Address of a loan collateral</td></tr><tr><td><code>uint256</code></td><td><code>collateralId</code></td><td>ID of a collateral. Zero if ERC-20</td></tr><tr><td><code>bool</code></td><td><code>checkCollateralStateFingerprint</code></td><td>Flag to enable check of collaterals state fingerprint (see <a href="https://eips.ethereum.org/EIPS/eip-5646">ERC-5</a><a href="https://eips.ethereum.org/EIPS/eip-5646">646</a>)</td></tr><tr><td><code>bytes32</code></td><td><code>collateralStateFingerprint</code></td><td>A collateral state fingerprint (see <a href="https://eips.ethereum.org/EIPS/eip-5646">ERC-5</a><a href="https://eips.ethereum.org/EIPS/eip-5646">646</a>)</td></tr><tr><td><code>uint256</code></td><td><code>creditAmount</code></td><td>Amount of credit asset</td></tr><tr><td><code>uint256</code></td><td><code>availableCreditLimit</code></td><td>Maximum credit limit of credit asset</td></tr><tr><td><code>uint40</code></td><td><code>expiration</code></td><td>Proposal expiration unix timestamp in seconds</td></tr><tr><td><code>address</code></td><td><code>allowedAcceptor</code></td><td>Allowed acceptor address. Zero address if propsal can be accepted by any account</td></tr><tr><td><code>address</code></td><td><code>proposer</code></td><td>Proposer address</td></tr><tr><td><code>bool</code></td><td><code>isOffer</code></td><td>Flag to determine if a proposal is an offer or loan request</td></tr><tr><td><code>uint256</code></td><td><code>refinancingLoanId</code></td><td>ID of a loan to be refinanced. Zero if creating a new loan.</td></tr><tr><td><code>uint256</code></td><td><code>nonceSpace</code></td><td>Nonce space of the proposal</td></tr><tr><td><code>uint256</code></td><td><code>nonce</code></td><td>Nonce of the proposal</td></tr><tr><td><code>address</code></td><td><code>loanContract</code></td><td>Loan type contract</td></tr></tbody></table>

### `Multiproposal` Struct

<table><thead><tr><th width="124.09421454876235">Type</th><th width="270.4565628764715">Name</th><th>Comment</th></tr></thead><tbody><tr><td><code>bytes32</code></td><td><code>multiproposalMerkleRoot</code></td><td>Root of the multiproposal merkle tree</td></tr></tbody></table>
