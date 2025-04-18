# State Fingerprint Computer

## 1. Summary

IStateFingerpringComputer.sol defines interface each State Fingerprint Computer contract has to implement. State Fingerprint Computers allow usage of dynamic assets in the PWN Protocol that don't implement [ERC-5646](https://eips.ethereum.org/EIPS/eip-5646) natively.

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/interfaces/IStateFingerpringComputer.sol" %}

## 3. Implementation

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.16;

/**
 * @title IStateFingerpringComputer
 * @notice State Fingerprint Computer Interface.
 * @dev Contract can compute state fingerprint of several tokens as long as they share the same state structure.
 */
interface IStateFingerpringComputer {

    /**
     * @notice Compute current token state fingerprint for a given token.
     * @param token Address of a token contract.
     * @param tokenId Token id to compute state fingerprint for.
     * @return Current token state fingerprint.
     */
    function computeStateFingerprint(address token, uint256 tokenId) external view returns (bytes32);

    /**
     * @notice Check if the computer supports a given token address.
     * @param token Address of a token contract.
     * @return True if the computer supports the token address, false otherwise.
     */
    function supportsToken(address token) external view returns (bool);

}

```
